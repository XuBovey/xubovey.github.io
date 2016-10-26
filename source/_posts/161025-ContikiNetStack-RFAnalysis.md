---
title: Contiki RF底层代码分析
toc: true
date: 2016-10-25 16:30:12
categories:
- Contiki
tags:
- Contiki
- RF
---

要先知道怎么工作的才好了解，移植。本文从main函数中的rf相关代码入手，顺藤摸瓜理清RF的相关操作和数据处理流程。

<!--more-->
# 1. main函数中的RF相关代码
为什么从main入手，因为这里有初始化等相关代码，基本可以顺藤摸瓜搞明白里面的问题所在。  
参考文件：contiki/platform/srf06-cc26xx/contiki-main.c，其中main函数里与射频相关的代码只有两句：
``` c
  netstack_init();
  set_rf_params();
```
# 2. set_rf_params函数
这里先看`set_rf_params()`因为函数体就定义在main函数中。核心代码：
``` c
  //获取设备地址，并取出16位小地址。
  ieee_addr_cpy_to(ext_addr, 8); 
  short_addr = ext_addr[7];
  short_addr |= ext_addr[6] << 8; 
  
  //同样是地址配置，这里将地址copy给了linkaddr_node_addr.
  /* Populate linkaddr_node_addr. Maintain endianness */
  memcpy(&linkaddr_node_addr, &ext_addr[8 - LINKADDR_SIZE], LINKADDR_SIZE);

  //这也是一些配置，问题是NETSTACK_RADIO.set_value是什么？
  NETSTACK_RADIO.set_value(RADIO_PARAM_PAN_ID, IEEE802154_PANID);
  NETSTACK_RADIO.set_value(RADIO_PARAM_16BIT_ADDR, short_addr);
  NETSTACK_RADIO.set_value(RADIO_PARAM_CHANNEL, RF_CORE_CHANNEL);
  NETSTACK_RADIO.set_object(RADIO_PARAM_64BIT_ADDR, ext_addr, 8);

  NETSTACK_RADIO.get_value(RADIO_PARAM_CHANNEL, &val);
```
注释中可以看到，这段代码引出了`NETSTACK_RADIO.set_value`函数，`NETSTACK_RADIO`应该是个包含有函数指针的结构体。  
在contiki/core/net/netstack.h文件中找到如下宏定义：
``` c
#define NETSTACK_RADIO NETSTACK_CONF_RADIO
```
继续在contiki/platform/srf06-cc26xx/contiki-conf.h中找到相关代码：
``` c
#define NETSTACK_CONF_RADIO        ieee_mode_driver //这就是我们要找的了。

#ifndef RF_CORE_CONF_CHANNEL
#define RF_CORE_CONF_CHANNEL                     25
#endif

#define NULLRDC_CONF_802154_AUTOACK_HW            1
#define NULLRDC_CONF_SEND_802154_ACK              0
```
嗯，好了把main函数中的set_value相关代码展开如下：
``` c
  ieee_mode_driver.set_value(RADIO_PARAM_PAN_ID, IEEE802154_PANID);
  ieee_mode_driver.set_value(RADIO_PARAM_16BIT_ADDR, short_addr);
  ieee_mode_driver.set_value(RADIO_PARAM_CHANNEL, RF_CORE_CHANNEL);
  ieee_mode_driver.set_object(RADIO_PARAM_64BIT_ADDR, ext_addr, 8);

  ieee_mode_driver.get_value(RADIO_PARAM_CHANNEL, &val);
```
找ieee-mode_driver，在contiki/cpu/cc26xx-cc13xx/rf-core/ieee-mode.c文件中找到该变量。
``` c
const struct radio_driver ieee_mode_driver = {
  init,             //完成参数初始化，中断配置，启动rf_core_process进程
  prepare,          //发送准备，将数据拷贝到tx_buf缓冲区
  transmit,         //完成数据发送
  send,             //调用prepare和transmit，完成发送
  read_frame,       //将当前接收缓冲区数据拷贝到用户缓冲区
  channel_clear,    //判断当前通道是否有被其它设备使用
  receiving_packet, //检查CAA中的RF_CMD_CCA_REQ_CCA_SYNC_BUSY是否为1，判断有接收包
  pending_packet,   //遍历RF接收缓冲区链表，如果有未处理数据调用process_poll(&rf_core_process)触发rf_core_process进程
  on,               //完成一些准备工作后，调用rx_on()->最终调用rf_cmd_ieee_rx()函数
  off,              //等待发送完成后调用rx_off();和rf_core_power_down();关闭rf接口
  get_value,        //读取单个RF配置参数，这些配置在init_rf_params函数（init函数调用）中被赋值
  set_value,        //单个RF参数设置
  get_object,       //单个参数读取RADIO_PARAM_64BIT_ADDR或RADIO_PARAM_LAST_PACKET_TIMESTAMP参数
  set_object,       //单个参数设置
};
```
经过查看set_value，set_object函数，发现这些操作都是在进行一些参数的配置。至此不必继续分析。只是ieee-mode中有些内部函数可以做简单说明：
* static uint8_t rf_is_on(void)         判定RF是否打开，1:处于接收模式，0：其他模式
* static uint8_t transmitting(void)     判断是否处于发送状态，1：发送，0：其他
* static uint8_t get_cca_info(void)     获取CAA信息
* static radio_value_t get_rssi(void)   获取当前信号强度，单位dBm
* static radio_value_t get_tx_power(void)  获取发送功率配置值，单位dBm
* static void set_tx_power(radio_value_t power) 配置发送功率值，单位dBm
* static uint8_t rf_radio_setup()       执行CMD_RADIO_SETUP命令完成参数配置
* static uint8_t rf_cmd_ieee_rx()       配置RF为IEEE802.15.4 RX模式，即：用将cmd_ieee_rx_buf的内容完成RF配置
* static void init_rx_buffers(void)     接收缓冲区初始化为循环链表
* static void init_rf_params(void)      完成cmd_ieee_rx_buf参数设置
* static int rx_on(void)                执行rf_cmd_ieee_rx();打开Rx
* static int rx_off(void)               给RF内核发送CMD_ABORT命令关闭RF
* static uint8_t request(void)          LPM_MODULE(cc26xx_rf_lpm_module, request, NULL, NULL, LPM_DOMAIN_NONE);
* static void soft_off(void)            给RF内核发送CMD_ABORT命令关闭RF
* static uint8_t soft_on(void)          调用rx_on函数打开RF
* static uint32_t calc_last_packet_timestamp(uint32_t rat_timestamp) 计算时间戳，用于流控

# 3 netstack_init函数
netstack_init函数位于contiki/core/net/netstack.c中，函数体并注释展开后的代码如下：
``` c
void
netstack_init(void)
{
  NETSTACK_RADIO.init();   //ieee_mode_driver.init();
  NETSTACK_RDC.init();     //contikimac_driver.init();
  NETSTACK_LLSEC.init();   //nullsec_driver.init();
  NETSTACK_MAC.init();     //csma_driver.init();
  NETSTACK_NETWORK.init(); //rime_driver.init();//IPV6 - sicslowpan_driver
}
```
其中第一个分析如下，其它几个为协议栈上的函数暂不分析，分析协议栈的时候再另起新篇吧。

# 4 ieee_mode_driver.init
这里调用的是init函数，函数内容：
``` c
/*---------------------------------------------------------------------------*/
static int
init(void)
{
  lpm_register_module(&cc26xx_rf_lpm_module);

  //调用函数HWREG(PRCM_BASE + PRCM_O_RFCMODESEL) = PRCM_RFCMODESEL_CURR_MODE5;
  //模式选择
  //这里出现的PRCM相关的宏定义位于Contiki/cpu/cc26xx-cc13xx/lib/cc26xxware/inc/hw-prcm.h,用find没找到。
  rf_core_set_modesel();  
 
  /* Initialise RX buffers */
  //初始化接收缓冲区，buf_0~3为4个缓冲区，构成循环链表
  memset(rx_buf_0, 0, RX_BUF_SIZE);
  memset(rx_buf_1, 0, RX_BUF_SIZE);
  memset(rx_buf_2, 0, RX_BUF_SIZE);
  memset(rx_buf_3, 0, RX_BUF_SIZE);

  /* Set of RF Core data queue. Circular buffer, no last entry */
  rx_data_queue.pCurrEntry = rx_buf_0;

  rx_data_queue.pLastEntry = NULL;

  /* Initialize current read pointer to first element (used in ISR) */
  //ISR中会对rx_read_entry进行处理。
  rx_read_entry = rx_buf_0;

  /* Populate the RF parameters data structure with default values */
  //RF相关参数初始化，例如：CRC、ack等。
  init_rf_params();

  //on()函数中做了很多事情：时钟的配置使能，还调用了函数init_rx_buffers();rf_core_setup_interrupts(poll_mode);rf_core_boot()；rf_radio_setup()；rx_on()；等。
  if(on() != RF_CORE_CMD_OK) {
    PRINTF("init: on() failed\n");
    return RF_CORE_CMD_ERROR;
  }

  ENERGEST_ON(ENERGEST_TYPE_LISTEN);
  
  //mode_ieee包含了两个函数soft_off和soft_on，将其配置给primary_mode变量。
  rf_core_primary_mode_register(&mode_ieee);

  check_rat_overflow(true);
  //溢出监视。
  ctimer_set(&rat_overflow_timer, RAT_OVERFLOW_PERIOD_SECONDS * CLOCK_SECOND / 2,
             handle_rat_overflow, NULL);

  //启动rf_core_process进程。
  process_start(&rf_core_process, NULL);
  return 1;
}
```
根据分析，init函数完成了接收缓冲区的清零操作，调用`init_rf_params();`函数完成参数初始化，调用on函数打开RF并配置了中断函数入口地址，回调函数定时器配置ctimer_set，最后完成启动进程rf_core_process。

# 5. rf_core_process
该进程定义于contiki/cpu/cc26xx-cc13xx/rf-core/rf_core.c文件中。函数体：
``` c
PROCESS_THREAD(rf_core_process, ev, data)
{
  int len;

  PROCESS_BEGIN();

  while(1) {
    PROCESS_YIELD_UNTIL(ev == PROCESS_EVENT_POLL);
    do {
      watchdog_periodic();
      packetbuf_clear();
      len = NETSTACK_RADIO.read(packetbuf_dataptr(), PACKETBUF_SIZE);

      if(len > 0) {
        packetbuf_set_datalen(len);

        NETSTACK_RDC.input();
      }
    } while(len > 0);
  }
  PROCESS_END();
}
```
该进程做的事情很简单，当PROCESS_EVENT_POLL事件发生的时候，调用`NETSTACK_RADIO.read`函数读取接收缓冲区数据，并调用`NETSTACK_RDC.input`函数通知上层协议。如果不需要协议层的支持，在这个process中调用自定义函数触发数据处理进程即可。PROCESS_EVENT_POLL事件是执行process_poll函数时触发的。上文分析在init函数中调用的on函数里调用了`rf_core_setup_interrupts(poll_mode);`最终执行如下代码：
``` c
HWREG(RFC_DBELL_NONBUF_BASE + RFC_DBELL_O_RFCPEIEN) = ENABLED_IRQS_POLL_MODE;
```
看来这里只定义了poll_mode模式，还以为在中断里调用了poll呢，那么就搜索吧，执行`find`命令：
```
user@instant-contiki:~/contiki$ find -name "*.c" | xargs grep  "process_poll(&rf_core_process)"
./cpu/cc26xx-cc13xx/rf-core/rf-core.c:    process_poll(&rf_core_process);
./cpu/cc26xx-cc13xx/rf-core/prop-mode.c:      process_poll(&rf_core_process);
./cpu/cc26xx-cc13xx/rf-core/ieee-mode.c:        process_poll(&rf_core_process);
```
原来只是没找到,rf-core.c中是在函数`void cc26xx_rf_cpe0_isr(void)`中调用process_poll的，而这个函数正是中断函数。  
ieee-mode.c中是在pending_packet函数中调用的，遍历接收缓冲区时，当存在缓冲区状态为DATA_ENTRY_STATUS_FINISHED时调用poll。触发数据接收。  
prop-mode.c和ieee-mode.c是并列文件，其中的process_poll函数同样是在pending_start函数中调用的。
# 6. 自定义配置
如果想自定义配置一些参数、功能，需要对RF部分的参数有清除的掌握才行。而这些参数的定义在文件cpu/cc26xx-cc13xx/rf-core/api/ieee-cmd.h中。
# 6.1 CMD_IEEE_RX命令参数
``` c
#define CMD_IEEE_RX                                             0x2801
struct __RFC_STRUCT rfc_CMD_IEEE_RX_s {
   uint16_t commandNo;                  //!<        The command ID number 0x2801
   uint16_t status;                     //!< \brief An integer telling the status of the command. This value is
                                        //!<        updated by the radio CPU during operation and may be read by the
                                        //!<        system CPU at any time.
   rfc_radioOp_t *pNextOp;              //!<        Pointer to the next operation to run after this operation is done
   ratmr_t startTime;                   //!<        Absolute or relative start time (depending on the value of <code>startTrigger</code>)
   struct {
      uint8_t triggerType:4;            //!<        The type of trigger
      uint8_t bEnaCmd:1;                //!< \brief 0: No alternative trigger command<br>
                                        //!<        1: CMD_TRIGGER can be used as an alternative trigger
      uint8_t triggerNo:2;              //!<        The trigger number of the CMD_TRIGGER command that triggers this action
      uint8_t pastTrig:1;               //!< \brief 0: A trigger in the past is never triggered, or for start of commands, give an error<br>
                                        //!<        1: A trigger in the past is triggered as soon as possible
   } startTrigger;                      //!<        Identification of the trigger that starts the operation
   struct {
      uint8_t rule:4;                   //!<        Condition for running next command: Rule for how to proceed
      uint8_t nSkip:4;                  //!<        Number of skips if the rule involves skipping
   } condition;
   uint8_t channel;                     //!< \brief Channel to tune to in the start of the operation<br>
                                        //!<        0: Use existing channel<br>
                                        //!<        11&ndash;26: Use as IEEE 802.15.4 channel, i.e. frequency is (2405 + 5 &times; (channel - 11)) MHz<br>
                                        //!<        60&ndash;207: Frequency is  (2300 + channel) MHz<br>
                                        //!<        Others: <i>Reserved</i>
   struct {
      uint8_t bAutoFlushCrc:1;          //!<        If 1, automatically remove packets with CRC error from Rx queue
      uint8_t bAutoFlushIgn:1;          //!<        If 1, automatically remove packets that can be ignored according to frame filtering from Rx queue
      uint8_t bIncludePhyHdr:1;         //!<        If 1, include the received PHY header field in the stored packet; otherwise discard it
      uint8_t bIncludeCrc:1;            //!<        If 1, include the received CRC field in the stored packet; otherwise discard it
      uint8_t bAppendRssi:1;            //!<        If 1, append an RSSI byte to the packet in the Rx queue
      uint8_t bAppendCorrCrc:1;         //!<        If 1, append a correlation value and CRC result byte to the packet in the Rx queue
      uint8_t bAppendSrcInd:1;          //!<        If 1, append an index from the source matching algorithm
      uint8_t bAppendTimestamp:1;       //!<        If 1, append a timestamp to the packet in the Rx queue
   } rxConfig;
   dataQueue_t* pRxQ;                   //!<        Pointer to receive queue
   rfc_ieeeRxOutput_t *pOutput;         //!<        Pointer to output structure (NULL: Do not store results)
   struct {
      uint16_t frameFiltEn:1;           //!< \brief 0: Disable frame filtering<br>
                                        //!<        1: Enable frame filtering
      uint16_t frameFiltStop:1;         //!< \brief 0: Receive all packets to the end<br>
                                        //!<        1: Stop receiving frame once frame filtering has caused the frame to be rejected.
      uint16_t autoAckEn:1;             //!< \brief 0: Disable auto ACK<br>
                                        //!<        1: Enable auto ACK.
      uint16_t slottedAckEn:1;          //!< \brief 0: Non-slotted ACK<br>
                                        //!<        1: Slotted ACK.
      uint16_t autoPendEn:1;            //!< \brief 0: Auto-pend disabled<br>
                                        //!<        1: Auto-pend enabled
      uint16_t defaultPend:1;           //!<        The value of the pending data bit in auto ACK packets that are not subject to auto-pend
      uint16_t bPendDataReqOnly:1;      //!< \brief 0: Use auto-pend for any packet<br>
                                        //!<        1: Use auto-pend for data request packets only
      uint16_t bPanCoord:1;             //!< \brief 0: Device is not PAN coordinator<br>
                                        //!<        1: Device is PAN coordinator
      uint16_t maxFrameVersion:2;       //!<        Reject frames where the frame version field in the FCF is greater than this value
      uint16_t fcfReservedMask:3;       //!<        Value to be AND-ed with the reserved part of the FCF; frame rejected if result is non-zero
      uint16_t modifyFtFilter:2;        //!< \brief Treatment of MSB of frame type field before frame-type filtering:<br>
                                        //!<        0: No modification<br>
                                        //!<        1: Invert MSB<br>
                                        //!<        2: Set MSB to 0<br>
                                        //!<        3: Set MSB to 1
      uint16_t bStrictLenFilter:1;      //!< \brief 0: Accept acknowledgement frames of any length >= 5<br>
                                        //!<        1: Accept only acknowledgement frames of length 5
   } frameFiltOpt;                      //!<        Frame filtering options
   struct {
      uint8_t bAcceptFt0Beacon:1;       //!< \brief Treatment of frames with frame type 000 (beacon):<br>
                                        //!<        0: Reject<br>
                                        //!<        1: Accept
      uint8_t bAcceptFt1Data:1;         //!< \brief Treatment of frames with frame type 001 (data):<br>
                                        //!<        0: Reject<br>
                                        //!<        1: Accept
      uint8_t bAcceptFt2Ack:1;          //!< \brief Treatment of frames with frame type 010 (ACK):<br>
                                        //!<        0: Reject, unless running ACK receive command<br>
                                        //!<        1: Always accept
      uint8_t bAcceptFt3MacCmd:1;       //!< \brief Treatment of frames with frame type 011 (MAC command):<br>
                                        //!<        0: Reject<br>
                                        //!<        1: Accept
      uint8_t bAcceptFt4Reserved:1;     //!< \brief Treatment of frames with frame type 100 (reserved):<br>
                                        //!<        0: Reject<br>
                                        //!<        1: Accept
      uint8_t bAcceptFt5Reserved:1;     //!< \brief Treatment of frames with frame type 101 (reserved):<br>
                                        //!<        0: Reject<br>
                                        //!<        1: Accept
      uint8_t bAcceptFt6Reserved:1;     //!< \brief Treatment of frames with frame type 110 (reserved):<br>
                                        //!<        0: Reject<br>
                                        //!<        1: Accept
      uint8_t bAcceptFt7Reserved:1;     //!< \brief Treatment of frames with frame type 111 (reserved):<br>
                                        //!<        0: Reject<br>
                                        //!<        1: Accept
   } frameTypes;                        //!<        Frame types to receive in frame filtering
   struct {
      uint8_t ccaEnEnergy:1;            //!<        Enable energy scan as CCA source
      uint8_t ccaEnCorr:1;              //!<        Enable correlator based carrier sense as CCA source
      uint8_t ccaEnSync:1;              //!<        Enable sync found based carrier sense as CCA source
      uint8_t ccaCorrOp:1;              //!< \brief Operator to use between energy based and correlator based CCA<br>
                                        //!<        0: Report busy channel if either ccaEnergy or ccaCorr are busy<br>
                                        //!<        1: Report busy channel if both ccaEnergy and ccaCorr are busy
      uint8_t ccaSyncOp:1;              //!< \brief Operator to use between sync found based CCA and the others<br>
                                        //!<        0: Always report busy channel if ccaSync is busy<br>
                                        //!<        1: Always report idle channel if ccaSync is idle
      uint8_t ccaCorrThr:2;             //!<        Threshold for number of correlation peaks in correlator based carrier sense
   } ccaOpt;                            //!<        CCA options
   int8_t ccaRssiThr;                   //!<        RSSI threshold for CCA
   uint8_t __dummy0;
   uint8_t numExtEntries;               //!<        Number of extended address entries
   uint8_t numShortEntries;             //!<        Number of short address entries
   uint32_t* pExtEntryList;             //!<        Pointer to list of extended address entries
   rfc_shortAddrEntry_t *pShortEntryList;//!<        Pointer to list of short address entries
   uint64_t localExtAddr;               //!<        The extended address of the local device
   uint16_t localShortAddr;             //!<        The short address of the local device
   uint16_t localPanID;                 //!<        The PAN ID of the local device
   uint16_t __dummy1;
   uint8_t __dummy2;
   struct {
      uint8_t triggerType:4;            //!<        The type of trigger
      uint8_t bEnaCmd:1;                //!< \brief 0: No alternative trigger command<br>
                                        //!<        1: CMD_TRIGGER can be used as an alternative trigger
      uint8_t triggerNo:2;              //!<        The trigger number of the CMD_TRIGGER command that triggers this action
      uint8_t pastTrig:1;               //!< \brief 0: A trigger in the past is never triggered, or for start of commands, give an error<br>
                                        //!<        1: A trigger in the past is triggered as soon as possible
   } endTrigger;                        //!<        Trigger that causes the device to end the Rx operation
   ratmr_t endTime;                     //!< \brief Time used together with <code>endTrigger</code> that causes the device to end the Rx
                                        //!<        operation
};
```
参数配置函数：
``` c
static void
init_rf_params(void)
{
  rfc_CMD_IEEE_RX_t *cmd = (rfc_CMD_IEEE_RX_t *)cmd_ieee_rx_buf;

  memset(cmd_ieee_rx_buf, 0x00, RF_CMD_BUFFER_SIZE);

  cmd->commandNo = CMD_IEEE_RX;
  cmd->status = RF_CORE_RADIO_OP_STATUS_IDLE;
  cmd->pNextOp = NULL;
  cmd->startTime = 0x00000000;
  cmd->startTrigger.triggerType = TRIG_NOW;
  cmd->condition.rule = COND_NEVER;
  cmd->channel = RF_CORE_CHANNEL;

  cmd->rxConfig.bAutoFlushCrc = 1;
  cmd->rxConfig.bAutoFlushIgn = 0;
  cmd->rxConfig.bIncludePhyHdr = 0;
  cmd->rxConfig.bIncludeCrc = 1;
  cmd->rxConfig.bAppendRssi = 1;
  cmd->rxConfig.bAppendCorrCrc = 1;
  cmd->rxConfig.bAppendSrcInd = 0;
  cmd->rxConfig.bAppendTimestamp = 1;

  cmd->pRxQ = &rx_data_queue;
  cmd->pOutput = (rfc_ieeeRxOutput_t *)rf_stats;

#if IEEE_MODE_PROMISCOUS
  cmd->frameFiltOpt.frameFiltEn = 0;
#else
  cmd->frameFiltOpt.frameFiltEn = 1;
#endif

  cmd->frameFiltOpt.frameFiltStop = 1;

#if IEEE_MODE_AUTOACK
  cmd->frameFiltOpt.autoAckEn = 1;
#else
  cmd->frameFiltOpt.autoAckEn = 0;
#endif

  cmd->frameFiltOpt.slottedAckEn = 0;
  cmd->frameFiltOpt.autoPendEn = 0;
  cmd->frameFiltOpt.defaultPend = 0;
  cmd->frameFiltOpt.bPendDataReqOnly = 0;
  cmd->frameFiltOpt.bPanCoord = 0;
  cmd->frameFiltOpt.maxFrameVersion = 2;
  cmd->frameFiltOpt.bStrictLenFilter = 0;

  /* Receive all frame types */
  cmd->frameTypes.bAcceptFt0Beacon = 1;
  cmd->frameTypes.bAcceptFt1Data = 1;
  cmd->frameTypes.bAcceptFt2Ack = 1;
  cmd->frameTypes.bAcceptFt3MacCmd = 1;
  cmd->frameTypes.bAcceptFt4Reserved = 1;
  cmd->frameTypes.bAcceptFt5Reserved = 1;
  cmd->frameTypes.bAcceptFt6Reserved = 1;
  cmd->frameTypes.bAcceptFt7Reserved = 1;

  /* Configure CCA settings */
  cmd->ccaOpt.ccaEnEnergy = 1;
  cmd->ccaOpt.ccaEnCorr = 1;
  cmd->ccaOpt.ccaEnSync = 1;
  cmd->ccaOpt.ccaCorrOp = 1;
  cmd->ccaOpt.ccaSyncOp = 0;
  cmd->ccaOpt.ccaCorrThr = 3;

  cmd->ccaRssiThr = IEEE_MODE_RSSI_THRESHOLD;

  cmd->numExtEntries = 0x00;
  cmd->numShortEntries = 0x00;
  cmd->pExtEntryList = 0;
  cmd->pShortEntryList = 0;

  cmd->endTrigger.triggerType = TRIG_NEVER;
  cmd->endTime = 0x00000000;

  /* set address filter command */
  filter_cmd.commandNo = CMD_IEEE_MOD_FILT;
  memcpy(&filter_cmd.newFrameFiltOpt, &cmd->frameFiltOpt, sizeof(cmd->frameFiltOpt));
  memcpy(&filter_cmd.newFrameTypes, &cmd->frameTypes, sizeof(cmd->frameTypes));
}
```

## 6.2 CMD_IEEE_TX命令参数
数据结构如下：
``` c
#define CMD_IEEE_TX                                             0x2C01
struct __RFC_STRUCT rfc_CMD_IEEE_TX_s {
   uint16_t commandNo;                  //!<        The command ID number 0x2C01
   uint16_t status;                     //!< \brief An integer telling the status of the command. This value is
                                        //!<        updated by the radio CPU during operation and may be read by the
                                        //!<        system CPU at any time.
   rfc_radioOp_t *pNextOp;              //!<        Pointer to the next operation to run after this operation is done
   ratmr_t startTime;                   //!<        Absolute or relative start time (depending on the value of <code>startTrigger</code>)
   struct {
      uint8_t triggerType:4;            //!<        The type of trigger
      uint8_t bEnaCmd:1;                //!< \brief 0: No alternative trigger command<br>
                                        //!<        1: CMD_TRIGGER can be used as an alternative trigger
      uint8_t triggerNo:2;              //!<        The trigger number of the CMD_TRIGGER command that triggers this action
      uint8_t pastTrig:1;               //!< \brief 0: A trigger in the past is never triggered, or for start of commands, give an error<br>
                                        //!<        1: A trigger in the past is triggered as soon as possible
   } startTrigger;                      //!<        Identification of the trigger that starts the operation
   struct {
      uint8_t rule:4;                   //!<        Condition for running next command: Rule for how to proceed
      uint8_t nSkip:4;                  //!<        Number of skips if the rule involves skipping
   } condition;
   struct {
      uint8_t bIncludePhyHdr:1;         //!< \brief 0: Find PHY header automatically<br>
                                        //!<        1: Insert PHY header from the buffer
      uint8_t bIncludeCrc:1;            //!< \brief 0: Append automatically calculated CRC<br>
                                        //!<        1: Insert FCS (CRC) from the buffer
      uint8_t :1;
      uint8_t payloadLenMsb:5;          //!< \brief Most significant bits of payload length. Should only be non-zero to create long
                                        //!<        non-standard packets for test purposes
   } txOpt;
   uint8_t payloadLen;                  //!<        Number of bytes in the payload
   uint8_t* pPayload;                   //!<        Pointer to payload buffer of size <code>payloadLen</code>
   ratmr_t timeStamp;                   //!<        Time stamp of transmitted frame
};

```


# 7. 下一步
在代码分析中有两个东西直接忽略掉了，下一步需要补上。  
* LPM 相关
* ENERGEST相关

