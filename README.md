# MediaTek-MTK-UART-enable


Introduction: Android Mediatek modifies UART device name and default debugging properties

This is the way to do it on Android. Later, when I worked on Linux system, I found that it can be done through a similar link method!



Linux method:

ln -sf /dev/ttyWCH0 /dev/ttyCH5

ln -sf /dev/ttyWCH1 /dev/ttyCH6

ln -sf /dev/ttyWCH2 /dev/ttyCH7

ln -sf /dev/ttyWCH3 /dev/ttyCH8



Android approach:

+++ b/system/core/rootdir/init.rc

@@ -245,8 +245,9 @@ on init

    symlink /proc/self/fd /dev/fd

 

    # set uart port

- symlink /dev/ttymxc2 /dev/ttyS1

+ symlink /dev/ttymxc2 /dev/ttyS2

    symlink /dev/ttymxc0 /dev/ttyS3

+ symlink /dev/ttymxc3 /dev/ttyS1





This modification mainly changes the name of the UART device in multiple files and modifies the default debugging properties.

Modified files:
build/make/core/main.mk
kernel-3.18/arch/arm64/configs/tb8735ap1_64_ztk_debug_defconfig
kernel-3.18/arch/arm64/configs/tb8735ap1_64_ztk_defconfig
kernel-3.18/drivers/misc/mediatek/uart/uart.c
Other related documents
File modification instructions:
1. main.mk
In this file, the default debug properties are modified. The original properties are ro.debuggable=1changed to ro.debuggable=0.

2. tb8735ap1_64_ztk_debug_defconfig and tb8735ap1_64_ztk_defconfig
In these two files, the UART device name in the kernel command line is modified. The original device name is ttyMT3, and it is changed to ttymxc3.

3. uart.c
In this file, the name of the UART device is changed. The original device name is ttyMT, change it to ttymxc.

4. Other relevant documents
Similar modifications have been made in other relevant documents.



 

 
diff --git a/build/make/core/main.mk b/build/make/core/main.mk
index 0c16c49f636..5518141cd84 100755
--- a/build/make/core/main.mk
+++ b/build/make/core/main.mk
@@ -276,7 +276,7 @@ ifeq (true,$(strip $(enable_target_debugging)))
   INCLUDE_TEST_OTA_KEYS := true
else # !enable_target_debugging
   # Target is less debuggable and adbd is off by default
- ADDITIONAL_DEFAULT_PROPERTIES += ro.debuggable=1
+ ADDITIONAL_DEFAULT_PROPERTIES += ro.debuggable=0
endif # !enable_target_debugging
 
##eng##
diff --git a/kernel-3.18/arch/arm64/configs/tb8735ap1_64_ztk_debug_defconfig b/kernel-3.18/arch/arm64/configs/tb8735ap1_64_ztk_debug_defconfig
index b334bd0755d..0873d48679b 100755
--- a/kernel-3.18/arch/arm64/configs/tb8735ap1_64_ztk_debug_defconfig
+++ b/kernel-3.18/arch/arm64/configs/tb8735ap1_64_ztk_debug_defconfig
@@ -37,7 +37,7 @@ CONFIG_ARMV8_DEPRECATED=y
CONFIG_SWP_EMULATION=y
CONFIG_CP15_BARRIER_EMULATION=y
CONFIG_SETEND_EMULATION=y
-CONFIG_CMDLINE="console=tty0 console=ttyMT3,921600n1 root=/dev/ram vmalloc=496M slub_max_order=0 slub_debug=O "
+CONFIG_CMDLINE="console=tty0 console=ttymxc3,921600n1 root=/dev/ram vmalloc=496M slub_max_order=0 slub_debug=O "
# CONFIG_EFI is not set
CONFIG_BUILD_ARM64_APPENDED_DTB_IMAGE=y
CONFIG_BUILD_ARM64_APPENDED_DTB_IMAGE_NAMES="mt6735"
diff --git a/kernel-3.18/arch/arm64/configs/tb8735ap1_64_ztk_defconfig b/kernel-3.18/arch/arm64/configs/tb8735ap1_64_ztk_defconfig
index 7b48da86d35..f828cdeb258 100755
--- a/kernel-3.18/arch/arm64/configs/tb8735ap1_64_ztk_defconfig
+++ b/kernel-3.18/arch/arm64/configs/tb8735ap1_64_ztk_defconfig
@@ -36,7 +36,7 @@ CONFIG_ARMV8_DEPRECATED=y
CONFIG_SWP_EMULATION=y
CONFIG_CP15_BARRIER_EMULATION=y
CONFIG_SETEND_EMULATION=y
-CONFIG_CMDLINE="console=tty0 console=ttyMT3,921600n1 root=/dev/ram vmalloc=496M slub_max_order=0 slub_debug=O "
+CONFIG_CMDLINE="console=tty0 console=ttymxc3,921600n1 root=/dev/ram vmalloc=496M slub_max_order=0 slub_debug=O "
# CONFIG_EFI is not set
CONFIG_BUILD_ARM64_APPENDED_DTB_IMAGE=y
CONFIG_BUILD_ARM64_APPENDED_DTB_IMAGE_NAMES="mt6735"
diff --git a/kernel-3.18/drivers/misc/mediatek/uart/uart.cb/kernel-3.18/drivers/misc/mediatek/uart/uart.c
old mode 100644
new mode 100755
index 45616ce9137..8b09254eb77
--- a/kernel-3.18/drivers/misc/mediatek/uart/uart.c
+++ b/kernel-3.18/drivers/misc/mediatek/uart/uart.c
@@ -765,12 +765,12 @@ static int find_fingerprint(char str[], int *offset)
   */
  static const char * const fingerprint[] = {
    "console=/dev/null",
- "console=ttyMT0",
- "console=ttyMT1",
- "console=ttyMT2",
- "console=ttyMT3",
+ "console=ttymxc0",
+ "console=ttymxc1",
+ "console=ttymxc2",
+ "console=ttymxc3",
#if (UART_NR > 4)
- "console=ttyMT4",
+ "console=ttymxc4",
#endif
  };
  int i;
@@ -883,7 +883,7 @@ unsigned int mtk_uart_pdn_enable(char *port, int enable)
    return -1;
  }
 
- if (find_string(port, "ttyMT", 0) == 0) {
+ if (find_string(port, "ttymxc", 0) == 0) {
    MSG_ERR("Format mismatch! str=%s\n", port);
    return -1;
  }
@@ -915,7 +915,7 @@ unsigned int mtk_uart_freeze_enable(char *port, int enable)
    return -1;
  }
 
- if (find_string(port, "ttyMT", 0) == 0) {
+ if (find_string(port, "ttymxc", 0) == 0) {
    MSG_ERR("Format mismatch! str=%s\n", port);
    return -1;
  }
@@ -1025,7 +1025,7 @@ static int __init mtk_uart_console_setup(struct console *co, char *options)
/*------------------------------------------------ --------------------------*/
static struct uart_driver mtk_uart_drv;
static struct console mtk_uart_console = {
- .name = "ttyMT",
+ .name = "ttymxc",
#if !defined(CONFIG_MTK_SERIAL_MODEM_TEST)
  /*don't configure UART4 as console */
  .write = mtk_uart_console_write,
@@ -2298,7 +2298,7 @@ static struct uart_ops mtk_uart_ops = {
static struct uart_driver mtk_uart_drv = {
  .owner = THIS_MODULE,
  .driver_name = DRV_NAME,
- .dev_name = "ttyMT",
+ .dev_name = "ttymxc",
  .major = UART_MAJOR,
  .minor = UART_MINOR,
  .nr = UART_NR,
@@ -2681,7 +2681,11 @@ static int mtk_uart_init_ports(void)
    uart->port.fifosize = UART_FIFO_SIZE;
    uart->port.ops = &mtk_uart_ops;
    uart->port.flags = UPF_BOOT_AUTOCONF;
- uart->port.line = i;
+ //uart->port.line = i;
+ if(i == 0) uart->port.line = 1;
+ else if (i == 1) uart->port.line = 0;
+ else uart->port.line = i;
+
    uart->port.uartclk = UART_SYSCLK;
    /* pll_for_uart = mt6575_get_bus_freq(); */
    /* uart->port.uartclk = mt6575_get_bus_freq()*1000/4; */
diff --git a/kernel-3.18/kernel/printk/printk.cb/kernel-3.18/kernel/printk/printk.c
index f73e332925d..6702c656142 100755
--- a/kernel-3.18/kernel/printk/printk.c
+++ b/kernel-3.18/kernel/printk/printk.c
@@ -1704,7 +1704,7 @@ static void call_console_drivers(int level, const char *text, size_t len)
    tmp2 = local_clock();
 
    differ = tmp2 - tmp1;
- if (!strcmp(con->name, "ttyMT")) {
+ if (!strcmp(con->name, "ttymxc")) {
      len_1 += len;
      accum_t1 += differ;
    } else if (!strcmp(con->name, "pstore")) {
diff --git a/system/core/rootdir/ueventd.rc b/system/core/rootdir/ueventd.rc
index 231c9c11728..6cc8549c5d1 100755
--- a/system/core/rootdir/ueventd.rc
+++ b/system/core/rootdir/ueventd.rc
@@ -134,4 +134,8 @@ subsystem sound
 
# DVB API device nodes
/dev/dvb* 0660 root system
-/dev/sglctl 0777 root system
\ No newline at end of file
+/dev/sglctl 0777 root system
+/dev/ttymxc0 0666 root system
+/dev/ttymxc1 0666 root system
+/dev/ttymxc2 0666 root system
+/dev/ttymxc3 0666 root system
\ No newline at end of file
diff --git a/vendor/mediatek/kernel_modules/connectivity/common/common_detect/mtk_wcn_stub_alps.cb/vendor/mediatek/kernel_modules/connectivity/common/common_detect/mtk_wcn_stub_alps.c
old mode 100644
new mode 100755
index b3e6b7c4528..5984d5949a4
--- a/vendor/mediatek/kernel_modules/connectivity/common/common_detect/mtk_wcn_stub_alps.c
+++ b/vendor/mediatek/kernel_modules/connectivity/common/common_detect/mtk_wcn_stub_alps.c
@@ -85,7 +85,7 @@ int gConnectivityChipId = -1;
* current used uart port name, default is "ttyMT2",
* will be changed when wmt driver init
*/
-char *wmt_uart_port_desc = "ttyMT2";
+char *wmt_uart_port_desc = "ttymxc2";
EXPORT_SYMBOL(wmt_uart_port_desc);
 
#ifdef MTK_WCN_REMOVE_KERNEL_MODULE
diff --git a/vendor/mediatek/proprietary/bootable/bootloader/lk/app/mt_boot/mt_boot.cb/vendor/mediatek/proprietary/bootable/bootloader/lk/app/mt_boot/mt_boot.c
index 55e5b07f6f2..59ee166483b 100755
--- a/vendor/mediatek/proprietary/bootable/bootloader/lk/app/mt_boot/mt_boot.c
+++ b/vendor/mediatek/proprietary/bootable/bootloader/lk/app/mt_boot/mt_boot.c
@@ -1538,9 +1538,9 @@ int boot_linux_fdt(void *kernel, unsigned *tags,
    case BUILD_TYPE_USER:
      if ((g_boot_mode == META_BOOT) && is_meta_log_disable &&
          (is_meta_log_disable() == 0))
- cmdline_append("printk.disable_uart=0");
+ cmdline_append("printk.disable_uart=1");
      else
- cmdline_append("printk.disable_uart=0");
+ cmdline_append("printk.disable_uart=1");
      break;
 
    case BUILD_TYPE_USERDEBUG:
@@ -1887,7 +1887,7 @@ void boot_linux(void *kernel, unsigned *tags,
  if (!has_set_p2u) {
    switch (eBuildType) {
    case BUILD_TYPE_USER:
- cmdline_append("printk.disable_uart=0");
+ cmdline_append("printk.disable_uart=1");
      break;
 
    case BUILD_TYPE_USERDEBUG:
diff --git a/vendor/mediatek/proprietary/bootable/bootloader/lk/platform/mt6735/include/platform/mt_reg_base.hb/vendor/mediatek/proprietary/bootable/bootloader/lk/platform/mt6735/include/platform/ mt_reg_base.h
old mode 100644
new mode 100755
index 907aef6275d..eb926aad8a0
--- a/vendor/mediatek/proprietary/bootable/bootloader/lk/platform/mt6735/include/platform/mt_reg_base.h
+++ b/vendor/mediatek/proprietary/bootable/bootloader/lk/platform/mt6735/include/platform/mt_reg_base.h
@@ -679,9 +679,9 @@
#endif
 
#ifdef MACH_FPGA
-#define COMMANDLINE_TO_KERNEL "console=tty0 console=ttyMT0,921600n1 " CMDLINE_ROOT " androidboot.hardware=mt6735 firmware_class.path=/vendor/firmware"
+#define COMMANDLINE_TO_KERNEL "console=tty0 console=ttymxc0,921600n1 " CMDLINE_ROOT " androidboot.hardware=mt6735 firmware_class.path=/vendor/firmware"
#else
-#define COMMANDLINE_TO_KERNEL "console=tty0 console=ttyMT3,921600n1 " CMDLINE_ROOT " vmalloc=496M androidboot.hardware=mt6735 slub_max_order=0 slub_debug=OFZPU firmware_class.path=/vendor/firmware"
+#define COMMANDLINE_TO_KERNEL "console=tty0 console=ttymxc3,921600n1 " CMDLINE_ROOT " vmalloc=496M androidboot.hardware=mt6735 slub_max_order=0 slub_debug=OFZPU firmware_class.path=/vendor/firmware"
#endif
 
#define CFG_FACTORY_NAME "factory.img"
diff --git a/vendor/mediatek/proprietary/bootable/bootloader/lk/platform/mt6735/uart.cb/vendor/mediatek/proprietary/bootable/bootloader/lk/platform/mt6735/uart.c
old mode 100644
new mode 100755
index d3fa9eb0572..011485c8a15
--- a/vendor/mediatek/proprietary/bootable/bootloader/lk/platform/mt6735/uart.c
+++ b/vendor/mediatek/proprietary/bootable/bootloader/lk/platform/mt6735/uart.c
@@ -163,16 +163,16 @@ static void change_uart_port(char * cmd_line, char new_val)
  len = strlen(cmd_line);
  ptr = cmd_line;
 
- i = strlen("ttyMT");
+ i = strlen("ttymxc");
  if(len < i)
    return;
  len = len-i;
 
  for(i=0; i<=len; i++)
  {
- if(strncmp(ptr, "ttyMT", 5)==0)
+ if(strncmp(ptr, "ttymxc", 6)==0)
    {
- ptr[5] = new_val; // Find and modify
+ ptr[6] = new_val; // Find and modify
      break;
    }
    ptr++;
diff --git a/vendor/mediatek/proprietary/bootable/bootloader/preloader/custom/tb8735ap1_64_ztk/cust_bldr.mak b/vendor/mediatek/proprietary/bootable/bootloader/preloader/custom/tb8735ap1_64_ztk/cust_bldr.mak
old mode 100644
new mode 100755
index fef16366f3d..002d02bf625
--- a/vendor/mediatek/proprietary/bootable/bootloader/preloader/custom/tb8735ap1_64_ztk/cust_bldr.mak
+++ b/vendor/mediatek/proprietary/bootable/bootloader/preloader/custom/tb8735ap1_64_ztk/cust_bldr.mak
@@ -7,7 +7,7 @@ CFG_BOOT_DEV :=BOOTDEV_SDMMC
else
CFG_BOOT_DEV :=BOOTDEV_NAND
endif
-CFG_UART_LOG :=UART1
+CFG_UART_LOG :=UART2
CFG_UART_META := UART1
CFG_TEE_SUPPORT = 0
CFG_TRUSTONIC_TEE_SUPPORT = 0
diff --git a/vendor/mediatek/proprietary/bootable/bootloader/preloader/platform/mt6735/default.mak b/vendor/mediatek/proprietary/bootable/bootloader/preloader/platform/mt6735/default.mak
old mode 100644
new mode 100755
index 438fa9a4d9d..2e77d7fcb3b
--- a/vendor/mediatek/proprietary/bootable/bootloader/preloader/platform/mt6735/default.mak
+++ b/vendor/mediatek/proprietary/bootable/bootloader/preloader/platform/mt6735/default.mak
@@ -31,7 +31,7 @@ CFG_USB_TOOL_HANDSHAKE :=1
CFG_USB_DOWNLOAD :=1
CFG_LOG_BAUDRATE :=921600
CFG_META_BAUDRATE := 115200
-CFG_UART_LOG :=UART1
+CFG_UART_LOG :=UART2
CFG_UART_META := UART1
 
#only enable in eng mode
diff --git a/vendor/mediatek/proprietary/factory/inc/utils.hb/vendor/mediatek/proprietary/factory/inc/utils.h
index 2fbf462752b..386ecfff6bf 100755
--- a/vendor/mediatek/proprietary/factory/inc/utils.h
+++ b/vendor/mediatek/proprietary/factory/inc/utils.h
@@ -48,7 +48,7 @@ extern "C" {
 
#define MODEM_MAX_NUM 2
 
-#define CCCI_MODEM_MT6252 "/dev/ttyMT0"
+#define CCCI_MODEM_MT6252 "/dev/ttymxc0"
#define CCCI_MODEM_MT8135 "/dev/ttyUSB1"
 
/*signaltest*/
diff --git a/vendor/mediatek/proprietary/factory/src/test/ftm_sim.cb/vendor/mediatek/proprietary/factory/src/test/ftm_sim.c
old mode 100644
new mode 100755
index a2d17931c12..46c833a370d
--- a/vendor/mediatek/proprietary/factory/src/test/ftm_sim.c
+++ b/vendor/mediatek/proprietary/factory/src/test/ftm_sim.c
@@ -167,7 +167,7 @@ char dev_node_data_2[32] = {0};
#define EFIFO_IOC_RESET_BUFFER _IO(CCCI_IOC_MAGIC, 203)
 
#define DEVICE_NAME_3 "/dev/ttyUSB1"
-#define DEVICE_NAME_EXTRA "/dev/ttyMT0"
+#define DEVICE_NAME_EXTRA "/dev/ttymxc0"
int fd_at = -1;
int fd_atdt = -1;
#define SIM_SWITCH_MODE_CDMA 0x010001
diff --git a/vendor/mediatek/proprietary/factory/src/util/uart_op.cpp b/vendor/mediatek/proprietary/factory/src/util/uart_op.cpp
old mode 100644
new mode 100755
index a9377a4d117..9fd19ca84ff
--- a/vendor/mediatek/proprietary/factory/src/util/uart_op.cpp
+++ b/vendor/mediatek/proprietary/factory/src/util/uart_op.cpp
@@ -251,7 +251,7 @@ int open_uart_port(int uart_id, int baudrate, int length, char parity_c, int sto
    return fd;
 
  //sprintf(dev, "/dev/ttyMT%d", (int)(uart_id-1) );
- snprintf(dev,sizeof(dev),"%s%d","/dev/ttyMT", (int)(uart_id-1));
+ snprintf(dev,sizeof(dev),"%s%d","/dev/ttymxc", (int)(uart_id-1));
  /* Open device now */
  fd = open(dev, O_RDWR|O_NOCTTY|O_NONBLOCK);
 
diff --git a/vendor/mediatek/proprietary/hardware/connectivity/bluetooth/driver/mt66xx/pure/combo/bt_relayer.cb/vendor/mediatek/proprietary/hardware/connectivity/bluetooth/driver/mt66xx/pure/combo/ bt_relayer.c
old mode 100644
new mode 100755
index 7968b683b1b..b73105175f5
--- a/vendor/mediatek/proprietary/hardware/connectivity/bluetooth/driver/mt66xx/pure/combo/bt_relayer.c
+++ b/vendor/mediatek/proprietary/hardware/connectivity/bluetooth/driver/mt66xx/pure/combo/bt_relayer.c
@@ -117,7 +117,7 @@ static int init_serial(int port, int speed)
     char usb_prop[PROPERTY_VALUE_MAX];
 
     if (port < 4) { /* serial port UART */
- sprintf(dev, "/dev/ttyMT%d", port);
+ sprintf(dev, "/dev/ttymxc%d", port);
     }
     else { /* serial port USB */
         sprintf(dev, "/dev/ttyGS2");
diff --git a/vendor/mediatek/proprietary/hardware/connectivity/combo_tool/src/stp_uart_launcher.cb/vendor/mediatek/proprietary/hardware/connectivity/combo_tool/src/stp_uart_launcher.c
old mode 100644
new mode 100755
index 279d751da66..461f8c591fa
--- a/vendor/mediatek/proprietary/hardware/connectivity/combo_tool/src/stp_uart_launcher.c
+++ b/vendor/mediatek/proprietary/hardware/connectivity/combo_tool/src/stp_uart_launcher.c
@@ -59,7 +59,7 @@
#endif
#define HCIUARTSETPROTO _IOW('U', 200, int)
#define CUST_COMBO_WMT_DEV "/dev/stpwmt"
-#define CUST_COMBO_STP_DEV "/dev/ttyMT2"
+#define CUST_COMBO_STP_DEV "/dev/ttymxc2"
#define CUST_COMBO_PATCH_PATH "/vendor/firmware"
#define CUST_COMBO_CFG_FILE "/system/vendor/firmware/WMT.cfg"
 
@@ -1125,7 +1125,7 @@ static void launcher_set_uart_port_name(char *stp_dev) {
     }
 
     if (!uart_name) {
- uart_name = "ttyMT2";
+ uart_name = "ttymxc2";
         ALOGI("use default uart %s\n", uart_name);
     }
 
diff --git a/vendor/mediatek/proprietary/hardware/connectivity/gps/gps.rc b/vendor/mediatek/proprietary/hardware/connectivity/gps/gps.rc
old mode 100644
new mode 100755
index c2850d8cca3..2f9f1ff5785
--- a/vendor/mediatek/proprietary/hardware/connectivity/gps/gps.rc
+++ b/vendor/mediatek/proprietary/hardware/connectivity/gps/gps.rc
@@ -9,8 +9,8 @@
     chown gps gps /sys/class/gpsdrv/gps/status
 
#/dev/ttyMT1 for GPS 3337 usage
-chmod 0660 /dev/ttyMT1
- chown system system /dev/ttyMT1
+ chmod 0660 /dev/ttymxc1
+ chown system system /dev/ttymxc1
 
# GPS EMI
     chmod 666 /dev/gps_emi
diff --git a/vendor/mediatek/proprietary/hardware/connectivity/gps/gps_hal/src/gpsinf3337.cb/vendor/mediatek/proprietary/hardware/connectivity/gps/gps_hal/src/gpsinf3337.c
old mode 100644
new mode 100755
index 6d38833e916..e0365fc7edc
--- a/vendor/mediatek/proprietary/hardware/connectivity/gps/gps_hal/src/gpsinf3337.c
+++ b/vendor/mediatek/proprietary/hardware/connectivity/gps/gps_hal/src/gpsinf3337.c
@@ -77,7 +77,7 @@
 
/* the name of the controlled socket */
#define GPS_POWER_NAME "/dev/gps"
-#define GPS_TTY_NAME "/dev/ttyMT1"
+#define GPS_TTY_NAME "/dev/ttymxc1"
 
#define MNLD_HAL2GPS "/dev/mt3337_gpsonly"
#define MNL_CONFIG_STATUS "persist.radio.mnl.prop"
diff --git a/vendor/mediatek/proprietary/hardware/meta/common/inc/MetaPub.hb/vendor/mediatek/proprietary/hardware/meta/common/inc/MetaPub.h
old mode 100644
new mode 100755
index fd29f016d6c..7d7fa8fe0a4
--- a/vendor/mediatek/proprietary/hardware/meta/common/inc/MetaPub.h
+++ b/vendor/mediatek/proprietary/hardware/meta/common/inc/MetaPub.h
@@ -131,10 +131,10 @@ Mux Header Format
#define MBLOG_PULL_STATUS "debug.MB.packed"
 
 
-#define UART1_PATH "/dev/ttyMT0"
-#define UART2_PATH "/dev/ttyMT1"
-#define UART3_PATH "/dev/ttyMT2"
-#define UART4_PATH "/dev/ttyMT3"
+#define UART1_PATH "/dev/ttymxc0"
+#define UART2_PATH "/dev/ttymxc1"
+#define UART3_PATH "/dev/ttymxc2"
+#define UART4_PATH "/dev/ttymxc3"
 
 
 
--
2.29.0
