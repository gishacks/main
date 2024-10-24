---
title: جلسه ۲ - وکتور

level: secret
---
## کار با Pandas
### ۱. نصب

برای نصب کتابخانه Pandas بعد از فعال کردن محیط مجازی پرژه دستور زیر را در ترمینال وارد کنید:

```
pip install pandas
```

!!! نکته
    در محیط Jupyter Notebook با قرار دادن علامت تعجب قبل از دستور فوق کتابخانه در محیط فعال Jupyter نصب می‌شود
    
    ```
    ! pip install pandas
    ```
    

### ۲. استفاده از دستورهای Pandas
برای استفاده از دستورهای کتابخانه Pandas در کد پایتون باید ابتدا آن را با دستور زیر فرا بخوانید

```python
import pandas as pd
```
از این پس هرجا از مخفف pd استفاده می‌کنید، دستورهای pandas فراخوانده می‌شود.


### ۳. کار با پایگاه‌های داده

1. دانلود فایل نمونه

    داده مربوط به زمین‌لرزه‌های یک ماه اخیر را از طریق [این لینک](https://earthquake.usgs.gov/earthquakes/search/) با فرمت csv دانلود و در آدرس repo خود ذخیره کنید.

2. خواندن فایل

    با دستور زیر می‌توانید فایل csv دانلود شده را در قالب یک DataFrame ذخیره کنید.

    ```python
    df = pd.read_csv('query.csv')
    ```
3. اطلاعات کلی درباره فایل

    با دستورهای زیر می‌توانید اطلاعات کلی از DataFrame تعریف شده دریافت کنید.

    ```python
    df.head()
    df.columns
    df.shape
    df.describe()
    df.info()
    df.iloc[0]
    ```
4. گزارش گیری

    با دستورهای زیر می‌توانید از جدول مدنظر خود گزارش بگیرید.
    ```python
    df.groupby('magSource').mean('mag')
    df.sort_values('mag', ascending=False)
    df['rank']=df['mag'].rank(ascending=False)
    ```
5. فیلتر کردن 

    با دستورهای زیر می‌توانید جدول خود بر اساس مشخصات مدنظر فیلتر کنید.
    ```python
    filrered = df[df['mag']>5]
    filrered = df[df['place'].str.contains('Iran')]
    ```
6. خروجی گرفتن

    با دستور زیر می‌توانید از DataFrame مدنظرتان خروجی بگیرید.
    ```python
    filrered.to_csv('alert.csv', index=False)
    ```



## کار با GeoPandas و MatPlotLib
### ۱. نصب و فراخواندن


برای نصب کتابخانه‌های GeoPandas و MatPlotLib بعد از فعال کردن محیط مجازی پرژه دستور زیر را در ترمینال وارد کنید:

```
pip install geopandas
pip install matplotlib
```
برای استفاده از دستورهای کتابخانه GeoPandas در کد پایتون باید ابتدا آن را با دستور زیر فرا بخوانید

```python
import geopandas as gpd
import matplotlib.pyplot as plt
```

### ۲. خواندن فایل‌های مکانی
با دستور زیر می‌توانید فایل‌های مکانی را به عنوان GeoDataFrame در کد خود فرا بخوانید.
```python
parcel_data_1375 = gpd.read_file(r"data\R01M4.shp")
parcel_data_1385 = gpd.read_file(r"data\R01M5.shp")
```
### ۳. چک کردن سیستم مختصات 
اگر از بیش از یک لایه مکانی استفاده می‌کنید قبل از هر کار با دستور زیر سیستم مختصات لایه‌ها را چک کنید و از تطابق آن‌ها اطمینان کسب کنید.
```python
parcel_data_1375.crs
parcel_data_1385.crs
```
در صورت عدم تطابق سیستم‌های مختصاتی می‌توانید با دستور زیر آن‌ها را با سیستم مدنظر تطبیق دهید.
```python
parcel_data_1375 = parcel_data_1375.to_crs(parcel_data_1385.crs)
```
!!! نکته
    با دستور زیر می‌توانید عمل چک و تطبیق سیستم مختصاتی را با هم انجام دهید.
    
    ```python
    if parcel_data_1375.crs != parcel_data_1385.crs:
        parcel_data_1375 = parcel_data_1375.to_crs(parcel_data_1385.crs)
    ```
### ۴. دستورهای GIS با GeoPandas

1. Spatial Join
    ```python
    joined_data = gpd.sjoin(parcel_data_1375,parcel_data_1385)
    ```
2. Select by Attribute
    ```python
    changed_parcels = joined_data[joined_data['luShora_left'] != joined_data['luShora_right']]
    ```
3. Summerize
    ```python
    changed_parcels.groupby('luChange').count()
    ```
!!! نکته
    علاوه بر groupby می‌توانید با دستور زیر از قابلیت pivot برای گزارش‌گیری استفاده کنید.        
    ```python
    changed_parcels.pivot_table(index='luShora_left', columns='luShora_right',
        aggfunc='count')
    ```

### ۵. خروجی گرفتن

1. خروجی تصویری

    با دستور زیر می‌توانید از DataFrame مدنظر خود خروجی نقشه به صورت عکس بگیرید و به صورت عکس ذخیره کنید.
    ```python
    changed_parcels.plot("luChange", cmap="Blues")
    plt.savefig('draft/luChanges.jpg')
    ```

2. خروجی وب
    با دستور زیر می‌توانید از DataFrame مدنظر خود خروجی نقشه به صورت صفحه وب بگیرید و به صورت فایل HTML ذخیره کنید.
    ```python
    map = changed_parcels.explore("luChange", cmap="Blues")
    map.save('draft/map.html')
    ```

## تمرین
در کد زیر x و y چه چیز ویژگی از قطعه را در خود ذخیره می‌کنند.

```python
import geopandas as gpd
import shapely
import math
parcels = gpd.read_file("parcel.shp")
for idx, feature in parcels.iterrows():
    parcel = feature['geometry'].simplify(tolerance=0.05)
    coords = list(parcel.boundary.coords)
    temp = []
    for a, b in zip(coords,coords[1:]):
        c = shapely.LineString([a,b]).length
        d = math.degrees(math.atan2(b[1]-a[1], b[0]-a[0]))
        temp.append({'c': c, 'd':d})    
    e = max(temp, key=lambda x: x['c'])
    parcels.loc[idx, 'x']= e['c']
    parcels.loc[idx, 'y']= e['d']
parcels.to_file('parcelx.shp')
```

!!! نکته
    ۱. کتابخانه [shapely](https://shapely.readthedocs.io/en/stable/manual.html) یکی از کتابخانه‌های پرکاربرد پایتون است که بیشتر برای ترسیم عوارض وکتوری استفاده می‌شود. این کتابخانه همراه GeoPandas نصب می‌شود و نیازمند نصب مجدد نیست.

        ```python
        point = shapely.Point(x,y)
        line = shapely.LineString([(x1,y1), (x2,y2)])
        polygon = shapely.Polygon([(x1,y1), (x2,y2), (x3,y3)])
        ```
    ۲. کتابخانه [math](https://docs.python.org/3/library/math.html) برای انجام محاسبات ریاضی استفاده می‌شود. این کتابخانه از کتابخانه‌های درونی پایتون است و همراه با پایتون نصب می‌شود.


    ۳. دستور [zip](https://www.w3schools.com/python/ref_func_zip.asp) تناظر یک به یک بین عناصر هم جایگاه در دو [tuple](https://www.w3schools.com/python/python_tuples.asp) را ایجاد می‌کند.