---
title: جلسه ۳ - رستر
level: secret
---
## کار با Rasterio
### ۱. نصب

بعد از نصب کتابخانه rasterio در قالب دستور with فایل‌های رستری مورد نیاز برای پروژه این جلسه را باز می‌کنیم.
```python
import rasterio

with rasterio.open('landsat2014_B5.TIF') as src:
    nir2014 = src.read()
    crs2014 = src.crs
with rasterio.open('landsat2014_B4.TIF') as src:
    red2014 = src.read()
with rasterio.open('landsat2024_B5.TIF') as src:
    nir2024 = src.read()
    crs2024 = src.crs
with rasterio.open('landsat2024_B4.TIF') as src:
    red2024 = src.read()
```

!!! نکته
    ۱. علت استفاده از دستور with: در پایتون وقتی از فایل بیرونی استفاده می‌کنیم بعد از باز کردن فایل باید فایل بسته شود. با کمک دستور with عملیات بازکردن و بستن فایل به طور خودکار انجام می‌شود.

    ۲. عکس‌های ماهواره‌ای را می‌توان علاوه بر سرویس‌های عمومی مانند Google و Bing از مراجع حرفه‌ای مانند USGS و Copernicus دریافت کرد. عکس‌های ماهواره‌ای حرفه‌ای مانند Landsat علاوه بر سه Band قرمز، سبز، و آبی دارای بندهای دیگری‌اند که در تحلیل‌های سنجش از راه دو استفاده می‌شود.

    ۳. برای انجام تمرین این جلسه ما به دو بند قرمز و نزدیک مادون قرمز (NIR) نیاز داریم. برای آشنایی با بندهای عکس‌های ماهواره‌ای Landsat می‌توانید به این آدرس مراجعه کنید.

    ۴. وقتی از چند لایه‌ رستری استفاده می‌کنید. باید از مطابقت سیستم‌های مختصات لایه‌های اطمینان کسب کنید. 

    ```python
    print(crs2014 == crs2024)
    ```

    در صورت عدم تطبیق سیستم‌های مختصاتی برای انطباق آن‌ها می‌توانید از تابع زیر استفاده کنید.

    ```python
    def reproject_image(input_image, output_image, dst_crs='EPSG:4326'):
        with rasterio.open(input_image) as src:
            transform, width, height = calculate_default_transform(
                src.crs, dst_crs, src.width, src.height, *src.bounds)
            kwargs = src.meta.copy()
            kwargs.update({
                'crs': dst_crs,
                'transform': transform,
                'width': width,
                'height': height
            })
            
            with rasterio.open(output_image, 'w', **kwargs) as dst:
                for i in range(1, src.count + 1):
                    reproject(
                        source=rasterio.band(src, i),
                        destination=rasterio.band(dst, i),
                        src_transform=src.transform,
                        src_crs=src.crs,
                        dst_transform=transform,
                        dst_crs=dst_crs,
                        resampling=Resampling.nearest)

    ```

    

### ۲. محاسبه شاخص NVDI
شاخص NVDI برای تحلیل پوشش گیاهی در عکس‌های ماهواره‌ای به کار گرفته می‌شود و برای محاسبه آن به بندهای قرمز و نزدیک مادون قرمز نیاز داریم. برای محاسبه آن می‌توانیم از تابع زیر استفاده کنیم. 

```python
def calculate_ndvi(nir, red):
    ndvi = (nir - red) / (nir + red)
    return ndvi

ndvi2014 = calculate_ndvi(nir2014, red2014)
ndvi2024 = calculate_ndvi(nir2024, red2024)

```


### ۳. محاسبه میزان توسعه شهری
یکی از کاربردهای شاخص NVDI محاسبه میزان توسعه شهرها است. چون با توسعه شهرها معمولاً زمین‌های کشاورزی و جنگلی پیرامون آن‌ها از بین می‌رود با سنجش میزان تغییر NVDI می‌تواند این پدیده را اندازه‌گیری کرد. برای این کار شاخص NVID دو سال مدنظر را از هم کم کرده و با استفاده از تابع where از کتابخانه Numpy آن را طبقه‌بندی مجدد می‌کنیم.    

```python
import numpy as np

nvdi_change = ndvi2024 - ndvi2014
classified_nvdi_change = np.where(nvdi_change>0.1,1,np.where(nvdi_change <-0.1,-1, 0))
```

### ۴. نمایش خروجی
همانطور که در جلسه پیش توضیح داده شد، یکی از کتابخانه‌های رایج برای تبدیل داده‌های توصیفی به نمودهای ترسیمی کتابخانه matplotlib است. برای گرفتن خروجی از لایه رستری از دستور imshow() از زیرمجموعه این کتابخانه استفاده می‌کنیم.

```python
classified_nvdi_change = classified_nvdi_change.squeeze()
plt.imshow(classified_nvdi_change, cmap='RdYlGn')
plt.colorbar()
plt.title('Land Use Change (2014-2024)')
plt.show()
```
!!! نکته
    علت استفاده از دستور squeeze قبل از گرفتن خروجی داشتن بعد اضافی در لایه رستری است. برای بدست آوردن شکل لایه رستری می‌توانید از دستور زیر استفاده کنید.

    ```python
    classified_nvdi_change.shape
    ```

    همانطور که می‌بینید خروجی (1, 7781, 7651) نشان می‌دهد که دارای یک بعد اضافی است. دستور squuze شکل درست (7781, 7651) یعنی لایه رستری شامل ۷۷۸۱ ردیف و ۷۶۵۱ ستون را به ما برمی‌گرداند.

### ۵. ذخیره خروجی
برای ذخیره فایل خروجی در قالب رستر نیاز به تعیین metadata است. برای این کار می‌توانیم از metadata لایه مبنا ورودی استفاده کنیم. بنابراین در باز کردن لایه مبنا علاوه بر محتوای رستری profile آن را نیز به صورت یک متغییر ذخیره می‌کنیم.

```python
with rasterio.open('landsat2014_B5.TIF') as src:
    nir2014 = src.read()
    profile = src.profile
```

با داشتن profile به عنوان یک متغیر می‌توانیم با دستور زیر از لایه نهایی خروجی بگیریم.

```python
with rasterio.open('nvdi_change.tif', 'w', **profile) as dst:
    dst.write(classified_nvdi_change.astype(rasterio.int32), 1)
```

### تمرین
تابعی را تعریف کنید که لایه نقاط و لایه رستر را گرفته و ارزش هر یک از نقاط را برای در لایه رستری در یک فیلد ذخیره کند.

!!! راهنما
    تابع زیر ارزش موجود در لایه raster را برای مختصات xy محاسبه می‌کند

    ```python
    def getRasterValue(xy, raster, transform):
        xy = list(xy)
        row, col = ~transform * (xy[1], xy[0])
        row, col = int(row), int(col)
        value = raster[row,col]
        return value
    ``` 


## کار با RSGISLib
### ۱. نصب و راه‌اندازی
متأسفانه کتابخانه rsgislib را نمی‌توان با دستور pip نصب کرد و برای نصب آن باید از conda استفاده کرد. از انجا که کتابخانه‌های Anaconda قابل نصب در محیط مجازی نیستند و تنها در محیط‌های conda نصب می‌شوند برای استفاده از این پکیج باید Anaconda نصب و محیط conda ساخته شود.

۱. نصب Anaconda

    با راهنمای موجود در سایت Anaconda این نرم‌افزار را نصب کنید.

۲. ساخت محیط جدید در Anacona Prompt

    بعد از نصب Anaconda ترمینال Anaconda Prompt را باز کنید و با دستور زیر یک محیط جدید به نام دلخواه (gisenv) بسازید.

    ```
    conda create -n gisenv
    ```


۳. فعال‌سازی محیط

    بعد از ساختن محیط جدید با دستور زیر محیط را فعال کنید.

    ```
    conda activate gisenv
    ```

۴. نصب RSGISLib 

    بعد از فعال‌سازی محیط با دستور زیر کتابخانه rsgislib را در محیطی که ساختید نصب کنید.

    ```
    conda install conda-forge::rsgislib
    ```

    !!! نکته

        برای استفاده از این محیط در Jupyter Notebook نرم‌افزار VSCode باید ipykernel را با دستور زیر داخل این محیط نصب کنید.

        ```
        conda install ipykernel --update-deps --force-reinstall
        ```

۶. بعد از انجام مراحل فوق می‌توانید kernel فایل Jupyter خود را در نرم‌افزار VSCode به محیط ساخته شده تغییر دهید.

۷. برای اطمینان از درستی نصب کتابخانه دستور زیر را در یکی از سلول‌های Jupyter وارد و اجرا کنید.

    ```python
    import rsgislib
    ```


!!! نکته

    داخل محیط conda می‌توانید از دستور pip برای نصب کتابخانه‌های مدنظر استفاده کنید.


### ۲. دستورهای پرکاربرد
کتابخانه RSGISLib دارای دستورهای متنوعی برای تحلیل لایه‌های رستری، به خصوص در زمینه سنجش از راه دور، است.

برخی از پرکاربردترین دستورهای این کتابخانه عبارتند از:

۱. Raster Calculator

    ```python
    from rsgislib import imagecalc
    imagecalc.image_band_math('output.tif', 'raster1 - raster2', 'KEA')
    ```

۲. Raster Reclassification:

    ```python
    from rsgislib.rastergis import ratutils

    reclasses = {1: 10, 2: 20, 3: 30}
    ratutils.reclassifyRaster('input.tif', 'reclassed.tif', reclasses)
    ```

۳. Zonal Statistics:

    ```python
    from rsgislib.stats import zonalstats

    zonalstats.calculateZonalStatistics('input.tif', 'zones..gpkg', 'zonal_stats.csv')
    ```

۴. Surface Analysis

    ```python
    from rsgislib import rastergis
    slope = rastergis.terrain.calculate_slope(dem)
    aspect = rastergis.terrain.calculate_aspect(dem)
    ```

۵. Image Segmentation:

    ```python
    from rsgislib.segmentation import segutils

    segutils.runShepherdSegmentation('input.tif', 'segment..gpkg', num_clusters=100)
    ```

۶. Image Classification

    ```python
    from rsgislib.classification import classutils

    classutils.performRFClassification('output_segments.gpkg', 'output_classes.gpkg', 'training_data.shp')
    ```

### تمرین 
پروژه بالا را بدون استفاده از Numpy  و به کمک RSGISLib انجام دهید.

