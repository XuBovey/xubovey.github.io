---
title: Contiki Rime协议分析-abc
toc: true
date: 2016-11-02 16:37:54
categories:
- Contiki
- Rime
tags:
- Contiki
---

# 0 背景
在研究rime过程中使用了example中的rime文件夹下的例程，中间遇到的问题进行记录。

<!--more-->

# 1 make出错
在rime例程文件夹下执行如下命令，以错误退出。
``` c
make TARGET=srf06-cc26xx BOARD=srf06/cc26xx
```
后来分析原因是里面有多个历程，复制rime到新的文件夹，将没用的c文件删除，哪些没用？可以看makefile文件中`all:`后面的名称对应的c文件，除了example-abc.c外的所有文件。

# 2 接收不到数据
接收方式：开发板+仿真器+smartrf radio 2.4.3
可以接收到数据但是有溢出警告。
警告没关系，看看数据对不对：
* 发送的内容：`Hello`  
* 对应的HEX：`48 65 65 60 6f`  

smartrf radio接收到的十六进制数据：  
```
16:58:33.737 | 16792 | 0e cd ab ff ff 03 3b 80 00 48 65 6c 6c 6f 00  |  -39
```
可以发现其中包含`48 65 65 60 6f`，看来数据正确，只是两种格式不一样而已，所以报错的。

如何才能正确显示呢？
很简单，两个板子都烧写同样的程序就好了。

# 3 example-abc数据怎么出去的
要说过程的话还是这个图片介绍的清楚：[转载](http://blog.sina.com.cn/s/blog_686ee2910102vvf9.html)  

![](http://oefaano2o.bkt.clouddn.com/blogimages/images/161102-Contiki-rime-abc/161102-Contiki-rime-abc-01.png-blog)
>如图2所示，描述了Contiki中整个无线传输链路基本结构，主要包含了：网络层（NETSTACK_NETWORK）、链路层安全驱动（link layer security driver, NETSTACK_LLSEC）、MAC层（Media Access Control, NETSTACK_MAC）,RDC层（Radio Duty Cycling，NETSTACK_RDC）以及物理层（NETSTACK_RADIO）。  
在图2的左边一列为对应不同层次，对应需要实现的驱动接口结构，右边为各个层次之间相互的调用情况（只提取了最重要的部分，相关处理没有细化）。  
其中网络层主要完成相关无线网络的连接、组网等情况，链路层安全驱动用于保证整个传输链路的数据安全，MAC层完成物理媒介的访问控制，RDC层用于控制无线电电路的开断，可以有效的控制无线电部分的耗电情况，RADIO层实现了不同无线媒介底层访问驱动，完成整个无线链路的访问与操作。  

## 3.1 函数调用过程跟踪：  
abc_send ->   
rime_output(&c->channel); ->  
NETSTACK_LLSEC.send(packet_sent, c); (nullsec_driver.send) =  
nullsec_driver.send(packet_sent, c); ->  
NETSTACK_MAC.send(sent, ptr); (csma_driver) =  
csma_driver.send(sent, ptr); =  
send_packet(sent, ptr); ->  
schedule_transmission(n); ->  
//这里用到ctiemr  
ctimer_set(&n->transmit_timer, delay, transmit_packet_list, n); ->  
transmit_packet_list(void *ptr) ->  
NETSTACK_RDC.send_list(packet_sent, n, q); (contikimac_driver) =  
contikimac_driver.qsend_list(packet_sent, n, q); ->  
ret = send_packet(sent, ptr, curr, is_receiver_awake); ->  
NETSTACK_RADIO.transmit(transmit_len); ->  
ret = rf_core_send_cmd((uint32_t)&cmd, &cmd_status);

## 3.2 数据传递过程：
packetbuf.c文件中定义有如下变量

``` c
struct packetbuf_attr packetbuf_attrs[PACKETBUF_NUM_ATTRS];
struct packetbuf_addr packetbuf_addrs[PACKETBUF_NUM_ADDRS];
static uint32_t packetbuf_aligned[(PACKETBUF_SIZE + 3) / 4];
static uint8_t *packetbuf = (uint8_t *)packetbuf_aligned;
```
# 4 abc
``` c
static struct abc_conn abc;
abc_open(&abc, 128, &abc_call);
packetbuf_copyfrom("Hello", 6);
abc_send(&abc);
```
其中abc_conn数据结构如下
``` c
struct abc_conn {
  struct channel channel;
  const struct abc_callbacks *u;
};
```
* abc_open(&abc, 128, &abc_call);
abc_open使能了128通道，并将abc的通道添加到channel_list上；  
并设置abc_call为abc的回调函数。
* packetbuf_copyfrom("Hello", 6);  
此函数将Hello拷贝到*packetbuf，即packetbuf_aligned内。
* abc_send(&abc);
函数中调用语句：
  ``` c
  rime_output(&c->channel);
  ```
* rime_output中调用函数`NETSTACK_LLSEC.send(packet_sent, c);`  

可以看到这些参数传递是通过`struct abc_conn abc;`进行的。

# 5 llsec - Link layer security 
驱动：
``` c
const struct llsec_driver nullsec_driver = {
  "nullsec",
  init,
  send,
  input
};
```
这里没有用到该层功能，所以调用过程只做参数传递。
NETSTACK_LLSEC.send = nullsec_driver.send，而Send函数如下   
``` c
static void
send(mac_callback_t sent, void *ptr)
{
  //实际执行语句 packetbuf_attrs[type].val = val;
  packetbuf_set_attr(PACKETBUF_ATTR_FRAME_TYPE, FRAME802154_DATAFRAME);
  NETSTACK_MAC.send(sent, ptr);
}
```

# 6 Carrier Sense Multiple Access (CSMA) MAC
驱动：
``` c
const struct mac_driver csma_driver = {
  "CSMA",
  init,
  send_packet,
  input_packet,
  on,
  off,
  channel_check_interval,
};
```
NETSTACK_MAC.send = csma_driver.send_packet  

## 6.1 mac层send_packet函数解析 
send_packet函数代码结构如下：
``` c
static void
send_packet(mac_callback_t sent, void *ptr)
{
  struct rdc_buf_list *q;
  struct neighbor_queue *n;
  static uint8_t initialized = 0;
  static uint16_t seqno;
  const linkaddr_t *addr = packetbuf_addr(PACKETBUF_ADDR_RECEIVER);

  if(!initialized) {
    initialized = 1;
    /* Initialize the sequence number to a random value as per 802.15.4. */
    seqno = random_rand();
  }

  if(seqno == 0) {
    /* PACKETBUF_ATTR_MAC_SEQNO cannot be zero, due to a pecuilarity
       in framer-802154.c. */
    seqno++;
  }
  packetbuf_set_attr(PACKETBUF_ATTR_MAC_SEQNO, seqno++);

  /* Look for the neighbor entry */
  n = neighbor_queue_from_addr(addr);
  if(n == NULL) {
    /* Allocate a new neighbor entry */
    n = memb_alloc(&neighbor_memb);
    ...
  }

  if(n != NULL) {
    /* Add packet to the neighbor's queue */
    ...
  } else {
    PRINTF("csma: could not allocate neighbor, dropping packet\n");
  }
  mac_call_sent_callback(sent, ptr, MAC_TX_ERR, 1);
}
```
数据结构：  
rdc_buf_list
``` c
/* List of packets to be sent by RDC layer */
struct rdc_buf_list {
  struct rdc_buf_list *next;
  struct queuebuf *buf;
  void *ptr;
};
```
neighbor_queue
``` c
/* Every neighbor has its own packet queue */
struct neighbor_queue {
  struct neighbor_queue *next;
  linkaddr_t addr;
  struct ctimer transmit_timer;
  uint8_t transmissions;
  uint8_t collisions;
  LIST_STRUCT(queued_packet_list);
  /*
  #define LIST_STRUCT(name) \
         void *LIST_CONCAT(name,_list); \
         list_t name 
  typedef void ** list_t;
  ==>
  LIST_STRUCT(queued_packet_list); = 
  void *  queued_packet_list_list; 
  void ** queued_packet_list;//这里等价与链表中的next*/
};
```
* const linkaddr_t *addr = packetbuf_addr(PACKETBUF_ADDR_RECEIVER);
    ``` c
    typedef union {
    unsigned char u8[LINKADDR_SIZE];
    #if LINKADDR_SIZE == 2
    uint16_t u16;
    #endif /* LINKADDR_SIZE == 2 */
    } linkaddr_t;
    ```
    packetbuf_addr函数执行语句
    ``` c
    return &packetbuf_addrs[type - PACKETBUF_ADDR_FIRST].addr;
    //type = PACKETBUF_ADDR_RECEIVER
    //#define PACKETBUF_ADDR_FIRST PACKETBUF_ADDR_SENDER
    //PACKETBUF_ADDR_SENDER = PACKETBUF_ADDR_RECEIVER + 1
    //这里等效为&packetbuf_addrs[1].addr
    ```
    packetbuf.c中定义
    ``` c
    struct packetbuf_addr packetbuf_addrs[PACKETBUF_NUM_ADDRS];
    //PACKETBUF_NUM_ADDRS = 4或2
    ```
    这里是这样的
    ``` c
    struct packetbuf_addr {
    linkaddr_t addr;
    };
    ```
    也就是说有以linkaddr_t为内容的packetbuf_addr结构体数组packetbuf_addrs，经过赋值语句`const linkaddr_t *addr = packetbuf_addr(PACKETBUF_ADDR_RECEIVER);`将接收相关的地址赋值给addr。
* LIST_STRUCT_INIT(n, queued_packet_list);
    ``` c
    #define LIST_STRUCT_INIT(struct_ptr, name)                              \
    do {                                                                \
       (struct_ptr)->name = &((struct_ptr)->LIST_CONCAT(name,_list));   \
       (struct_ptr)->LIST_CONCAT(name,_list) = NULL;                    \
       list_init((struct_ptr)->name);                                   \
    } while(0)

    void
    list_init(list_t list)
    {
    *list = NULL;
    }
    ```
    ==>
    ``` c
    n->queued_packet_list = &(n->queued_packet_list_list); //= next
    n->queued_packet_list_list = NULL;
    * n->queued_packet_list = NUL;
    ```

* packetbuf_set_attr(PACKETBUF_ATTR_MAC_SEQNO, seqno++);

    ``` c
    int
    packetbuf_set_attr(uint8_t type, const packetbuf_attr_t val)
    {
    packetbuf_attrs[type].val = val;
    return 1;
    }
    ```

    ``` c
    struct packetbuf_attr packetbuf_attrs[PACKETBUF_NUM_ATTRS];
    ```
    这些是对数组中变量的更新。
* neighbor
    一个设备周围的能与其通信的设备都称作邻居/临近节点。设备用链表方式记录其邻居信息，链表的数据结构如下：

    ``` c
    /* Every neighbor has its own packet queue */
    struct neighbor_queue {
    struct neighbor_queue *next;
    linkaddr_t addr;
    struct ctimer transmit_timer;
    uint8_t transmissions;
    uint8_t collisions;
    LIST_STRUCT(queued_packet_list);
    };
    ```
    还定义了一个链表查找函数`neighbor_queue_from_addr`,应用示例：
    ``` c
    /* Look for the neighbor entry */
    n = neighbor_queue_from_addr(addr); //---->
    ```
    如果邻居链表中有addr的节点则将其在链表中的位置返回，如果没有则返回NULL。同时后面的语句会根据这n值进行处理，如果为NULL则新建(memb_alloc)一个邻居节点，将addr赋值给新建的邻居节点，然后将该节点添加到邻居链表中。之后是给这个邻居分配内存空间，执行数据发送。
* packet_memb
    定义`MEMB(packet_memb, struct rdc_buf_list, MAX_QUEUED_PACKETS);`语句`q = memb_alloc(&packet_memb);`进行分配，然后将待发送数据地址放入邻居节点的rdc_buf_list中。  
    其中

    ``` c
    /* List of packets to be sent by RDC layer */
    struct rdc_buf_list {
    struct rdc_buf_list *next;
    struct queuebuf *buf;
    void *ptr;
    };
    ```
    内存分配：  
    * q->ptr = memb_alloc(&metadata_memb);  
    * q->buf = queuebuf_new_from_packetbuf();
    分配内存赋值：
    * struct qbuf_metadata *metadata = (struct qbuf_metadata *)q->ptr;
    * metadata->max_transmissions = CSMA_MAX_MAX_FRAME_RETRIES + 1;
    * metadata->sent = sent;
    * metadata->cptr = ptr; //---->

* schedule_transmission

``` c
list_add(n->queued_packet_list, q);

if(list_head(n->queued_packet_list) == q) {
              schedule_transmission(n);//---->
            }
```

``` c
static void
schedule_transmission(struct neighbor_queue *n)
{
  clock_time_t delay;
  int backoff_exponent; /* BE in IEEE 802.15.4 */

  backoff_exponent = MIN(n->collisions, CSMA_MAX_BE);

  /* Compute max delay as per IEEE 802.15.4: 2^BE-1 backoff periods  */
  delay = ((1 << backoff_exponent) - 1) * backoff_period();
  if(delay > 0) {
    /* Pick a time for next transmission */
    delay = random_rand() % delay;
  }

  PRINTF("csma: scheduling transmission in %u ticks, NB=%u, BE=%u\n",
      (unsigned)delay, n->collisions, backoff_exponent);
  ctimer_set(&n->transmit_timer, delay, transmit_packet_list, n); //---->
}
```

* transmit_packet_list

``` c
static void
transmit_packet_list(void *ptr)
{
  struct neighbor_queue *n = ptr;
  if(n) {
    struct rdc_buf_list *q = list_head(n->queued_packet_list);
    if(q != NULL) {
      PRINTF("csma: preparing number %d %p, queue len %d\n", n->transmissions, q,
          list_length(n->queued_packet_list));
      /* Send packets in the neighbor's list */
      NETSTACK_RDC.send_list(packet_sent, n, q); //---->
    }
  }
}
```

# 7  queuebuf
默认的queuebuf数量是8，自定义数量少于8则使用swap，并使用ram of cfs，如果大于等于8则不使用swap。  
数据结构、申请内存空间、接口函数如下：

``` c
/* ignore the debug and swap variable */
struct queuebuf {
    struct queuebuf_data *ram_ptr;
};

/* The actual queuebuf data */
// PACKETBUF_SIZE = 128
struct queuebuf_data {
  uint8_t data[PACKETBUF_SIZE];
  uint16_t len;
  struct packetbuf_attr attrs[PACKETBUF_NUM_ATTRS];
  struct packetbuf_addr addrs[PACKETBUF_NUM_ADDRS];
};

// Memory
// QUEUEBUF_NUM = 8, 可自定义
// QUEUEBUFRAM_NUM = 8， 根据QUEUEBUF_NUM变化
MEMB(bufmem, struct queuebuf, QUEUEBUF_NUM); //声明指针空间
MEMB(buframmem, struct queuebuf_data, QUEUEBUFRAM_NUM); //声明数据空间

void queuebuf_init(void); //bufmem、 buframmem init

// 从bufmem、 buframmem中获取1个元素
struct queuebuf *queuebuf_new_from_packetbuf(void); 

// 将b中的attrs和addrs复制到packetbuf.c中定义的attrs、addrs数组中
void queuebuf_update_attr_from_packetbuf(struct queuebuf *b);

// 1. 将b中的attrs和addrs复制到packetbuf.c中定义的attrs、addrs数组中
// 2. 将packetbuf.c中定义packetbuf中的数据复制到b中的data中
void queuebuf_update_from_packetbuf(struct queuebuf *b);
// 与上一语句功能相反
void queuebuf_to_packetbuf(struct queuebuf *b);

// 释放b占用的内存空间
void queuebuf_free(struct queuebuf *b);

// 返回b中data数组地址
void *queuebuf_dataptr(struct queuebuf *b);

// 返回b中data数组内有效数据长度
int queuebuf_datalen(struct queuebuf *b);

// 读取type类型在addrs数组中的内容
linkaddr_t *queuebuf_addr(struct queuebuf *b, uint8_t type);
// 读取type参数在attrs中的内容
packetbuf_attr_t queuebuf_attr(struct queuebuf *b, uint8_t type);
// 打印queuebuf_list信息
void queuebuf_debug_print(void);

//释放bufmem内存
int queuebuf_numfree(void);
```




# 8 RDC层 
驱动与数据结构如下

``` c
/* List of packets to be sent by RDC layer */
struct rdc_buf_list {
  struct rdc_buf_list *next;
  struct queuebuf *buf;
  void *ptr;
};

/**
 * The structure of a RDC (radio duty cycling) driver in Contiki.
 */
struct rdc_driver {
  char *name;

  /** Initialize the RDC driver */
  void (* init)(void);

  /** Send a packet from the Rime buffer  */
  void (* send)(mac_callback_t sent_callback, void *ptr);

  /** Send a packet list */
  void (* send_list)(mac_callback_t sent_callback, void *ptr, struct rdc_buf_list *list);

  /** Callback for getting notified of incoming packet. */
  void (* input)(void);

  /** Turn the MAC layer on. */
  int (* on)(void);

  /** Turn the MAC layer off. */
  int (* off)(int keep_radio_on);

  /** Returns the channel check interval, expressed in clock_time_t ticks. */
  unsigned short (* channel_check_interval)(void);
};
```
驱动分别在两个文件中都有定义nordc.c和nullrdc.h，其中nordc中的驱动函数均是空函数。所以来分析nullrdc中的函数，驱动如下：  

``` c
// in nullrdc.c
const struct rdc_driver nullrdc_driver = {
  "nullrdc",
  init,
  send_packet,
  send_list,
  packet_input,
  on,
  off,
  channel_check_interval,
};
```
## 8.1 send_list
send_list函数比较简单函数体如下：
``` c
//NETSTACK_RDC.send_list(packet_sent, n, q); //---->
static void
send_list(mac_callback_t sent, void *ptr, struct rdc_buf_list *buf_list)
{
  while(buf_list != NULL) {
    /* We backup the next pointer, as it may be nullified by
     * mac_call_sent_callback() */
    struct rdc_buf_list *next = buf_list->next;
    int last_sent_ok;

    queuebuf_to_packetbuf(buf_list->buf); //data数组复制到packetbuf数组中
    last_sent_ok = send_one_packet(sent, ptr); //---->

    /* If packet transmission was not successful, we should back off and let
     * upper layers retransmit, rather than potentially sending out-of-order
     * packet fragments. */
    if(!last_sent_ok) {
      return;
    }
    buf_list = next;
  }
}
```
## 8.2 send_one_packet
该函数实现了封包和发送两件事； 
* NETSTACK_FRAME.create完成组包 
* NETSTACK_RADIO.prepare将数据从packetbuf搬移到ieee-mode.c中定义的tx_buf
* NETSTACK_RADIO.send完成发送

## 8.3 NETSTACK_FRAME.create

``` c
#define NETSTACK_FRAME framer_802154
const struct framer framer_802154 = {
  hdr_length,
  create,
  parse
};
```
执行create后会完成如下数据结构的内容：  
fcf = Frame control field 帧控制
``` c
typedef struct {
  /* The fields dest_addr and src_addr must come first to ensure they are aligned to the
   * CPU word size. Needed as they are accessed directly as linkaddr_t*. Note we cannot use
   * the type linkaddr_t directly here, as we always need 8 bytes, not LINKADDR_SIZE bytes. */
  uint8_t dest_addr[8];           /**< Destination address */
  uint8_t src_addr[8];            /**< Source address */
  frame802154_fcf_t fcf;          /**< Frame control field  */
  uint8_t seq;                    /**< Sequence number */
  uint16_t dest_pid;              /**< Destination PAN ID */
  uint16_t src_pid;               /**< Source PAN ID */
  frame802154_aux_hdr_t aux_hdr;  /**< Aux security header */
  uint8_t *payload;               /**< Pointer to 802.15.4 payload */
  int payload_len;                /**< Length of payload field */
} frame802154_t;

typedef struct {
  uint8_t frame_type;        /**< 3 bit. Frame type field, see 802.15.4 */
  uint8_t security_enabled;  /**< 1 bit. True if security is used in this frame */
  uint8_t frame_pending;     /**< 1 bit. True if sender has more data to send */
  uint8_t ack_required;      /**< 1 bit. Is an ack frame required? */
  uint8_t panid_compression; /**< 1 bit. Is this a compressed header? */
  /*   uint8_t reserved; */  /**< 1 bit. Unused bit */
  uint8_t sequence_number_suppression; /**< 1 bit. Does the header omit sequence number?, see 802.15.4e */
  uint8_t ie_list_present;   /**< 1 bit. Does the header contain Information Elements?, see 802.15.4e */
  uint8_t dest_addr_mode;    /**< 2 bit. Destination address mode, see 802.15.4 */
  uint8_t frame_version;     /**< 2 bit. 802.15.4 frame version */
  uint8_t src_addr_mode;     /**< 2 bit. Source address mode, see 802.15.4 */
} frame802154_fcf_t;

/** \brief 802.15.4 Aux security header */
typedef struct {
  frame802154_scf_t security_control;        /**< Security control bitfield */
  frame802154_frame_counter_t frame_counter; /**< Frame counter, used for security */
  frame802154_key_source_t key_source;       /**< Key Source subfield */
  uint8_t key_index;                         /**< Key Index subfield */
} frame802154_aux_hdr_t;
```
## 8.4 NETSTACK_RADIO.send



