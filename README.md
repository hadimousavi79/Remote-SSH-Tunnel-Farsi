<h1 id="rtl-markdown" dir="rtl">آموزش راه اندازی تونل معکوس بین 2 سرور با استفاده از SSH و اتوماسیون با Supervisor</h1>

این نوع تونل اصولا در شرایطی استفاده میشود که سرور مد نظر ما پشت firewall بسته و یا NAT باشد
شرایط لازم این است که سرور شما پورت های ورودی tcp آن توسط یکی از مواردی که گفته شد بسته شده باشن، اما پورت های خروجی tcp آن باز باشند.
مفهوم انجام این نوع تونل بدین شرح است که ما از طریق سرور فیلتر شده به سرور آزاد ارتباط SSH برقرار میکنیم، سپس با استفاده از همین ارتباط پایدار به صورت معکوس پورت tcp مورد نظر را از سرور آزاد به پورت tcp سرور فیلتر شده هدایت میکنیم (port forwarding)، در نتیجه کاربرات با استفاده از پورت tcp هدایت شده از سرور آزاد میتوانند به سرور فیلتر شده دسترسی داشته باشن.

اگر سوال و مشکلاتی داشتید میتونید تلگرام ازم بپرسید:
<a href="t.me/nillkiggersfurnbaggotskatehikes">nillkiggersfurnbaggotskatehikes@</a>


**تصویر مفهومی برای توضیح بهتر این نوع تونل:**
<p dir="rtl">client = سرور فیلتر</p>
<p dir="rtl">gateway = سرور آزاد</p>
<p dir="rtl">precious resource = سرویس مورد نظر</p>

<p dir="rtl"><img src="https://user-images.githubusercontent.com/104469759/235637820-ae1e5c76-78bc-4c4b-ab51-129c517dde34.png" alt="remote ssh tunnel" title="Remote SSH Tunnel Concept"></p>


<h4 dir="rtl">توجه داشته باشید که تمام فرمان ها در این آموزش باید با یوزر root اجرا شوند
اگر دسترسی روت ندارید میتونید با اجرای فرمان زیر به یوزر روت سوییچ کنید سپس به اجرای فرمان ها بپردازید.</h4>
<code>sudo -i</code>


<h2 dir="rtl">1. آماده سازی سرور آزاد</h2>
<p dir="rtl">توجه: این مرحله روی سرور آزاد انجام میشه</p>

فرمان زیر امکان گوش دادن به درخواست های ورودی به سرور آزاد را توسط پورت های فوروارد شده از طریق تونل باز میکند:

<code>echo "GatewayPorts yes" >> /etc/ssh/sshd_config</code>

سپس برای اعمال تغییری که تو تنظیمات sshd دادید این فرمان رو اجرا کنید:

<code>systemctl restart sshd.service</code>


<h2 dir="rtl">2. تهیه کلید SSH درون سرور فیلتر شده و شناخت آن به سرور آزاد</h2>
<p dir="rtl">توجه: این مرحله روی سرور فیلتر انجام میشه</p>

(در اکثر مواقع برای دسترسی به سرور فیلتر شده شما نمیتونید ssh مستقیم بزنید تا فرمان های لازم رااجرا کنید، مگر این که به طریقی شما بتونید دسترسی داشته باشید بهش که معمولا مجبور هستید از صفحه Console در پنل تهیه کننده هاست استفاده کنید که اگر قبلا با آن کار کرده باشید میدانید که paste کردن فرمان ها در آن ممکن نیست، در نتیجه پیشنهاد میشود از نرم افزار <a href="https://github.com/Collective-Software/ClickPaste">Clickpaste</a> استفاده کنید که متن یا فرمان کپی شده را با 2 کلیک ساده برای شما به جای paste کردن **تایپ** میکند)

با فرمان زیر یک کلید جفتی ssh از نوع rsa تولید میکنیم:

<code>ssh-keygen -t rsa</code>


در خصوص آدرس و نام برای ثبت کلید چیزی ننویسید و خالی کلید Enter را بزنید، همچنین برای Passphrase درخواستی هم خالی کلید Enter را بزنید.
در انتها کلید جفتی با اسامی پیشفرض id_rsa و id_rsa.pub در مسیر root/.ssh/ ساخته میشه.

(فایل id_rsa کلید خصوصی شما و id_rsa.pub کلید عمومی شما میباشد، شما با قرار دادن محتویات داخلی کلید عمومیتان در هر سروری میتوانید با استفاده از کلید خصوصی خود بدون نیاز به پسورد و با ایمنی تمام ارتباط ssh برقرار کنید)

حال با فرمان زیر کلید عمومی ساخته شده در سرور آزاد کپی میشود:

<code>ssh-copy-id -i ~/.ssh/id_rsa.pub root@آی.پی.سرور.آزاد</code>


از این رو که اولین باری هست که سرور فیلتر به سرور آزاد ارتباط ssh ایجاد میکند از شما درباره اعتماد به فینگرپرینت سرور آزاد سوال میشود که باید جواب دهید:

<code>yes</code>

سپس با وارد کردن پسورد سرور آزاد، این روند تکمیل میشود که در نتیجه دفعات بعدی شما میتوانید بدون استفاده از پسورد از درون سرور فیلتر به سرور آزاد دسترسی داشته باشید

در حال حاضر متوجه شدید که بعد از انجام عملیات کپی کلید، وارد سرور آزاد شده و داره بهتون شل سرور آزاد رو نشون میده، ما باقی مراحلمونم هنوز رو سرور فیلتر باید انجام بشه پس بنویسید exit و از سرور آزاد خارج بشید که برگرده رو سرور فیلتر تا ادامه بدیم.

<p dir="rtl">*نکته، محض اطلاع: این روند به صورت اتوماتیک انجام میشود، مقصد کپی کردن کلید در سرور آزاد  فایل ذکر شده میباشد:*

/root/.ssh/authorized_keys</p>

<h2 dir="rtl">3. نصب سرویس منیجر supervisor و ایجاد تونل</h2>
<p dir="rtl">توجه: این مرحله روی سرور فیلتر انجام میشه</p>

(به صورت مجزا میتوانید مشابه همین تنظیمات رو با خود systemd هم انجام دهید اگر اطلاعات کافی برای ساخت سرویس با اون روش را دارید)

نصب supervisor:

<code>apt update && apt-get install supervisor -y</code>

ساخت جاب برای برای supervisor: (میتونید نام رو با اسم دل بخواه برای خودتون جایگزین کنید، مثلا من زدم vless-reverse)

<code>nano /etc/supervisor/conf.d/نام.conf</code>

سپس داخل آن مشابه زیر سرویس را بسازید:

<code>[program:نام]
command=ssh -N -R 0.0.0.0:پورت:localhost:پورت root@آی.پی.سرور.آزاد
directory=/root
user=root
autostart=true
autorestart=true
stdout_logfile=/var/log/supervisor/نام.log
redirect_stderr=true
numprocs=1</code>

<h3>*** توجه: این متن در ادامه فقط برای توضیح هست !!! ***</h3>

<p>اینجا من کامند sshی که استفاده شده رو براتون یه توضیحی میدم فقط محض رفع شک و شبهه ها</p>
<p>ssh -N -R 0.0.0.0:99:localhost:99 root@آی.پی.سرور.آزاد</p>
<p>این کامند رو به 2 قسمت براتون تقسیم میکنم</p>
<p>قسمت فاقد دستور تونل میشه</p>
<p>ssh -N root@آی.پی.سرور.آزاد</p>
<p>اول همه این قسمت اجرا میشه</p>
<p>وقتی کهssh کامل وصل شد به سرور مد نظر</p>
<p>تازه قسمت دستور تونل اجرا میشه</p>
<p>-R 0.0.0.0:99:localhost:99</p>
<p>در اینجا هم 0.0.0.0:99 روی سرور آزاد تنظیم میشه که تایین میکنه از تمام ای پی های ممکن روی پورت 99 درخواست ها رو قبل کن و هدایت کن به سرور خودمون</p>
<p>در اصل به localhost:99 سرویسیه که رو پورت 99 رو سرور فیلتر شده داره کار میکنه و به درخواست هایی که بهش هدایت میشن جواب میده.</p>


<p> </p>
<h3>*** توجه: توضیحات به اتمام رسید، ادامه مراحل را پیگیری کنید !!! ***</h3>
<p> </p>



<p>بعد از ذخیره فایل برای بازخوانی جاب های جدید در supervisor از این فرمان استفاده کنید:</p>

<code>supervisorctl reread</code>

برای راه اندازی مجدد سرویس supervisord:

<code>supervisorctl reload</code>

برای مشاهده وضعیت جاب های فعال در supervisor:

<code>supervisorctl status</code>


در نهایت جابی که تنظیم کردید اجرا میشه و میتونید روندشونو توی htop مشاهده کنید


<h2 dir="rtl">4. ایجاد اتوماسیون برای supervisorctl reload</h2>
<p dir="rtl">توجه: این مرحله روی سرور فیلتر انجام میشه</p>

اگر تونل برای شما پایدار نیست و بعد چند ساعت قطع میشه و باید بیاید دستی ریستارت کنید تونل رو، 
با این روش میتونید فرمان ریلود سوپروایزر رو هر 1 ساعت به صورت اتوماتیک اجرا کنید: 

 ابتدا مطمئن شوید که crontab روی سیستم شما نصب هست:

<code>apt update && apt-get install cron -y</code>

سپس با اجرای این فرمان تنظیمات جاب های crontab را باز کنید:

<code>crontab -e</code>

در این مرحله از شما میپرسد کدام تکست ادیتور را برای مشاهده فایل ترجیح میدهید، 
برای سهولت امر از nano که معمولا گزینه 1 هست استفاده کنید

حالا در انتهای فایل باز شده این خط را اضافه کنید:

<code>0 */1 * * * supervisorctl reload</code>

 
<h2 dir="rtl">5. تست کردن وضعیت تونل روی سرور آزاد</h2>
<p dir="rtl">توجه: این مرحله روی سرور آزاد انجام میشه</p>
<p dir="rtl">مهم ترین تست اینه که رو سرور آزاد باید بتونید با وارد کردن فرمان زیر:</p>

<code>lsof -i -P -n | grep LISTEN</code>


<p dir="rtl">تمام پورت هایی که روی سرور آزاد در حال listen شدن هستن رو مشاهده کنید</p>

<p dir="rtl">که لازمه برای موفقیت آمیز بودن تونل معکوسمون این پورت به این شکل 99:* باز باشه (* = 0.0.0.0 = "هر ای پی ای")</p>

<p dir="rtl">چیزی مشابه 2 خط زیر رو باید بتونید ببینید:</p>

<code>sshd   1111 root  10u IPv4 2222   0t0 TCP *:99 (LISTEN)</code>


<p dir="rtl">در نهایت هم یادتون باشه همیشه ریبوت کردن سرور آزاد میتونه مشکلات قطعی تونل رو حل کنه</p>
<p dir="rtl">برای ریبوت کردن هر سیستم لینوکسی فرمان زیر را استفاده کنید:</p>
<code>reboot</code>


اگر سوال و مشکلاتی داشتید میتونید تلگرام ازم بپرسید:
<a href="t.me/nillkiggersfurnbaggotskatehikes">nillkiggersfurnbaggotskatehikes@</a>
