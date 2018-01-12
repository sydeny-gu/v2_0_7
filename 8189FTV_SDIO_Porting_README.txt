8189FTV SDIO Interface ����Linux SDIO Structure ����

���룺
----------------------
����λ��:project/stmicro_stm32f2xx_freertos_networking_8189em_8189fm_SDIO

����Ŀ��:stm32f2xx sdio host driver
���빤��:Sdiolib.uvproj
�������ã�#define WIFI_INTERFACE	WIFI_INTERFACE_SDIO   in project/stmicro_stm32f2xx_freertos_networking_8189em_8189fm_SDIO/inc/stm32_platform.h        

����Ŀ�꣺8189FTV sdio wifi driver
���빤�̣�Wlanlib_8189fm.uvproj
�������ã���#define CONFIG_SDIO_HCI �ر�CONFIG_GSPI_HCI	  in component/common/drivers/wlan/realtek/include/autoconf.h


����ṹ
----------------------
1.code��SDIO BUS Interface �����˷�װ���ӿ�ΪSDIO_BUS_OPS rtw_sdio_bus_ops���ṹ���£�

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

������wifi ͨ�ŵĽӿ��У�
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

�ֱ���linux kernel code��
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
��Ӧ

2.code�� sdio_func�Ľṹ�����޸ģ��ṹ����

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

linux kernel�� sdio_func �ṹ�壺
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

��linux kernel��ȣ����˾���ȥ��struct device dev�����void *drv_priv�����߹�����ͬ����SDIO Host��ȡwifi adapter ����

���ڲ�ͬ�ͻ�ʹ�õ�SDIO Host���ڲ��죬ʹ��8189FTV Wlan_lib for SDIOʱ��Ҫ�Ĳ����У�
1��component/common/drivers/sdio/wifi_io.h ��Ҫrelease���ͻ�
2��SDIO Host driver �������SDIO_BUS_OPS�ӿڴ���SDIO_BUS_OPS rtw_sdio_bus_ops ���󣬸ö������ʵ����wifi ͨ����صĽӿڣ�12����claim_irq��release_irq��claim_host....��writel����
2��SDIO Host driver �������struct sdio_func����struct sdio_func *wifi_sdio_func ����

