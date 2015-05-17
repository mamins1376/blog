---
layout: post
title:  "کانفیگ tor و privoxy مثل آب خوردن"
date:   2015-05-17 17:35:13
categories: openvpn network manager
---
به مناسبت [روز جهانی جامعه‌ی اطلاعاتی][WISDay]، از تور و لطف بزرگی که در حق ما می‌کنه و کانفیگش به همراه دوست عزیزش، privoxy می‌نویسم. چند وقتیه تور خوب وصل نمی‌شه (البته بیچاره تقصیر خودش نیست) ولی خب هر مشکلی راه حلی داره! privoxy هم کمکمون می‌کنه تا علاوه بر فیلتر تبلیغات، یه پروکسی http خوب از تور درست کنیم.
<!-- ادامه -->

#### ۱. نصب تور، پریواکسی و obfsproxy ####
هر سه‌تا توی مخازن رسمی موجودن:
{% highlight bash %}
root@debian:~# apt-get install tor obfsproxy privoxy
{% endhighlight %}

#### ۲. کانفیگ تور ####
قبل از این که فایل کانفیگ رو دستکاری کنیم، به تعدادی پل احتیاج داریم؛ obfsproxy موجود در مخزن رسمی فعلا فقط از obfs2 پشتیبانی می‌کنه وگرنه obfs4 توصیه می‌شه. روش خوب برای گرفتن پل‌ها [فرستادن یک ایمیل به تور][torbridges] بدون عنوان با محتوای زیره:
{% highlight text %}
get transport obfs2
{% endhighlight %}
**توجه!** فعلا فقط ایمیل از سرویس‌های یاهو، جیمیل و Riseup قابل قبول هستن.  
در پاسخ یک ایمیل حاوی آدرس پل‌ها دریافت می‌کنین. در دستور زیر، به جای عبارت <TRANSPORT> یکی از پل‌هایی رو که دریافت کردین رو قرار بدین.
{% highlight bash %}
root@debian:~# cat << "EOF" >> /etc/tor/torrc
UseBridges 1
# e.g. Bridge obfs2 123.45.67.89:1011 UI373R...9238F98
Bridge <TRANSPORT>
ClientTransportPlugin obfs2 exec /usr/bin/obfsproxy --managed
EOF
root@debian:~# service tor restart
{% endhighlight %}

#### ۳. کانفیگ پریواکسی ####
{% highlight bash %}
root@debian:~# cat << "EOF" >> /etc/privoxy/config
forward-socks5   /    127.0.0.1:9050 .
forward              192.168.*.*/  .
forward   10.*.*.*/  .
forward   127.*.*.*/  .
forward   localhost/  .
EOF
root@debian:~# service tor restart
{% endhighlight %}

#### ۴. سلام کردن به اینترنت ####
لاگ خود تور رو می‌شه با دستور زیر مانیتور کرد:
{% highlight bash %}
root@debian:~# tail -f /var/log/tor/log
{% endhighlight %}
پریواکسی روی پورت 8118 با پروتکل‌های http، https و ftp آماده‌ی هدایت شما به اینترنته!

[WISDay]: https://en.wikipedia.org/wiki/World_Information_Society_Day
[torbridges]: mailto:bridges@torproject.org
