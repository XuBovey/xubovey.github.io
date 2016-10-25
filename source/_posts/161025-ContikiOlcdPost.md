---
title: Contiki OLCD移植
toc: true
date: 2016-10-25 14:28:32
categories:
- Contiki
- OLCD
tags:
- Contiki
- OLCD
---

硬件环境AMOMCU的CC2650DK V0.2;  
使用例程contiki/examples/hello-world;  
经过前一阶段对Contiki的深入了解，目前算是对定时器和进程调用有了一个全面的了解。好吧开始和printf较真。例程中默认输出是到串口的，而我想让他输出到OLCD上。 
如果您有同样的目的或需要可以继续往下看： 
<!--more-->

# printf与putchar
contiki中不同平台的输出相关函数多在cpu路径下，以CC26XX为例，在contiki/cpu/cc26xx/putchar.c文件中。  
文件内包含三个函数：
``` c
int putchar(int c);
int puts(const char *str);
unsigned int dbg_send_bytes(const unsigned char *s, unsigned int len);
```
以`int putchar(int c)`函数为例进行说明，该函数完成将单个数据进行发送。而发送是通过函数中的语句`cc26xx_uart_write_byte(c);`实现的，那么移植就很简单了只要在该语句出现的地方放上实现相同功能，只不过是输出到LCD的函数即可,此处为`olcd_write_byte(c);`

# 接口初始化
同样参考串口的初始化函数，参考[从main函数分析contiki](https://xubovey.github.io/2016/10/20/161020-Contiki-platform-main-function-analyse/),可以找到main函数相关的描述，分析。结论串口初始化在进入main函数的while(1)之前完成，更进一步是在main函数中printf函数执行之前进行的。  
此处将相应函数放在了led_init之后：
``` c
  leds_init();
  hw_lcd_init();
```
hw_lcd_init函数体根据屏的不同也会有所不同。此处参考了开发板提供的源码，最终代码如下：
``` c
#define GPIOPinWrite(...) GPIO_writeDio(__VA_ARGS__)

unsigned short inited = 0;
/*********************LCD init**********************************/
void hw_lcd_init(void)
{
    if(inited)
    {
        return;
    }
    inited = 1;
    
    ti_lib_ioc_pin_type_gpio_output(LCD_SCL);
    ti_lib_ioc_pin_type_gpio_output(LCD_SDA);
    ti_lib_ioc_pin_type_gpio_output(LCD_RST);
    ti_lib_ioc_pin_type_gpio_output(LCD_DC);   

    GPIOPinWrite(LCD_SCL, 1);  
    GPIOPinWrite(LCD_RST, 0);

    LCD_DLY_ms(250);

    GPIOPinWrite(LCD_RST, 1);      //从上电到下面开始初始化要有足够的时间，即等待RC复位完毕   

    LCD_WrCmd(0xae);//--turn off oled panel
    LCD_WrCmd(0x00);//---set low column address
    LCD_WrCmd(0x10);//---set high column address
    LCD_WrCmd(0x40);//--set start line address  Set Mapping RAM Display Start Line (0x00~0x3F)
    LCD_WrCmd(0x81);//--set contrast control register
    LCD_WrCmd(0xcf); // Set SEG Output Current Brightness
    LCD_WrCmd(0xa1);//--Set SEG/Column Mapping     0xa0左右反置 0xa1正常
    LCD_WrCmd(0xc8);//Set COM/Row Scan Direction   0xc0上下反置 0xc8正常
    LCD_WrCmd(0xa6);//--set normal display
    LCD_WrCmd(0xa8);//--set multiplex ratio(1 to 64)
    LCD_WrCmd(0x3f);//--1/64 duty
    LCD_WrCmd(0xd3);//-set display offset    Shift Mapping RAM Counter (0x00~0x3F)
    LCD_WrCmd(0x00);//-not offset
    LCD_WrCmd(0xd5);//--set display clock divide ratio/oscillator frequency
    LCD_WrCmd(0x80);//--set divide ratio, Set Clock as 100 Frames/Sec
    LCD_WrCmd(0xd9);//--set pre-charge period
    LCD_WrCmd(0xf1);//Set Pre-Charge as 15 Clocks & Discharge as 1 Clock
    LCD_WrCmd(0xda);//--set com pins hardware configuration
    LCD_WrCmd(0x12);
    LCD_WrCmd(0xdb);//--set vcomh
    LCD_WrCmd(0x40);//Set VCOM Deselect Level
    LCD_WrCmd(0x20);//-Set Page Addressing Mode (0x00/0x01/0x02)
    LCD_WrCmd(0x02);//
    LCD_WrCmd(0x8d);//--set Charge Pump enable/disable
    LCD_WrCmd(0x14);//--set(0x10) disable
    LCD_WrCmd(0xa4);// Disable Entire Display On (0xa4/0xa5)
    LCD_WrCmd(0xa6);// Disable Inverse Display On (0xa6/a7) 
    LCD_WrCmd(0xaf);//--turn on oled panel
    LCD_Fill(0); //初始成黑屏
    LCD_Set_Pos(0,0); 
    lcd_x = 0;
    lcd_y = 0;
}
```
其中lcd_x和lcd_y为屏坐标的全局变量，初始化为(0, 0)。LCD_WrCmd函数为开发板提供的源码，直接使用。

# olcd_write_byte函数
上文提到的olcd_write_byte函数体如下：
``` c
#define LCD_MAX_LINE_COUNT      8
static unsigned char lcd_x, lcd_y;
void olcd_write_byte (unsigned char c)
{
    static unsigned char i;

    if(c == '\n') { lcd_x = 0; lcd_y ++; }//判断是否为换行符
    else
    {
       if(lcd_x > 126){lcd_x = 0; lcd_y ++;}//判断是否需要换行

       if(lcd_y > LCD_MAX_LINE_COUNT)//0~7共8行，判断是否满屏，如果是则清屏
       {
           lcd_y = 0;
           lcd_x = 0;
           LCD_Set_Pos(lcd_x, lcd_y); //修改坐标为(0， 0)
           LCD_Fill(0); //清屏
       }

       c -= 32; //用于匹配字库表F6x8，字库从空格字符“ ”开始，“ ”对应32。
       if(c >= 0 && c <= 94)
       {
           //更新坐标，并将字符写入
           LCD_Set_Pos(lcd_x, lcd_y); 
           for(i=0;i<6;i++)
           {
               LCD_WrDat(F6x8[c][i]); //将数据写入LCD屏，完成字符显示。
           }
       }
       lcd_x += 6;
    }
}
```

