---
title: جلسه اول - مبانی
level: secret
---

## راه‌اندازی محیط کار
### ۱. نصب پایتون:

آخرین نسخه پیاتون را از [این لینک](https://www.python.org/downloads/) دانلود و نصب کنید.

!!! نکته
    در مراحل نصب *حتماً* تیک Add Python to Path را بزنید.
        <figure markdown>
        ![](assets/pygis101-1.png)
        <figcaption></figcaption>
        </figure>

برای اطمینان از نصب درست دستور زیر را در محیط Command Prompt وارد کنید.

``` 
python --version
```

### ۲. ساخت محیط مجازی

```
python -m venv name
```
به جای name در دستور بالا نام محیط مجازی مدنظر را وارد کنید. برای فعال‌سازی محیط مجازی دستور زیر را وارد کنید:
```
name\Scripts\activate
```

### ۳. نصب vscode

آخرین نسخه نرم‌افزار vscode را از [این لینک](https://code.visualstudio.com/download) دانلود و نصب کنید.

### ۴. راه‌اندازی GitHub

a. در [وب‌سایت GitHub](https://github.com/) اکانت بسازید و بعد از نهایی شدن ثبت‌نام یک repository به صورت public برای تمرین‌های کلاس بسازید.

b. بعد از این‌کار نرم‌افزار GitHub Desktop را از [این لینک](https://desktop.github.com/download/) دانلود و نصب کنید و بعد از نصب درون نرم‌افزار وارد اکانت گیت‌هاب خود شده و repository که در مرحله قبل ساختید clone کنید.

c. از منوی Repository گزینه Open in Visual Studio Code را انتخاب کنید.

### ۵. نصب Jupyter Lab

a. در نرم‌افزار vscode از پنل Extentions افزونه‌های Python و Jupyter را نصب کنید.

b. محیط terminal را از منوی View فرا بخوانید و در پنل اضافه شده در پائین نرم‌افزار، با دستور مرحله ۲، یک Virtual Environment بسازید و آن را فعال کنید. 

c. بعد از فعال‌سازی محیط مجازی، در ترمینال، با دستور زیر Jupyter Lab را نصب کنید.

    pip install jupyterlab

d. با میان‌بر زیر صفحه دستورهای vscode را فرا بخوانید و با جستجوی jupyter گزینه Create: New Jupyter Notebook را پیدا و انتخاب کنید.

    Ctrl + Shift + P

e. بعد از باز شدن صفحه Jupyter Notebook گزینه Select Kernel را زده و از Existing Python Environment محیط مجازی ساخته شده در مرحله قبل را انتخاب کنید.

<figure markdown>
![](assets/pygis101-2.png)
<figcaption></figcaption>
</figure>


f. برای اطمینان از درستی انجام مراحل قبل اولین کد پایتون خود را بنویسید و با زدن Ctrl + Enter آن را اجرا کنید.

    print('Hello World!')
    
## شروع برنام‌نویسی
### چهار عمل اصلی
۱. تعریف متغیر:

    ```python
    name = 'Tayebi'
    print(name)
    ```

۲. حلقه For

    ```python
    for chr in name:
        print(chr)
    ```
    
۳. منطق شرطی If

    ```python
    for chr in name:
        if chr == 'a':
            print(True)
        else:
            print(False)
    ```

    افزودن شمارنده

    ```python
    count = 0
    for chr in name:
        if chr == 'a':
            count = count + 1
    print('Number of a: '+str(count))
    ```

۴. تعریف تابع Function

    ```python
    def countChr(c, t):
        t = t.lower()
        count = 0
        for chr in t:
            if chr == c:
                count += 1
        print('Number of '+c+': '+ str(count))
    countChr('d', name)
    ```

### کار با فایل
۱. تعداد فراوانی یک کلمه در یک فایل نوشتاری

    ```python
    fhand =open('hamlet.txt')
    hamlet = fhand.read()
    text = hamlet.split()

    count = 0
    for word in text:
        word = word.lower()
        if word == 'hamlet':
            count += 1
    print(count)
    ```

۲. ایجاد ابر کلمات

    ```python
    words = {}
    for word in text:
        word = word.lower()
        if word in words:
            words[word]+=1
        else:
            words[word] = 1
    print(words)
    ```
