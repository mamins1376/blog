---
layout: post
title:  "تبدیل ایمیج به مخزن محلی"
date:   2015-05-10 16:37:40
categories: lmms music
---
ایمیج‌های دبیان(و طبیعتاً نوادگانش مثل اوبونتو) خیلی از بسته‌های پایه رو توی خودشون دارن که ساختاری دقیقا شبیه به مخزن‌های apt داره. DVDهای دبیان هم رایج‌ترین بسته‌ها(مثل میزکارها و ...) رو به همراه دارن. برای کسی که دانلود شبانه داره یکی از راه‌ها دانلود کردن بسته‌ها از قبل برای نصب در آینده‌ست. مسلما خیلی‌ها بعد از نصب کردن برنامه‌ی مورد نیازشون بسته‌های دانلود شده رو حذف می‌کنن و بدی این روش اینه‌که هر برنامه‌ای که لازم دارین باید تا دانلودش توی ساعت رایگان صبر کنین. خب چرا اغلب بسته‌ها رو دانلود نکنیم تا لازم نباشه هر دفعه صبر کنیم؟
<!-- ادامه -->

**توجه!** همه‌ی فرمان‌های زیر به دسترسی روت نیاز دارن. این ایمیج‌ها شامل بسته‌های `non-free` نمی‌شن.

#### ۱. دانلود ایمیج‌ها ####
من ۳ تا از DVDهای دبیان ۶۴بیتی رو دانلود کردم و توی پوشه‌ی خونگیم(یعنی `/root/`) ریختم:
{% highlight bash %}
root@debian:~# ls -sh1 /root/*.iso
3.8G debian-8.0.0-amd64-DVD-1.iso
4.4G debian-8.0.0-amd64-DVD-2.iso
4.4G debian-8.0.0-amd64-DVD-3.iso
{% endhighlight %}

#### ۲. سوارکردن ایمیج‌ها ####
من ۳ تا ایمیج دارم، پس ۳ تا پوشه برای هرکدوم توی مسیر `/mnt/` درست می‌کنم:
{% highlight bash %}
root@debian:~# mkdir -p /mnt/debian/dvd{1,2,3}
{% endhighlight %}
برای این که کار راحت‌تر بشه و نیازی به تایپ دستورهای طولانی نداشته باشیم، یه اسکریپت می‌سازیم که خودش برامون ایمیج‌ها رو سوار کنه. ایمیج‌های من توی `/root/` هستن. شما باید با توجه به مسیری که ایمیج‌هاتون اونجا هستن خط `ISO_DIR=/root` رو تغییر بدین:
{% highlight bash %}
root@debian:~# cat > /usr/local/sbin/mount-repo
#!/bin/sh
ISO_DIR=/root
mount -o loop,ro -t iso9660 $ISO_DIR/debian-8.0.0-amd64-DVD-1.iso /mnt/debian/dvd1/
mount -o loop,ro -t iso9660 $ISO_DIR/debian-8.0.0-amd64-DVD-2.iso /mnt/debian/dvd2/
mount -o loop,ro -t iso9660 $ISO_DIR/debian-8.0.0-amd64-DVD-3.iso /mnt/debian/dvd3/
^D
root@debian:~# chmod 744 /usr/local/sbin/mount-repo
root@debian:~# /usr/local/sbin/mount-repo
{% endhighlight %}
اگر اسکریپت `mount-repo` رو داخل `cron` قرار بدین دیگه نیازی نیست که هر دفعه خودتون اون رو فراخوانی کنین.

#### ۳. تنظیم apt ####
چون بسته‌هایی که داخل ایمیج‌ها هستن نیاز من رو برطرف می‌کردن، مخزن‌های راه دور رو کلا حذف کردم؛ اگر شما به مخازن فعلی‌تون نیاز دارین خطوط زیر رو به فایل مخازن apt فقط اضافه کنین. این فایل `etc/apt/sources.list/` منه:
{% highlight debsources %}
deb file:///mnt/debian/dvd1/ jessie contrib main
deb file:///mnt/debian/dvd2/ jessie contrib main
deb file:///mnt/debian/dvd3/ jessie contrib main
{% endhighlight %}
و در آخر:
{% highlight bash %}
root@debian:~# apt-get update
{% endhighlight %}
