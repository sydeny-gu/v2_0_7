8189FTV SDIO Interface 基于Linux SDIO Structure 开发

编译：
----------------------
工程位置:project/stmicro_stm32f2xx_freertos_networking_8189em_8189fm_SDIO

编译目标:stm32f2xx sdio host driver
编译工程:Sdiolib.uvproj
编译设置：#define WIFI_INTERFACE	WIFI_INTERFACE_SDIO   in project/stmicro_stm32f2xx_freertos_networking_8189em_8189fm_SDIO/inc/stm32_platform.h        

编译目标：8189FTV sdio wifi driver
编译工程：Wlanlib_8189fm.uvproj
编译设置：打开#define CONFIG_SDIO_HCI 关闭CONFIG_GSPI_HCI	  in component/common/drivers/wlan/realtek/include/autoconf.h


代码结构
----------------------
1.code对SDIO BUS Interface 进行了封装，接口为SDIO_BUS_OPS rtw_sdio_bus_ops，结构如下：

typedef struct _SDIO_BUS_OPS{
	int (*bus_probe)(void);
	int (*bus_remove)(void);
	int (*enable)(struct sdio_func*func);	/*enables a SDIO function for usage*/
	int (*disable)(struct sdio_func *func); 
	int (*reg_driver)(struct sdio_driver*); /*register sdio function device driver callback*/
	void (*unreg_driver)(struct sdio_driver *); 
	int (*claim_irq)(struct sdio_func *func, void(*handler)(struct sdio_func *));
	int (*release_irq)(struct sdio_func*func);
	void (*claim_host)(struct sdio_func*func);	/*exclusively claim a bus for a certain SDIO function*/
	void (*release_host)(struct sdio_func *func); 
	unsigned char (*readb)(struct sdio_func *func, unsigned int addr, int *err_ret);/*read a single byte from a SDIO function*/
	unsigned short (*readw)(struct sdio_func *func, unsigned int addr, int *err_ret);	/*read a 16 bit integer from a SDIO function*/
	unsigned int (*readl)(struct sdio_func *func, unsigned int addr, int *err_ret); /*read a 32 bit integer from a SDIO function*/
	void (*writeb)(struct sdio_func *func, unsigned char b,unsigned int addr, int *err_ret);	/*write a single byte to a SDIO function*/
	void (*writew)(struct sdio_func *func, unsigned short b,unsigned int addr, int *err_ret);	/*write a 16 bit integer to a SDIO function*/
	void (*writel)(struct sdio_func *func, unsigned int b,unsigned int addr, int *err_ret); /*write a 32 bit integer to a SDIO function*/
	int (*memcpy_fromio)(struct sdio_func *func, void *dst,unsigned int addr, int count);/*read a chunk of memory from a SDIO functio*/
	int (*memcpy_toio)(struct sdio_func *func, unsigned int addr,void *src, int count);  /*write a chunk of memory to a SDIO function*/ 
}SDIO_BUS_OPS;

其中与wifi 通信的接口有：
	int (*claim_irq)(struct sdio_func *func, void(*handler)(struct sdio_func *));
	int (*release_irq)(struct sdio_func*func);
	void (*claim_host)(struct sdio_func*func);	
	void (*release_host)(struct sdio_func *func); 
	unsigned char (*readb)(struct sdio_func *func, unsigned int addr, int *err_ret);
	unsigned short (*readw)(struct sdio_func *func, unsigned int addr, int *err_ret);	
	unsigned int (*readl)(struct sdio_func *func, unsigned int addr, int *err_ret);
	void (*writeb)(struct sdio_func *func, unsigned char b,unsigned int addr, int *err_ret);	
	void (*writew)(struct sdio_func *func, unsigned short b,unsigned int addr, int *err_ret);
	void (*writel)(struct sdio_func *func, unsigned int b,unsigned int addr, int *err_ret); 
	int (*memcpy_fromio)(struct sdio_func *func, void *dst,unsigned int addr, int count);/*read a chunk of memory from a SDIO functio*/
	int (*memcpy_toio)(struct sdio_func *func, unsigned int addr,void *src, int count);  /*write a chunk of memory to a SDIO function*/ 

分别与linux kernel code中
	sdio_claim_irq,
	sdio_release_irq,
	sdio_claim_host,
	sdio_release_host,
	sdio_readb,
	sdio_readw,
	sdio_readl,
	sdio_writeb,
	sdio_writew,
	sdio_writel,
	sdio_memcpy_fromio,
	sdio_memcpy_toio,
对应

2.code对 sdio_func的结构做了修改，结构如下

struct sdio_func {
	struct mmc_card		*card;		/* the card this device belongs to */
	void	(*irq_handler)(struct sdio_func *); /* IRQ callback */

	unsigned	int	max_blksize;	/* maximum block size */ 
	unsigned	int	cur_blksize;	/* current block size */	
	unsigned	int	enable_timeout;	/* max enable timeout in msec */ 
	unsigned int	num;		/* function number *///add
	unsigned short		vendor;		/* vendor id */ //add
	unsigned short		device;		/* device id */ //add
	unsigned		num_info;	/* number of info strings */ 
	const char		**info;		/* info strings */ //add
	unsigned char		class;		/* standard interface class */

	unsigned int			tmpbuf_reserved; //for tmpbuf 4 byte alignment
	unsigned char			tmpbuf[4];	/* DMA:able scratch buffer */

#ifdef CONFIG_READ_CIS
	struct sdio_func_tuple *tuples;
#endif
	void *drv_priv;
}

linux kernel中 sdio_func 结构体：
struct sdio_func {
	struct mmc_card		*card;		/* the card this device belongs to */
	struct device		dev;		/* the device */
	sdio_irq_handler_t	*irq_handler;	/* IRQ callback */
	unsigned int		num;		/* function number */

	unsigned char		class;		/* standard interface class */
	unsigned short		vendor;		/* vendor id */
	unsigned short		device;		/* device id */

	unsigned		max_blksize;	/* maximum block size */
	unsigned		cur_blksize;	/* current block size */

	unsigned		enable_timeout;	/* max enable timeout in msec */

	unsigned int		state;		/* function state */
#define SDIO_STATE_PRESENT	(1<<0)		/* present in sysfs */

	u8			tmpbuf[4];	/* DMA:able scratch buffer */

	unsigned		num_info;	/* number of info strings */
	const char		**info;		/* info strings */

	struct sdio_func_tuple *tuples;
};

与linux kernel相比，做了精简，去除struct device dev，添加void *drv_priv，两者功能相同：让SDIO Host获取wifi adapter 对象。

由于不同客户使用的SDIO Host存在差异，使用8189FTV Wlan_lib for SDIO时需要的操作有：
1、component/common/drivers/sdio/wifi_io.h 需要release给客户
2、SDIO Host driver 必须根据SDIO_BUS_OPS接口创建SDIO_BUS_OPS rtw_sdio_bus_ops 对象，该对象必须实现与wifi 通信相关的接口（12个：claim_irq、release_irq、claim_host....、writel）；
2、SDIO Host driver 必须根据struct sdio_func创建struct sdio_func *wifi_sdio_func 对象。

