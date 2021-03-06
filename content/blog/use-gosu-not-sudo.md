+++
date = "2016-11-16T14:54:13+04:30"
draft = false
title = "چرا بجای sudo بهتر است از gosu استفاده کنید؟"

+++

چرا بجای sudo بهتر است از gosu استفاده کنید؟
===

*جهت اجرای دستورات گفته شده در این مقاله نیاز دارید قبلا داکر را نصب کرده باشید. روش نصب داکر روی [ویندوز](http://elastico.io/blog/install-docker-windows.html) یا [لینوکس CentOS](http://elastico.io/blog/install-docker-centos7.html) را میتوانید در همین سایت مطالعه کنید. همچنین برای یادگیری بهتر این مطلب ممکن است آشنایی با [مفاهیم پایه ای داکر](http://elastico.io/blog/docker-basic-concepts.html) به شما کمک کند.*

ابزار سنتی لینوکس برای اجرای دستورات تحت یک کاربر خاص sudo نام دارد و به احتمال زیاد شما تابحال به دفعات از آن استفاده کرده اید. اگر چه sudo ابزار قوی و کاراییست، محدودیتهایی دارد که آنرا برای استفاده در کانتینرها غیر ایده آل میکند. به عنوان مثال، اگر دستور `sudo ps aux` را در یک کانتینر Ubuntu اجرا کنید خروجی آن به صورت زیر خواهد بود:

```
$ docker run --rm ubuntu:trusty sudo ps aux 
USER PID ... COMMAND 
root   1     sudo ps aux 
root   5     ps aux
```

همانطور که مشاهده میکنید در اینجا دو برنامه در داخل کانتینر در حال اجرا هستند، یکی sudo و دیگری دستوری که با استفاده از آن اجرا کردیم. یکی از مشکلاتی که در این حالت ایجاد میشود این است که سیگنالهایی که به کانتینر شما فرستاده شود بجای پردازه اصلی که برنامه مورد نظر شماست، به پردازه sudo فرستاده میشود.

حالا اگر همان دستور قبلی یعنی `ps aux` را با کمک ابزار gosu که روی یک کانتینر Ubuntu نصب شده است اجرا کنید نتیجه به شکل دیگری خواهد شد:

```
$ docker run --rm amouat/ubuntu-with-gosu gosu root ps aux
USER PID ... COMMAND
root   1     ps aux
```

همانطور که مشاهده میکنید تنها یک پردازه در داخل کانتینر در حال اجراست که برنامه مورد نظر ماست و اثری از gosu نیست. مهمتر اینکه شناسه این پردازه (PID) شماره ۱ است یعنی تمامی سیگنالهایی که به این کانتینر فرستاده شود به این پردازه خواهد رسید و مشکلات sudo را نخواهید داشت.

با کمک ابزار gosu، یکی از روشهایی که بتوانید به راحتی برنامه مورد نظرتان را در داخل کانتینر و تحت یک کاربر خاص اجرا کنید استفاده از یک اسکریپت bash به صورت زیر به عنوان entry-point است:

```
#!/bin/bash
set -e
if [ "$1" = 'myprogram' ]; then 
    chown -R myuser .
    exec gosu myuser "$@"
fi
exec "$@"
```

بکارگیری exec و gosu به طور همزمان تضمین میکند که برنامه شما به عنوان پردازه شماره ۱ در داخل کانتینر اجرا خواهد شد.
