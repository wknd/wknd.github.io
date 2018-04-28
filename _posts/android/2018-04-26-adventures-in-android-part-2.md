---
layout: post
title:  "adventures in android flashing part 2"
date:   2018-04-26 08:00:00 +0200
last-modified: 2018-04-26 08:00:00 +0200
categories: blog android nethunter
featured: false
head-image: 
  name: ewaste
  global: global
  
---
Time for the real magic, tweaking the kernel for that extra functionality. I'll try doing it in stages with just small tweaks and test them out. First up are the [HID patches for keyboard and mouse](https://github.com/pelya/android-keyboard-gadget).

## Patching

I started off this patching effort by looking at the already existing patches. I was hoping by picking something by the same manufacturer and same kernel version I wouldn't have to do much.

Unfortunately that wasn't the case. I started by looking at the [msm8974 kernel_3.10 patch](https://github.com/pelya/android-keyboard-gadget/blob/master/patches/existing_tested/by-manufacturer-and-soc/motorola__msm8974______kernel_3.10.patch) and while manually going through it noticed it was creating new functions that already existed, instead of just overwriting them. That should have been a red flag, but I carried on simply fixing this silly code and only keeping the new one.  
When it came time to compile... it failed. 

```
drivers/usb/gadget/f_hid.o: In function `hidg_bind_config':
/home/weekend/Code/android/lineage/kernel/motorola/msm8952/drivers/usb/gadget/f_hid.c:804: multiple definition of `hidg_bind_config'
drivers/usb/gadget/android.o:/home/weekend/Code/android/lineage/kernel/motorola/msm8952/drivers/usb/gadget/f_hid.c:804: first defined here
drivers/usb/gadget/f_hid.o: In function `ghid_setup':
/home/weekend/Code/android/lineage/out/target/product/athene/obj/KERNEL_OBJ/../../../../../../kernel/motorola/msm8952/drivers/usb/gadget/f_hid.c:860: multiple definition of `ghid_setup'
drivers/usb/gadget/android.o:/home/weekend/Code/android/lineage/out/target/product/athene/obj/KERNEL_OBJ/../../../../../../kernel/motorola/msm8952/drivers/usb/gadget/f_hid.c:860: first defined here
drivers/usb/gadget/f_hid.o: In function `ghid_cleanup':
/home/weekend/Code/android/lineage/out/target/product/athene/obj/KERNEL_OBJ/../../../../../../kernel/motorola/msm8952/drivers/usb/gadget/f_hid.c:876: multiple definition of `ghid_cleanup'
drivers/usb/gadget/android.o:/home/weekend/Code/android/lineage/out/target/product/athene/obj/KERNEL_OBJ/../../../../../../kernel/motorola/msm8952/drivers/usb/gadget/f_hid.c:876: first defined here
```

I had the order wrong in some parts of the code, causing me to use functions before they were defined. After I fixed that it still failed, I had forgotten one of the duplicate functions. After fixing that I wrongly thought that maybe the previous error got cached and was messing everything up because I kept seeing the above message about multiple definitions even though there were none in the code.  
So naturally, not wanting to delve much deeper at the moment and with the wrong idea that the cache was messing things up I set out to resolve the wrong problem, this caused the most delays of all. Deleting already build stuff and clearing cache takes a **very** long time when its a project this size. And then I decided to undo my changes and sync back with the original code, again something that took a **very very** long time. In hindsight it was an obvious error, but at the time I just didn't see it.

In the end I looked closer at the already  [existing](https://github.com/LineageOS/android_kernel_motorola_msm8952/blob/cm-14.1/drivers/usb/gadget/android.c) [code](https://github.com/LineageOS/android_kernel_motorola_msm8952/blob/cm-14.1/drivers/usb/gadget/f_hid.c) again and at what the patch was doing, when I finally spotted it. Not only were some of these functions and some variables already defined in ```android.c``` but more importantly it was also already including a very relevant file.
{% highlight c %}
#include "f_hid.c"
{% endhighlight %}
While the patch called for creating a new ```f_hid.h``` file and:
{% highlight c %}
#include "f_hid.h"
#include "f_hid_android_keyboard.c"
#include "f_hid_android_mouse.c"
{% endhighlight %}

So I was defining function prototypes in ```f_hid.h```, and then defining them in the included ```f_hid.c``` AND also compiling f_hid.o (and thus defining it again just as the error tried to tell me)! I don't usually include ```.c``` files in my ```.c``` files, so I can understand why I didn't notice it right away. I even double checked to see if the ```f_hid.h``` had the proper ```#ifndef``` guards before I remembered I'm also creating the ```f_hid.o``` file. But I still think I should have caught it sooner.

To fix it I decided to not create a ```f_hid.h``` file(just like in the original code) and to not build a separate ```f_hid.o``` file. End result: a smaller patch is applied, without changing the ```MakeFile``` or creating a new ```f_hid.h``` file.

## Testing
After all that and waiting forever for it to build again, I got no errors. I packaged up the kernel in a zip using the [nethunter installer](https://github.com/offensive-security/kali-nethunter/blob/master/nethunter-installer/README.md) like before and tried it out. This time the [USB keyboard app](https://play.google.com/store/apps/details?id=remote.hid.keyboard.client) got a bit further, but it still wouldn't let me use it as a keyboard or mouse. It was stuck on:
```
opening /dev/hidg0 opening /dev/hidg1
```
At least my phone booted and there actually was a ```/dev/hidg0``` so we could overcome this problem too. It seemed like a permissions issue to me, and it was. I temporarily set selinux in permissive mode by using the adb shell with:
{% highlight shell %}
$ adb shell
athene:/ # setenforce 0
{% endhighlight %}
And now the app works, I could use it as a mouse and as a keyboard (though my machine not being qwerty and it of course just sending keycodes made it a bit annoying to type).

## Improvements
I need to keep in mind when I have created and tested all my desired patches, that I have to set proper selinux stuff. I don't want to manually set these things and I don't want to allow all things just to be able to use some of my extra functionality. In fact, I don't even want to be able to use ```setenforce 0``` via adb or locally at all. So I should tweak my build type too at some point (no more ```-userdebug``` or ```-eng``` builds when this is all over!).

All the patches will go in [this repo](https://github.com/wknd/android_kernel_motorola_msm8952-patches), and if I can I'll also submit them upstream to the respective projects. But just because I think the syntax highlighting of diff's are pretty, heres the patch:
{% highlight diff %}{% raw %}
diff --git a/drivers/usb/gadget/android.c b/drivers/usb/gadget/android.c
index 9bc546d..60d9892 100644
--- a/drivers/usb/gadget/android.c
+++ b/drivers/usb/gadget/android.c
@@ -62,6 +62,8 @@
 #include "f_ccid.c"
 #include "f_mtp.c"
 #include "f_accessory.c"
+#include "f_hid_android_keyboard.c"
+#include "f_hid_android_mouse.c"
 #include "f_rndis.c"
 #include "rndis.c"
 #include "f_qc_ecm.c"
@@ -1417,10 +1419,22 @@ static void hid_function_cleanup(struct android_usb_function *f)
 	ghid_cleanup();
 }
 
-static int hid_function_bind_config(struct android_usb_function *f,
-					struct usb_configuration *c)
+static int hid_function_bind_config(struct android_usb_function *f, struct usb_configuration *c)
 {
-	return hidg_bind_config(c, NULL, 0);
+	int ret;
+	printk(KERN_INFO "hid keyboard\n");
+	ret = hidg_bind_config(c, &ghid_device_android_keyboard, 0);
+	if (ret) {
+		pr_info("%s: hid_function_bind_config keyboard failed: %d\n", __func__, ret);
+		return ret;
+	}
+	printk(KERN_INFO "hid mouse\n");
+	ret = hidg_bind_config(c, &ghid_device_android_mouse, 1);
+	if (ret) {
+		pr_info("%s: hid_function_bind_config mouse failed: %d\n", __func__, ret);
+		return ret;
+	}
+	return 0;
 }
 
 static struct android_usb_function hid_function = {
@@ -1954,7 +1968,6 @@ static struct android_usb_function charger_function = {
 	.bind_config	= charger_function_bind_config,
 };
 
-
 static int
 mtp_function_init(struct android_usb_function *f,
 		struct usb_composite_dev *cdev)
@@ -2997,6 +3010,7 @@ static struct android_usb_function usbnet_function = {
 	.ctrlrequest	= usbnet_function_ctrlrequest,
 };
 
+
 static struct android_usb_function *supported_functions[] = {
 	&ffs_function,
 	&mbim_function,
@@ -3387,7 +3401,8 @@ functions_store(struct device *pdev, struct device_attribute *attr,
 							name, err);
 		}
 	}
-
+  /* HID driver always enabled, it's the whole point of this kernel patch */
+  android_enable_function(dev, conf, "hid");
 	/* Free uneeded configurations if exists */
 	while (curr_conf->next != &dev->configs) {
 		conf = list_entry(curr_conf->next,
diff --git a/drivers/usb/gadget/f_hid.c b/drivers/usb/gadget/f_hid.c
index f5ca673..5abd976 100644
--- a/drivers/usb/gadget/f_hid.c
+++ b/drivers/usb/gadget/f_hid.c
@@ -17,6 +17,7 @@
 #include <linux/poll.h>
 #include <linux/uaccess.h>
 #include <linux/wait.h>
+#include <linux/delay.h>
 #include <linux/sched.h>
 #include <linux/usb/g_hid.h>
 
@@ -64,6 +65,43 @@ struct f_hidg {
 	struct usb_ep			*out_ep;
 };
 
+/* Hacky device list to fix f_hidg_write being called after device destroyed.
+   It covers only most common race conditions, there will be rare crashes anyway. */
+enum { HACKY_DEVICE_LIST_SIZE = 4 };
+static struct f_hidg *hacky_device_list[HACKY_DEVICE_LIST_SIZE];
+static void hacky_device_list_add(struct f_hidg *hidg)
+{
+	int i;
+	for (i = 0; i < HACKY_DEVICE_LIST_SIZE; i++) {
+		if (!hacky_device_list[i]) {
+			hacky_device_list[i] = hidg;
+			return;
+		}
+	}
+	pr_err("%s: too many devices, not adding device %p\n", __func__, hidg);
+}
+static void hacky_device_list_remove(struct f_hidg *hidg)
+{
+	int i;
+	for (i = 0; i < HACKY_DEVICE_LIST_SIZE; i++) {
+		if (hacky_device_list[i] == hidg) {
+			hacky_device_list[i] = NULL;
+			return;
+		}
+	}
+	pr_err("%s: cannot find device %p\n", __func__, hidg);
+}
+static int hacky_device_list_check(struct f_hidg *hidg)
+{
+	int i;
+	for (i = 0; i < HACKY_DEVICE_LIST_SIZE; i++) {
+		if (hacky_device_list[i] == hidg) {
+			return 0;
+		}
+	}
+	return 1;
+}
+
 static inline struct f_hidg *func_to_hidg(struct usb_function *f)
 {
 	return container_of(f, struct f_hidg, func);
@@ -199,6 +237,11 @@ static ssize_t f_hidg_read(struct file *file, char __user *buffer,
 	if (!access_ok(VERIFY_WRITE, buffer, count))
 		return -EFAULT;
 
+  if (hacky_device_list_check(hidg)) {
+  	pr_err("%s: trying to read from device %p that was destroyed\n", __func__, hidg);
+  	return -EIO;
+  }
+
 	spin_lock_irqsave(&hidg->spinlock, flags);
 
 #define READ_COND (!list_empty(&hidg->completed_out_req))
@@ -269,6 +312,11 @@ static ssize_t f_hidg_write(struct file *file, const char __user *buffer,
 	if (!access_ok(VERIFY_READ, buffer, count))
 		return -EFAULT;
 
+	if (hacky_device_list_check(hidg)) {
+		pr_err("%s: trying to write to device %p that was destroyed\n", __func__, hidg);
+		return -EIO;
+	}
+  
 	mutex_lock(&hidg->lock);
 
 #define WRITE_COND (!hidg->write_pending)
@@ -283,6 +331,11 @@ static ssize_t f_hidg_write(struct file *file, const char __user *buffer,
 				hidg->write_queue, WRITE_COND))
 			return -ERESTARTSYS;
 
+    if (hacky_device_list_check(hidg)) {
+    	pr_err("%s: trying to write to device %p that was destroyed\n", __func__, hidg);
+    	return -EIO;
+    }
+
 		mutex_lock(&hidg->lock);
 	}
 
@@ -322,8 +375,17 @@ static unsigned int f_hidg_poll(struct file *file, poll_table *wait)
 {
 	struct f_hidg	*hidg  = file->private_data;
 	unsigned int	ret = 0;
+  if (hacky_device_list_check(hidg)) {
+  		pr_err("%s: trying to poll device %p that was destroyed\n", __func__, hidg);
+  		return -EIO;
+  }
 
 	poll_wait(file, &hidg->read_queue, wait);
+  if (hacky_device_list_check(hidg)) {
+  		pr_err("%s: trying to poll device %p that was destroyed\n", __func__, hidg);
+  		return -EIO;
+  }
+  
 	poll_wait(file, &hidg->write_queue, wait);
 
 	if (WRITE_COND)
@@ -422,7 +484,12 @@ static int hidg_setup(struct usb_function *f,
 	case ((USB_DIR_IN | USB_TYPE_CLASS | USB_RECIP_INTERFACE) << 8
 		  | HID_REQ_GET_PROTOCOL):
 		VDBG(cdev, "get_protocol\n");
-		goto stall;
+    length = min_t(unsigned, length, 1);
+		if (hidg->bInterfaceSubClass == USB_INTERFACE_SUBCLASS_BOOT)
+			((u8 *) req->buf)[0] = 0;	/* Boot protocol */
+		else
+			((u8 *) req->buf)[0] = 1;	/* Report protocol */
+		goto respond;
 		break;
 
 	case ((USB_DIR_OUT | USB_TYPE_CLASS | USB_RECIP_INTERFACE) << 8
@@ -434,6 +501,14 @@ static int hidg_setup(struct usb_function *f,
 	case ((USB_DIR_OUT | USB_TYPE_CLASS | USB_RECIP_INTERFACE) << 8
 		  | HID_REQ_SET_PROTOCOL):
 		VDBG(cdev, "set_protocol\n");
+		length = 0;
+		if (hidg->bInterfaceSubClass == USB_INTERFACE_SUBCLASS_BOOT) {
+			if (value == 0)		/* Boot protocol */
+				goto respond;
+		} else {
+			if (value == 1)		/* Report protocol */
+				goto respond;
+		}
 		goto stall;
 		break;
 
@@ -589,7 +664,8 @@ static int hidg_bind(struct usb_configuration *c, struct usb_function *f)
 	struct f_hidg		*hidg = func_to_hidg(f);
 	int			status;
 	dev_t			dev;
-
+  pr_info("%s: creating device %p\n", __func__, hidg);
+  
 	/* allocate instance-specific interface IDs, and patch descriptors */
 	status = usb_interface_id(c, f);
 	if (status < 0)
@@ -655,7 +731,7 @@ static int hidg_bind(struct usb_configuration *c, struct usb_function *f)
 		goto fail;
 
 	device_create(hidg_class, NULL, dev, NULL, "%s%d", "hidg", hidg->minor);
-
+  hacky_device_list_add(hidg);
 	return 0;
 
 fail:
@@ -674,12 +750,22 @@ static void hidg_unbind(struct usb_configuration *c, struct usb_function *f)
 {
 	struct f_hidg *hidg = func_to_hidg(f);
 
+  pr_info("%s: destroying device %p\n", __func__, hidg);
+  /* This does not cover all race conditions, only most common one */
+  mutex_lock(&hidg->lock);
+  hacky_device_list_remove(hidg);
+  mutex_unlock(&hidg->lock);
+  
 	device_destroy(hidg_class, MKDEV(major, hidg->minor));
 	cdev_del(&hidg->cdev);
 
 	/* disable/free request and end point */
 	usb_ep_disable(hidg->in_ep);
 	usb_ep_dequeue(hidg->in_ep, hidg->req);
+  /* TODO: calling this function crash kernel,
+     not calling this funct ion crash kernel inside f_hidg_write */
+  /* usb_ep_dequeue(hidg->in_ep, hidg->req); */
+  	
 	kfree(hidg->req->buf);
 	usb_ep_free_request(hidg->in_ep, hidg->req);
 
diff --git a/drivers/usb/gadget/f_hid_android_keyboard.c b/drivers/usb/gadget/f_hid_android_keyboard.c
new file mode 100644
index 0000000..1824bdd
--- /dev/null
+++ b/drivers/usb/gadget/f_hid_android_keyboard.c
@@ -0,0 +1,44 @@
+#include <linux/platform_device.h>
+#include <linux/usb/g_hid.h>
+
+/* hid descriptor for a keyboard */
+static struct hidg_func_descriptor ghid_device_android_keyboard = {
+	.subclass		= 1, /* Boot Interface Subclass */
+	.protocol		= 1, /* Keyboard */
+	.report_length		= 8,
+	.report_desc_length	= 63,
+	.report_desc		= {
+		0x05, 0x01,	/* USAGE_PAGE (Generic Desktop)	          */
+		0x09, 0x06,	/* USAGE (Keyboard)                       */
+		0xa1, 0x01,	/* COLLECTION (Application)               */
+		0x05, 0x07,	/*   USAGE_PAGE (Keyboard)                */
+		0x19, 0xe0,	/*   USAGE_MINIMUM (Keyboard LeftControl) */
+		0x29, 0xe7,	/*   USAGE_MAXIMUM (Keyboard Right GUI)   */
+		0x15, 0x00,	/*   LOGICAL_MINIMUM (0)                  */
+		0x25, 0x01,	/*   LOGICAL_MAXIMUM (1)                  */
+		0x75, 0x01,	/*   REPORT_SIZE (1)                      */
+		0x95, 0x08,	/*   REPORT_COUNT (8)                     */
+		0x81, 0x02,	/*   INPUT (Data,Var,Abs)                 */
+		0x95, 0x01,	/*   REPORT_COUNT (1)                     */
+		0x75, 0x08,	/*   REPORT_SIZE (8)                      */
+		0x81, 0x03,	/*   INPUT (Cnst,Var,Abs)                 */
+		0x95, 0x05,	/*   REPORT_COUNT (5)                     */
+		0x75, 0x01,	/*   REPORT_SIZE (1)                      */
+		0x05, 0x08,	/*   USAGE_PAGE (LEDs)                    */
+		0x19, 0x01,	/*   USAGE_MINIMUM (Num Lock)             */
+		0x29, 0x05,	/*   USAGE_MAXIMUM (Kana)                 */
+		0x91, 0x02,	/*   OUTPUT (Data,Var,Abs)                */
+		0x95, 0x01,	/*   REPORT_COUNT (1)                     */
+		0x75, 0x03,	/*   REPORT_SIZE (3)                      */
+		0x91, 0x03,	/*   OUTPUT (Cnst,Var,Abs)                */
+		0x95, 0x06,	/*   REPORT_COUNT (6)                     */
+		0x75, 0x08,	/*   REPORT_SIZE (8)                      */
+		0x15, 0x00,	/*   LOGICAL_MINIMUM (0)                  */
+		0x25, 0x65,	/*   LOGICAL_MAXIMUM (101)                */
+		0x05, 0x07,	/*   USAGE_PAGE (Keyboard)                */
+		0x19, 0x00,	/*   USAGE_MINIMUM (Reserved)             */
+		0x29, 0x65,	/*   USAGE_MAXIMUM (Keyboard Application) */
+		0x81, 0x00,	/*   INPUT (Data,Ary,Abs)                 */
+		0xc0		/* END_COLLECTION                         */
+	}
+};
diff --git a/drivers/usb/gadget/f_hid_android_mouse.c b/drivers/usb/gadget/f_hid_android_mouse.c
new file mode 100644
index 0000000..c0432b3
--- /dev/null
+++ b/drivers/usb/gadget/f_hid_android_mouse.c
@@ -0,0 +1,39 @@
+#include <linux/platform_device.h>
+#include <linux/usb/g_hid.h>
+
+/* HID descriptor for a mouse */
+static struct hidg_func_descriptor ghid_device_android_mouse = {
+	.subclass      = 1, /* Boot Interface Subclass */
+	.protocol      = 2, /* Mouse */
+	.report_length = 4,
+	.report_desc_length	= 52,
+	.report_desc = {
+		0x05, 0x01,  //Usage Page(Generic Desktop Controls)
+		0x09, 0x02,  //Usage (Mouse)
+		0xa1, 0x01,  //Collection (Application)
+		0x09, 0x01,  //Usage (pointer)
+		0xa1, 0x00,  //Collection (Physical)
+		0x05, 0x09,  //Usage Page (Button)
+		0x19, 0x01,  //Usage Minimum(1)
+		0x29, 0x05,  //Usage Maximum(5)
+		0x15, 0x00,  //Logical Minimum(1)
+		0x25, 0x01,  //Logical Maximum(1)
+		0x95, 0x05,  //Report Count(5)
+		0x75, 0x01,  //Report Size(1)
+		0x81, 0x02,  //Input(Data,Variable,Absolute,BitField)
+		0x95, 0x01,  //Report Count(1)
+		0x75, 0x03,  //Report Size(3)
+		0x81, 0x01,  //Input(Constant,Array,Absolute,BitField)
+		0x05, 0x01,  //Usage Page(Generic Desktop Controls)
+		0x09, 0x30,  //Usage(x)
+		0x09, 0x31,  //Usage(y)
+		0x09, 0x38,  //Usage(Wheel)
+		0x15, 0x81,  //Logical Minimum(-127)
+		0x25, 0x7F,  //Logical Maximum(127)
+		0x75, 0x08,  //Report Size(8)
+		0x95, 0x03,  //Report Count(3)
+		0x81, 0x06,  //Input(Data,Variable,Relative,BitField)
+		0xc0,  //End Collection
+		0xc0  //End Collection
+	}
+};

{% endraw %}{% endhighlight %}
<br />

-----

<br />
<br />
side note: having such large projects open in my editor and other places meant I hit a limit and jekyll wouldn't serve files locally instead giving this error:
```
FATAL: Listen error: unable to monitor directories for changes.
Visit https://github.com/guard/listen/wiki/Increasing-the-amount-of-inotify-watchers for info on how to fix this.
```
Sure I could increase the amount.. but maybe having "smart" editors keep track of entire kernel repository isn't the best idea.
