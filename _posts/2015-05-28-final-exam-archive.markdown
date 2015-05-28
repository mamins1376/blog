---
layout: post
title:  "آرشیو کردن نمونه سوالات امتحان نهایی"
date:   2015-05-28 15:50:08
categories: final exam bash script
---
الآن فصل امتحاناته و منم خیلی کمتر اینجا می‌نویسم. اخیرا خواستم سوالات امتحانات نهایی رو کامل و یکجا داشته باشم: تمیز و مرتب. سایت [کنکور][konkur] سوالات رو برای دانلود گذاشته ولی چند تا مشکل دارن:

۱- فایل‌های اضافه داره(لینک و تصویر تبلیغاتی و ...)
۲- همه‌ی فایل‌ها با فرمت rar هستن که پسورد هم دارن
۳- خیلی زیادن!

از اونجا که من هم وقت استخراج تک‌تک اونها رو ندارم هم حالشو، یه اسکریپت نوشتم که این‌کارها رو برام انجام بده. خوش‌بختانه فایل‌های روی سرور اسماشون قاعده داره و منم از همین استفاده می‌کنم تا با داشتن قالب کلی هر لینک، کل فایل‌ها رو در اختیار داشته باشم.

این اسکریپت بش، کارهای زیر رو برامون انجام میده:

۱- دانلود فایل‌ها
۲- استخراجشون. متأسفانه نتونستم با unrar-free کنار بیام و unrar-nonfree رو به کار بردم
۳- حذف فایل‌های اضافه
۴- تغییر نام فایل‌ها. تمیز بودن کلا برای من خیلی اهمیت داره ؛)
۵- و در نهایت آرشیو کردنشون توی یه فایل zip.

{% highlight bash %}
#!/bin/bash
EXAM=$1

if [ "x$EXAM" = "x" ]; then
  echo 'Please give a name! e.g. Adabiyat'
  exit 1
fi

# 1st: Download
for YEAR in `seq 87 93`
do
  KHORDAD="http://dl.konkur.in/post/Exam/3/$EXAM/Khordad-$YEAR-$EXAM-%5Bwww.konkur.in%5D.rar"
  SHAHRIVAR="http://dl.konkur.in/post/Exam/3/$EXAM/Shahrivar-$YEAR-$EXAM-%5Bwww.konkur.in%5D.rar"
  DEY="http://dl.konkur.in/post/Exam/3/$EXAM/Dey-$YEAR-$EXAM-%5Bwww.konkur.in%5D.rar"
  wget -c "$KHORDAD" "$SHAHRIVAR" "$DEY"
done

# 2nd: Extract them
for RAR in *.rar
do
  unrar e -pwww.konkur.in -o+ -y "$RAR"
done

# 3rd: Clear area
rm *.rar *.URL *.png

# 4th: Clear filenames
rename s/'-\[www.konkur.in\]'//g *
rename s/-/\ /g *
rename s/\ $EXAM//g *

# finally archive them in zip format
for ITEM in Khordad Shahrivar Dey
do
  mkdir $ITEM
  mv ${ITEM}*.pdf $ITEM
done

zip -r ${EXAM}.zip Khordad Shahrivar Dey

rm -rf Khordad Shahrivar Dey
{% endhighlight %}

[konkur]: http://konkur.in/
