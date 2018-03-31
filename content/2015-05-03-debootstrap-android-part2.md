+++
title = "دبیان بر بستر اندروید: پیش به سوی جسی!(قسمت دوم)"
date = 2015-05-03
tags = ["دبیان", "اندروید", "لینوکس", "debootstrap"]
category = "لینوکس"

+++

در پست قبل، بسته‌ها بارگیری و به پوشه‌ی `debroot` منتقل شدند. حالا ادامه‌ی مراحل را انجام می‌دهیم.
<!-- ادامه -->

#### ۳. ساخت تصویر توزیع ####
یک ایمیج خالی با استفاده از `dd` درست می‌کنیم:
```bash
root@host:~# dd if=/dev/zero of=Debian.img count=0 bs=1M seek=1K
```
حجم ایمیج ساخته‌شده ۱گیگابایت خواهد بود. برای داشتن اندازه‌ی دلخواهتان مقدار `seek` را بر حسب مگابایت تغییر دهید.  
آن را فرمت می‌کنیم:
```bash
root@host:~# mkfs.ext4 -F -L Debian Debian.img
```
و سپس سوار می‌کنیم:
```bash
root@host:~# mount -t ext4 -o loop,rw Debian.img /mnt
```
حالا همه‌ی بسته‌های بارگیری شده را به ایمیج کپی می‌کنیم:
```bash
root@host:~# cp -r debroot/* /mnt/; sync
```
و ایمیج را جدا:
```bash
root@host:~# umount /mnt
```

حالا ما یک ایمیج از توزیع‌مان داریم ولی هنوز آماده نیست. از این‌جا دیگر به پوشه‌ی `debroot` نیاز نداریم.

#### ۴. انتقال توزیع به اندروید ####
مهم نیست که ایمیج کجا کپی می‌شود، ولی باید قابل دسترس باشد و فضای کافی داشته باشد. مثلا ایمیج من در `/storage/sdcard1/Disks/` قرار دارد. پس از انتقال ایمیج، دیگر کاری با ماشین گنو/لینوکسی نداریم.  
**توجه!** در مرحله‌ی بعد به [BusyBox][busybox] نیاز خواهیم داشت. اگر هنوز آن را روی اندروید نصب نکرده‌اید، همین حالا [نصب کنید][busybox-install].

#### ۵. نصب توزیع ####
یک پوشه می‌سازیم. این پوشه محل سوارشدن ایمیج خواهد بود:
```bash
root@android:~# mkdir /data/debian
root@android:~# busybox mount -o loop,rw -t ext4 <IMAGE> /data/debian
```
یادتان نرود که در کدهای گفته شده، `<IMAGE>` را با مسیر ایمیجی که کپی کردیم عوض کنید. وارد توزیع می‌شویم:
```bash
root@android:~# export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
root@android:~# export USER=root
root@android:~# export HOME=/root
root@android:~# busybox chroot /data/debian /bin/bash
root@localhost:/# 
```
اکنون شما وارد توزیع شدید. اما هنوز بسته‌ها را استخراج نکردیم!
```bash
root@localhost:/# /debootstrap/debootstrap --second-stage
```
`debootstrap` شروع به نصب سیستم می‌کند.

**تبریک!** توزیع ما آماده است؛ ولی هنوز کارهای کوچکی باقی‌مانده که می‌تواند کار ما را راحت‌تر کند. مثلا با اسکریپت زیر، هربار توزیع آماده و اجرا می‌شود و پس از اتمام کار خودبه‌خود جدا می‌شود. من [این اسکریپت][script] را در مسیر `system/bin/debian/` قرار دادم.
```bash
export DEBMNT=/data/debian
busybox mount -t ext2 <IMAGE> $DEBMNT
busybox mount -t proc none $DEBMNT/proc
busybox mount -t sysfs none $DEBMNT/sys
busybox mount -o bind /dev $DEBMNT/dev
busybox mount -t devpts none $DEBMNT/dev/pts
unset LD_PRELOAD
LAST_PATH=$PATH
export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
export USER=root
export HOME=/root
export TERM=linux
busybox chroot $DEBMNT /bin/bash
export PATH=$LAST_PATH
sync
busybox umount $DEBMNT/dev/pts
busybox umount $DEBMNT/dev
busybox umount $DEBMNT/proc
busybox umount $DEBMNT/sys
busybox umount $DEBMNT || sh -c busybox umount -l $DEBMNT && echo 'Force unmounted!'
```
#### ۶. تنظیم apt ####
لازم است که تغییراتی در `sources.list` مربوط به مدیر بسته‌ی `apt` به‌وجود آوریم تا بتوانیم بسته‌های مورد نیازمان را نصب کنیم.
```bash
root@localhost:/# vi /etc/apt/sources.list
```
**توجه!** به جای ویرایش‌گر `vi`، هر ویرایشی که با آن راحت‌تر هستید استفاده کنید. `nano` به‌طور پیش‌فرض نصب است.  
خطوط زیر را به آن اضافه کنید یا فایل آماده را [دریافت کنید][sources.list].
```debsources
deb http://ftp.ir.debian.org/debian/ jessie contrib main non-free
deb-src http://ftp.ir.debian.org/debian/ jessie contrib main non-free

deb http://ftp.ir.debian.org/debian/ jessie-backports contrib main non-free
deb-src http://ftp.ir.debian.org/debian/ jessie-backports contrib main non-free

deb http://ftp.ir.debian.org/debian/ jessie-updates contrib main non-free
deb-src http://ftp.ir.debian.org/debian/ jessie-updates contrib main non-free
```
و در آخر،‌ برای به‌روزرسانی کَش `apt`:
```bash
root@localhost:/# apt-get update
```
**توجه!** فرمان بالا نیاز به اتصال اینترنت دارد.  
اکنون توزیع شما دقیقا مانند یک توزیع گنو/لینوکس عادی رفتار می‌کند.

در قسمت بعد به نصب محیط گرافیکی(به هر دو روش `framebuffer` و `VNC`) خواهیم پرداخت.

[busybox]: http://busybox.net/
[busybox-install]: https://play.google.com/store/apps/details?id=com.jrummy.busybox.installer
[script]: {{ site.baseurl }}/downloads/2015/05/debian
[sources.list]: {{ site.baseurl }}/downloads/2015/05/sources.list
