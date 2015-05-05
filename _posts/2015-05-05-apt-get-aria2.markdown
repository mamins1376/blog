---
layout: post
title:  "آریا۲ و اپت: سرعتی دوچندان"
date:   2015-05-05 14:04:00
categories: linux cron rtcwake
---
یکی از مشکلات کاربران دبیان-بیس‌ها، سرعت apt در امر طاقت‌فرسای دانلوده که باعث می‌شه خیلی‌ها سراغ مواردی مثل apt-fast برن. ما با ترکیب آریا۲ و اپت، یه دانلود کننده‌ی توپ می‌سازیم.
<!-- ادامه -->

#### ۱. استخراج لینک‌ها ####
مسلماً بسته‌های دبیان چیزی جز فایل نیستن و apt هم این «فایل»ها رو دانلود می‌کنه. خب وقتی خودش می‌تونه دانلود کنه،‌ چرا ما نتونیم؟ خوش‌بختانه برای همچین‌کاری یه سوئیچ خوب وجود داره:
{% highlight bash %}
root@debian:~# apt-get -y --print-uris { install <PACKAGE> | upgrade | ... }
{% endhighlight %}
این سوئیچ به apt می‌گه که فقط لینک رو بنویس و بی‌خیال دانلود شو. سوئیچ `y` هم می‌گه که خودت تائید کن.
مثلا من guake رو امتحان می‌کنم:

{% highlight bash %}
root@debian:~# apt-get -y --print-uris install guake 
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following extra packages will be installed:
  python-glade2 python-vte
Suggested packages:
  python-gtk2-doc
The following NEW packages will be installed:
  guake python-glade2 python-vte
0 upgraded, 3 newly installed, 0 to remove and 0 not upgraded.
Need to get 201 kB of archives.
After this operation, 1,381 kB of additional disk space will be used.
'http://ir.archive.ubuntu.com/ubuntu/pool/main/v/vte/python-vte_0.28.2-5ubuntu1_amd64.deb' python-vte_1%3a0.28.2-5ubuntu1_amd64.deb 21726 MD5Sum:a0b9e19f09fa001ca3e75534a5298c51
'http://ir.archive.ubuntu.com/ubuntu/pool/main/p/pygtk/python-glade2_2.24.0-3ubuntu3_amd64.deb' python-glade2_2.24.0-3ubuntu3_amd64.deb 8744 MD5Sum:1aef0e4a843eac23ffc619bc02382f03
'http://ir.archive.ubuntu.com/ubuntu/pool/universe/g/guake/guake_0.4.4-1ubuntu1_amd64.deb' guake_0.4.4-1ubuntu1_amd64.deb 171024 MD5Sum:a24f28469f7a83debc4aed2ec6ea1515
{% endhighlight %}
اطلاعاتی که ما لازم داریم سه خط آخر هستن که وجه اشتراکشون `//:http` ابتداشونه. پس با `grep` فیلترشون می‌کنیم:
{% highlight bash %}
root@debian:~# apt-get --print-uris -y install guake | grep 'http://'
'http://ir.archive.ubuntu.com/ubuntu/pool/main/v/vte/python-vte_0.28.2-5ubuntu1_amd64.deb' python-vte_1%3a0.28.2-5ubuntu1_amd64.deb 21726 MD5Sum:a0b9e19f09fa001ca3e75534a5298c51
'http://ir.archive.ubuntu.com/ubuntu/pool/main/p/pygtk/python-glade2_2.24.0-3ubuntu3_amd64.deb' python-glade2_2.24.0-3ubuntu3_amd64.deb 8744 MD5Sum:1aef0e4a843eac23ffc619bc02382f03
'http://ir.archive.ubuntu.com/ubuntu/pool/universe/g/guake/guake_0.4.4-1ubuntu1_amd64.deb' guake_0.4.4-1ubuntu1_amd64.deb 171024 MD5Sum:a24f28469f7a83debc4aed2ec6ea1515
{% endhighlight %}
ولی ما فقط به اون قسمت لینک نیاز داریم پس اون‌رو هم با `cut` فیلتر می‌کنیم:
{% highlight bash %}'''
root@debian:~# apt-get --print-uris -y install guake | grep 'http://' | cut -d\' -f2
http://ir.archive.ubuntu.com/ubuntu/pool/main/v/vte/python-vte_0.28.2-5ubuntu1_amd64.deb
http://ir.archive.ubuntu.com/ubuntu/pool/main/p/pygtk/python-glade2_2.24.0-3ubuntu3_amd64.deb
http://ir.archive.ubuntu.com/ubuntu/pool/universe/g/guake/guake_0.4.4-1ubuntu1_amd64.deb
'''{% endhighlight %}
حالا لینک ها رو داریم. می‌ریزیمشون توی یه فایل به اسم `Links.txt`:
{% highlight bash %}
root@debian:~# apt-get --print-uris -y install guake | grep 'http://' | cut -d\' -f2 > Links.txt
{% endhighlight %}

#### ۲. دانلود بسته‌ها ####
برای این کار از `aria2` استفاده می‌کنیم.
{% highlight bash %}
root@debian:~# aria2c -d /var/cache/apt/archives/ -i Links.txt -s 16 -x 16 -c
{% endhighlight %}

#### ۳. نصب بسته ها ####
همیشه اپت بسته‌ها رو توی مسیر `/var/cache/apt/archives/` می‌ریزه. خب، ما که خودمون این کار رو با aria2 کردیم! پس دیگه apt نیازی به دانلود نداره.
{% highlight bash %}
root@debian:~# apt-get -y install guake
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following extra packages will be installed:
  python-glade2 python-vte
Suggested packages:
  python-gtk2-doc
The following NEW packages will be installed:
  guake python-glade2 python-vte
0 upgraded, 3 newly installed, 0 to remove and 0 not upgraded.
Need to get 0 B/201 kB of archives.
After this operation, 1,381 kB of additional disk space will be used.
Selecting previously unselected package python-vte.
(Reading database ... 240702 files and directories currently installed.)
Preparing to unpack .../python-vte_0.28.2-5ubuntu1_amd64.deb ...
Unpacking python-vte (1:0.28.2-5ubuntu1) ...
Selecting previously unselected package python-glade2.
Preparing to unpack .../python-glade2_2.24.0-3ubuntu3_amd64.deb ...
Unpacking python-glade2 (2.24.0-3ubuntu3) ...
Selecting previously unselected package guake.
Preparing to unpack .../guake_0.4.4-1ubuntu1_amd64.deb ...
Unpacking guake (0.4.4-1ubuntu1) ...
Processing triggers for gconf2 (3.2.6-0ubuntu2) ...
Processing triggers for man-db (2.6.7.1-1ubuntu1) ...
Processing triggers for gnome-menus (3.10.1-0ubuntu2) ...
Processing triggers for desktop-file-utils (0.22-1ubuntu1) ...
Processing triggers for bamfdaemon (0.5.1+14.04.20140409-0ubuntu1) ...
Rebuilding /usr/share/applications/bamf-2.index...
Processing triggers for mime-support (3.54ubuntu1.1) ...
Processing triggers for hicolor-icon-theme (0.13-1) ...
Setting up python-vte (1:0.28.2-5ubuntu1) ...
Setting up python-glade2 (2.24.0-3ubuntu3) ...
Setting up guake (0.4.4-1ubuntu1) ...
{% endhighlight %}
خوش باشین.
