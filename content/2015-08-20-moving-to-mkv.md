+++
title = "مهاجرت به قالب ماتروسکا"
date = 2015-08-20
category = "لینوکس"
tags = ["mkv", "Matroska", "ماتروسکا", "لینوکس", "اسکریپت"]

+++

به قصد پاکسازی سیستمم، تصمیم گرفتم تا جای ممکن از قالب‌ها، نرم‌افزارها و مجوزهای آزاد استفاده کنم. در راستای این مهم، [ماتروسکا][Matroska] رو برای پرونده‌های تصویری انتخاب کردم. قالب خیلی جالبیه و همه‌جا هم به راحتی قابل پخشه و البته شناخته شده(با پسوند mkv). یکی از مزایایی هم که داره کم‌تر بودن حجمش نسبت به امپگ هست. اسکریپتی برای پاکسازی سیستم از قالب منحوس و شوم(!) MP4 نوشتم که تقریبا هوشمندانه عمل می‌کنه. البته به قول [جادی][Jadi]، ما هر چیزی رو که براش if می‌ذاریم بهش می‌گیم هوشمند :)

نحوه‌ی کار به این صورته که `find`، پرونده‌هایی رو که پسوند MP4 دارند رو داخل پوشه‌ی Videos در پوشه‌ی خانگی کاربر جاری پیدا می‌کنه و با یه حلقه‌ی `for` مانند (که با `while` پیاده شده)، تابع `convert_to_mkv` رو به ازای هر کدوم از این پرونده‌ها صدا می‌زنه. این تابع، موظفه که برنامه‌ی `avconv` رو اجرا کنه که عمل اصلی تبدیل رو انجام می‌ده. بقیه‌ی کارها هم مختص فرستادن نامه به کاربره که گزارش تبدیل رو به کاربر جاری ایمیل می‌کنه.

```bash
#!/bin/sh
# make cron usage easy :)
export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
# go to the user's videos directory
cd /home/$USER/Videos/

# this function converts given filename
convert_to_mkv() {
  INPUT="$1"
  OUTPUT="$(echo $1 | sed -r -e 's/\.\w+$/.mkv/')"
  avconv -v info -i "$INPUT" -c:v libx264 -f matroska "$OUTPUT"
}

# find any mp4 and kill it!
find . -type f -name '*.mp4' | while read FULL_PATH; do
  FILE="$(echo "$FULL_PATH" | sed 's/\.\///')"
  LOGFILE="$(basename "${FILE}" | sed -e 's/ /_/g' -e 's/mp4$/log/')"
  if convert_to_mkv "$FILE" < /dev/null >> "$LOGFILE" 2>&1; then
    echo "$FILE converting successfully finished. see attachment for details." | mail -A "$LOGFILE" -s 'Converting Finished' $USER
    rm "$LOGFILE" "${FILE}"
  else
    echo "$FILE converting faild. see attachment for details." | mail -A "$LOGFILE" -s 'Error While Converting' $USER
  fi
done
```

در پست‌های بعدی، درباره‌ی مهاجرت به قالب‌های آزاد دیگه (نظیر Vorbis و ...) خواهم نوشت.


[Matroska]: https://fa.wikipedia.org/wiki/%D9%85%D8%A7%D8%AA%D8%B1%D9%88%D8%B3%DA%A9%D8%A7
[Jadi]: http://jadi.net/
