---
title: جلسه ۵ - وب
level: secret
---
## ارتباط با API
### ۱. ساخت نقاط تصادفی

۱. گرفتن محدوده مورد نظر 

    برای دریافت محدوده مورد نظر از لایه مکانی موجود می‌توان از دستور total_bounds از کتابخانه geopandas استفاده کرد.

    ```python
    minx, miny, maxx, maxy = roi.total_bounds 
    ```

۲. تعریف تابع ساخت نقطه تصادفی در محدوده تعیین شده

    با استفاده از دستور random از کتابخانه numpy می‌توان طول و عرض جغرافیایی رندومی را بین حداقل و حداکثر طول عرض جغرافیایی مدنظر ساخت.

    ```python
    def random_points_in_bound(minX, minY, maxX, maxY):
        x = np.random.uniform(minX, minY)
        y = np.random.uniform(minY, maxY)
        return Point(x,y)
    ```
۳. تعرفی متغیرها

    متعیرهایی برای تعریف تعداد نقاط مدنظر و ذخیره نقاط در قالب لیست می‌سازیم.

    ```python
    numPoints = 100
    randomPoints = []
    ```
۴. ساخت نقاط تصادفی

    با استفاده از تابع تعریف شده در مرحله قبل و به کمک دستور while عمل ساخت نقاط تصادفی را تا رسیدن به تعداد تعریف شده ادامه می‌دهیم.

    ```python
    while len(randomPoints) < numPoints:
        point = random_points_in_bound(minx,miny, maxx, maxy)
        if roi.contains(point).any():
            randomPoints.append(point)
    ```

    !!! نکته

        ۱. دستور len تعداد اعضای مجوعه را باز می‌گرداند.

        ۲. از آنجا که total_bounds مستطیل در برگیرنده لایه را باز می‌گرداند،  با استفاده از دستور contains از قرار گرفتن نقطه در محدوده مورد نظر اطمینان کسب می‌کنیم.

۴. تعریف GeoDataFrame

    نقاط ساخته شده در مرحله قبل را با استفاده از دستور GeoDataFrame به لایه جغرافیایی تبدیل می‌کنیم.

    ```python
    gdf = gpd.GeoDataFrame(geometry=randomPoints, crs=roi.crs)
    ```

۵. ذخیره طول و عرض جغرافیایی در دو ستون جداگانه

    ```python
    points_gdf['x'] = points_gdf.geometry.x
    points_gdf['y'] = points_gdf.geometry.y
    ```

### ۲. فرستادن درخواست به API
در تمرین این جلسه از API وب‌سایت [Open-Meteo](https://open-meteo.com/) استفاده می‌کنیم. در وب‌سایت‌هایی که سرویس API ارائه می‌کنند، معمولاً بخشی به نام Documatation یا Developer وجود دارد که در آن راهنمای ارسال درخواست و قراردادهای تعریف شده برای فرستان درخواست و استفاده از پاسخ آمده است. 

درخواست API معمولاً از طریق یک آدرس URL ارسال می‌شود که از یک بخش اصلی (Base URL) و مشخصات درخواست که به عنوان پارامتر بعد از علامت سؤال (?) در آدرس می‌آید تشکیل شده است. 

```
https://api.open-meteo.com/v1/forecast?latitude=52.52&longitude=13.41&hourly=temperature_2m
```

برای نمونه در درخواست بالا Base URL و پارامترها عبارتند از:

```
# Base URL: https://api.open-meteo.com/v1/forecast
# Parameters:
# - latitude=52.52
# - longitude=13.41
# - hourly=temperature_2m
```

۱. استفاده از کتابخانه requests از کتابخانه‌های درونی Python

    از آنجا که کتابخانه requests از کتابخانه‌های درونی Python است، نیازی به نصب آن نیست.

    ```python
    import requests
    ```

    !!! نکته
        
        از آنجا که در APIها معمولا طول و عرض جغرافیایی با واحد اندازه‌گیری درجه، و نه متر، و در سیستم مختصاتی WGIS 1984 وارد می‌شود. قبل از ارسال درخواست باید سیستم مختصات لایه مکانی مدنظر را تغیر دهیم.

        ```python
        gdf = gdf.to_crs(epsg=4326)
        ```

۲. تعریف یک حلقه تکرار درخواست

    از آنجا که در این تمرین، برای هر نقطه تصادفی یک درخواست به سایت Open-Meteo ارسال می‌شود تا دمای هوا و ارتفاع از سطح دریا گرفته شود، از حلقه for برای اینکار استفاده می‌کنیم.

    ```python
    for idx, point in gdf.iterrows():
    ```
۳. تعریف پارامترها

    پارامترهای ضروری برای تعریف درخواست را به صورت یک متغیر از جنس دیکشنری تعریف می‌کنیم.

    ```python
        params = {
            'latitude': point.geometry.y,
            'longitude': point.geometry.x,
            'current': 'temperature_2m',
            'forecast_days': 1
        }    
    ```

۴. فرستادن درخواست و ذخیره جواب

    بعد از تعریف پارامترها، درخواست را بر اساس دستورالعمل API ارسال کرده و جواب را به صورت یک متغیر ذخیره می‌کنیمو.

    ```python
        response = requests.get(BASE_URL, params=params)
    ```

۵. چک کردن وضعیت جواب

    در جواب بازگردانده شده معمولاً پارامتری به نام status_code وجود دارد که موفقیت یا عدم موفقیت در ارسال جواب را به صورت کد عددی باز می‌گرداند. معمولاً در صورت موفقیت کد 200 بازگردانده می‌شود. در صورت عدم موفقیت کدهای دیگر باز گردانده می‌شود که در صفحه راهنمای API کدها توضیح داده شده است.  از این رو قبل از خواندن و استفاده از محتوای جواب چک می‌کنیم که عملیات ارسال درخواست و دریافت جواب درست انجام شده باشد.

    ```python
        if response.status_code == 200:
    ```

۶. خواندن جواب

    جواب API به صورت یک بسته داده در قالب json بازگردانده می‌شود. از این رو، با استفاده از دستور json جواب را در قالب یک متغیر ذخیره و از آن برای ذخیره در فیلد جدید در لایه مکانی ساخته شده استفاده می‌کنیم.

    ```python
            jsonData = response.json()
            gdf.loc[idx, 'temperature'] = jsonData['current']['temperature_2m']
            gdf.loc[idx, 'elevation'] = jsonData['elevation']
    ```
۷. کد نهایی

    درنهایت کد python برای انجام این تمرین به این شکل خواهد بود:

    ```python
    import requests

    BASE_URL = 'https://api.open-meteo.com/v1/forecast'

    gdf = gdf.to_crs(epsg=4326)

    for idx, point in gdf.iterrows():
        params = {
            'latitude': point.geometry.y,
            'longitude': point.geometry.x,
            'current': 'temperature_2m',
            'forecast_days': 1
        }    
        response = requests.get(BASE_URL, params=params)
        if response.status_code == 200:
            jsonData = response.json()
            gdf.loc[idx, 'temperature'] = jsonData['current']['temperature_2m']
            gdf.loc[idx, 'elevation'] = jsonData['elevation']
    ```

### ۳. استفاده از پاسخ دریافتی
بعد از دریافت پاسخ و ذخیره آن در ستون‌های جدید در جدول لایه مکانی، اکنون می‌توانیم از آن در تحلیل‌های مدنظر استفاده کنیم.


۱. همبستگی بین دما و ارتفاع از سطح دریا

    ```python
    corr = gdf.drop('geometry', axis=1).corr()
    sns.heatmap((corr), annot=True)
    ```

۲. ترسیم نمودار رگرسیون

    ```python
    sns.regplot(data=gdf, x='elevation', y='temperature', line_kws={"color":"green","alpha":0.6,"lw":4})
    plt.title('Elevation vs. Temperature in Tehran')
    plt.xlabel('Elevation (m)')
    plt.ylabel('Temperature (°C)')
    plt.show()
    ```

## گرفتن خروجی وب
یکی از رایج‌ترین کتابخانه‌های پایتون برای گرفتن خروجی وب کتابخانه folium است. این کتابخانه از کتابخانه جاوااسکریپت leaflet برای ساختن خروجی وب استفاده می‌کند. بنابراین دستورهای قابل استفاده در leaflet در اینجا به زبان پایتون در آمده است.

قبل از استفاده از این کتابخانه باید آن را در محیط مجازی پروژه نصب و در کد خود وارد کنیم.

```
! pip install foluim
```

```python
import folium
```
   
### ۱. ساخت نقشه خالی
در ابتدا باید نقشه خالی را به کمک دستور زیر ساخته و در قالب یک متغیر ذخیره کنیم.

```python
map = folium.Map(location=[35.6892, 51.3890], zoom_start=10)
```

!!! نکته

    ۱. در پارامتر location عرض و طول جغرافیایی [latitude, longitude] مرکز نقشه مشخص می‌شود.

    ۲. در پارامتر zoom_start مشخص می‌شود که نقشه در دید اولیه در چه درجه زومی نمایش داده شود. عدد ۱ کل زمین را نمایش داده و هر چقدر عدد بزرگتر می‌شود نقشه با مقیاس کوچکتر نمایش داده می‌شود.

### ۲. اضافه کردن نقشه پایه

۱. استفاده از نقشه‌های پایه درونی folium

    به صورت پیش‌فرض folium از نقشه OpenStreetMap به عنوان نقشه پایه استفاده می‌کند. علاوه بر این نقشه، این کتابخانه ۵ نقشه پایه دیگر را می‌شناسد که می‌توانیم با استفاده پارامتر tiles در دستور Map آن را تغیر دهیم.

    ```python
    # Tiles:
    # "OpenStreetMap" (default)
    # "Stamen Terrain"
    # "Stamen Toner"
    # "Stamen Watercolor"
    # "CartoDB positron"
    # "CartoDB dark_matter"

    m = folium.Map(location=[35.6892, 51.3890], zoom_start=1, tiles='Stamen Terrain')
    ```

۲. استفاده از نقشه پایه شخصی

    علاوه بر این شش نقشه پایه، می‌توانیم نقشه پایه دلخواه خود را نیز در folium تغریف کنیم. برای این کار باید آدرس نقشه پایه را در پارامتر tiles وارد کنیم. طیف گسترده‌ای از نقشه‌های موجود در وب را می‌توانید در این [آدرس](https://leaflet-extras.github.io/leaflet-providers/preview/) مشاهده و از بین آن‌ها انتخاب کنید. مجموعه منتخب از آن‌ها در راهنمای [نقشه‌های پایه](/hacks/basemaps/) موجود است.

    ```python
    m = folium.Map(location=[35.6892, 51.3890], zoom_start=11, tiles='https://mt1.google.com/vt/lyrs=s&x={x}&y={y}&z={z}', attr='Google')
    ```

    !!! نکته

        در صورتی که خودتان نقشه پایه را از این طریق تعریف کردید، اضافه کردن پارامتر attr الزامی است. در این پارامتر منبع نقشه استفاده شده وارد می‌شود.

۳. استفاده از چند نقشه پایه

    با استفاده از دستور TileLayer می‌توانیم چند نقشه پایه برای انتخاب داشته باشیم.

    ```python
    folium.TileLayer("https://mt1.google.com/vt/lyrs=m&x={x}&y={y}&z={z}", attr="Google", name='Google Map').add_to(m)
    folium.TileLayer("https://mt1.google.com/vt/lyrs=s&x={x}&y={y}&z={z}", attr="Google", name="Google Satellite").add_to(m)
    ```

### ۳. اضافه کردن لایه‌های مکانی
با استفاده از دستور add_to می‌توانید لایه‌های مکانی مدنظر را به نقشه خام ساخته شده اضافه کنید. در این مثال از اطلاعات زمین‌لرزه‌ها گرفته شده در جلسه دوم استفاده می‌کنیم.

۱. ساخت لایه خالی

    برای لایه‌های وکتوری از دستور FeatureGroup برای ساخت لایه خالی استفاده کرده و آن را به عنوان یک متغیر ذخیره و به نقشه خام اضافه می‌کنیم.

    ```python
    fg = folium.FeatureGroup(name="Earthquakes").add_to(m)
    ```

۲. ذخیره عوارض لایه مکانی در لایه ساخته شده در مرحله قبل

    ```python
    df = pd.read_csv('query.csv')
    gdf = gpd.GeoDataFrame(df, geometry=gpd.points_from_xy(df.longitude, df.latitude), crs='epsg:4326')

    for idx, point in gdf.iterrows():
        popup=(f"<strong>Location:</strong> {row['place']}<br>"
                f"<strong>Time:</strong> {row['time']}<br>"
                f"<strong>Magnitude:</strong> {row['mag']}<br>"
                f"<strong>Depth:</strong> {row['depth']} km"),
        color = 'red' if point['mag'] > 5 else 'blue'  
        folium.Marker(
            location=[point.geometry.y, point.geometry.x],
            popup=popup_info,
            icon=folium.Icon(color=color)
        ).add_to(fg)

    ```

    !!! نکته

        ۱. در این مثال از دستور Marker برای ترسیم پین در نقاط ساخته شده در مراحل قبلی استفاده شده است.

        ۲. در دستور Marker پارامترهای مختلفی درباره نحوه نمایش پین‌ها می‌توان کرد. در این مثال دو در متغیر popup و color رنگ و محتوای نمایش داده شده بعد کلیک نقطه را تعریف شده است.

۳. با استفاده از دستور LayerControl به نقشه ساخته شده راهنما اضافه می‌کنیم.

    ```python
    folium.LayerControl().add_to(m) 
    ```

### ۴. گرفتن خروجی
نقشه ساخته شده را می‌توانید با استفاده از دستور save در قالب فایل html ذخیره کنید.

```python
map.save('eq.html')
```

### ۵. استفاده از پلاگین‌ها
کتابخانه folium مجموعه متنوعی از پلاگین‌ها را دارد که می‌توان با استفاده از آن‌ها نقشه‌های مختلفی ساخت. برخی از رایج‌ترین آن‌ها عبارتند از:

۱. اضافه کردن نقشه راهنما (MiniMap)

    ```python
    from folium.plugins import MiniMap

    minimap = MiniMap(toggle_display=True)
    map.add_child(minimap)
    ```
۲. استفاده از MarkCluster

    هنگامی قصد نمایش نقاط انبوه در نقشه داریم در زوم‌های کوچک‌تر نقاط روی هم می‌افتند. دستور MarkCluster در این موارد با نمایش تعداد و رنگ نقشه‌ها را خواناتر می‌سازد. برای استفاده از این دستور در تعریف لایه وکتوری خام به جای FeatureGroup از MarkCluster استفاده می‌کنیم.

    ```python
    from folium.plugins import MarkerCluster

    fg = MarkerCluster().add_to(m)
    ```

۳. نقشه حرارتی (HeatMap)

    یکی دیگر راه‌های نمایش نقاط انبوه ساخت نقشه حرارتی است.

    ```python
    from folium.plugins import HeatMap

    heat_data = [[row.geometry.y, row.geometry.x] for _, row in gdf.iterrows()]
    HeatMap(heat_data, radius=15).add_to(m)
    ```

۴. تمرین: نقشه زمانی و موقعیت مکانی کاربر (TimestampedGeoJson, LocateControl)

    کد زیر را کالبد شکافی کنید.

    ```python
    from folium.plugins import TimestampedGeoJson, LocateControl

    df = pd.read_csv('query.csv')
    df['time'] = pd.to_datetime(df['time'], format="%Y-%m-%dT%H:%M:%S.%fZ")

    gdf = gpd.GeoDataFrame(df, geometry=gpd.points_from_xy(df.longitude, df.latitude), crs="EPSG:4326")

    features = []
    for _, row in gdf.iterrows():
        color = "red" if row['mag'] >= 3 else "blue"
        feature = {
            "type": "Feature",
            "geometry": {
                "type": "Point",
                "coordinates": [row.geometry.x, row.geometry.y],
            },
            "properties": {
                "time": row['time'].isoformat(),
                "style": {
                    "color": color,
                    "weight": row['mag'] * 0.5, 
                    "fillOpacity": 0.6
                },
                "icon": "circle",
                "iconstyle": {
                    "fillColor": color,
                    "fillOpacity": 0.6,
                    "stroke": "true",
                    "radius": row['mag'] * 2
                }
            }
        }
        features.append(feature)

    geojson_data = {
        "type": "FeatureCollection",
        "features": features
    }

    m = folium.Map(location=[gdf.geometry.y.mean(), gdf.geometry.x.mean()], zoom_start=2)

    TimestampedGeoJson(
        data=geojson_data,
        period="PT6H",
        transition_time=200,
        loop=False,
        auto_play=False,
        add_last_point=True
    ).add_to(m)

    LocateControl().add_to(m)


    ```

