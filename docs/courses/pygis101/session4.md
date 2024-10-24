---
title: جلسه ۴ - آمار
level: secret
---
## گزارش‌های آماری ساده
### ۱. با استفاده از describe
دستور describe از کتابخانه pandas اطلاعات آماری کلی از محتویات ستون‌های جدول ارائه می‌دهد.

```python
import pandas as pd

df = pd.read_csv('aqi.csv')

df['pm25'].describe()
```

همچنین می‌توانیم با فیلتر کرد پایگاه داده اطلاعات آماری مرتبط با بازه‌ای خاص را دریافت کنیم.

```python
filtered_df = df[df['date_objects'] > pd.Timestamp('2020-01-01')]

filtered_df['pm25'].describe()
```

!!! نکته
    برای زمان‌مند کردن پایگاه داده باید فیلد نوشتاری که در آن تاریخ به صورت string وارد شده است را به فیلدی از زمان تغییر دهیم. برای این کار می‌توانیم از دستور to_datetime از کتابخانه pandas استفاده کنیم.

    ```python
    df['date_objects'] = pd.to_datetime(df['date'], format="%m/%d/%Y")
    ```

    برای آشنایی بیشتر با نحوه تعریف فرمت تاریخ می‌توانید به [این لینک](https://strftime.org/) مراجعه کنید.    

### ۲. خلاصه‌کردن جدول
با استفاده از دستور groupby می‌توانید جدول مورد نظر را بر اساس محتویات یک ستون خلاصه کرد.

```python
aq_df = df.groupby(['Name','X', 'Y']).agg({'pm25':'mean','pm10':'mean','co':'mean','o3':'mean'}).reset_index()

```

### ۳. ترسیم نمودارهای آماری ساده
از دستورهای زیر می‌توان برای ترسیم نمودارهای ساده آماری استفاده کرد.


۱. نمودار ستونی

    ```python
    plt.hist(aq_df['pm25'],bins=8)
    ```

۲. نمودار پراکندگی

    ```python
    plt.scatter(aq_df['X'],aq_df['Y'],alpha=0.5,c ='red')
    ```

## تحلیل آماری پیشرفته
!!! نکته

    ردیف‌های حاوی محتوای null برای تحلیل‌های آماری پیشرفته مشکل ایجاد می‌کند. بنابراین بهتر است در ابتدا با استفاده از دستور dropna از کتابخانه pandas این ردیف‌ها را از تحلیل خارج کنیم.

    ```python
    df = df.dropna()
    ```
   
### ۱. همبستگی

۱. با استفاده از pandas

    ```python
    aq_df.drop('Name', axis=1).corr()
    ```

۲. با استفاده از numpy

    ```python
    import numpy as np

    np.corrcoef(aq_df['pm25'],aq_df['o3'])
    ```

۳. با استفاده از scipy

    ```python
    from scipy import stats as spt

    spt.pearsonr(aq_df['pm25'],aq_df['pm10'])
    ```

### ۲. خوشه‌بندی
خوشه‌بندی از تحلیل‌های آماری رایج در تحلیل داده‌های انبوه است و از آن برای دسته‌بندی یک مجموعه بر اساس ویژگی‌های مشابه آن‌ها استفاده می‌شود. برای این‌کار در پایتون اغلب از کتابخانه sklearn استفاده می‌شود.

```python
from sklearn.cluster import KMeans

X=df[['pm10','pm25','co','o3']]
# creating the cluster object
k_means = KMeans(n_clusters=3)
k_means.fit(X)

df['cluster'] = k_means.labels_

```
### ۳. رگرسیون
تحلیل رگرسیون از تحلیل‌های آماری رایج است که در پایتون برای انجام آن اغلب از کتابخانه statsmodel استفاده می‌شود.

```python
import statsmodels.api as sm

independent_Var = df['Y']
dependent_var   = df['pm25']

independent_Var = sm.add_constant(independent_Var)

linearModel = sm.OLS(dependent_var, independent_Var)
results = linearModel.fit()
print(results.summary())
```

### ۴. ترسیم نمودارهای آماری پیشرفته
برای ترسیم نمودارهای آماری پیشرفته اغلب از دستورهای کتابخانه seaborn استفاده می‌شود.

۱. نمودار پراکندگی

    ```python
    import seaborn as sns

    sns.scatterplot(data=df, x='o3', y="pm25")
    
    sns.displot(data=df, x='o3', y="pm25")
    ```

۲. نمودار حرارتی

    ```python

    aq_corr = aq_df.drop('Name', axis=1).corr()

    sns.heatmap((aq_corr))
    ```

۳. نموداب رگرسیون

    ```python

    sns.regplot(x=aq_df['Y'], y=aq_df['pm10'], line_kws={"color":"green","alpha":0.6,"lw":4})
    ```

۵. نمودار ترکیبی

    ```python

    sns.jointplot(data=aq_df, x='pm10', y='pm25', kind="reg")
    ```

## تحلیل‌های مکان‌آماری
یکی از مدل‌های رایج در تحلیل‌های مکان‌آماری مدل Moran's I است. این مدل در واقع قرارگیری فیزیکی اعضای مجموعه با ویژگی‌های مشابه را تحلیل می‌کند. برای انجام این کار از روش همبستگی فیزیکی استفاده می‌کند. برای انجام این تحلیل در پایتون اغلب از کتابخانه pysal استفاده می‌شود.

### ۱. محاسبه Global Moran's I
این شاخص برای تحلیل کلی مجموعه عوارض جغرافیایی از نظر تشکیل cluster های فضایی استفاده می‌شود. برای محاسبه خروجی این تحلیل عددی بین صفر و یک است. هرچقدر عدد به صفر نزدیک‌تر باشد به این معنا است که کلاسترهای معناداری بر اساس ویژگی مورد تحلیل در مجموعه داده‌های مکانی شکل نگرفته است.

```python
from pysal.lib import weights
from pysal.explore import esda
import geopandas as gpd

gdf=gpd.GeoDataFrame(df,geometry=gpd.points_from_xy(df.X, df.Y),crs='epsg:32639')

w=weights.Queen.from_dataframe(gdf)
moran=esda.Moran(gdf['pm25'],w)
print(moran.I)
```

### ۲. محاسبه Local Moran's I
این شاخص به دنبال نقاط داغ و نقاط سرد داده‌های مکانی بر اساس بالا یا پائینی بودن یک ویژگی توصیفی است. به بیان دیگر به دنبال خوشه‌های فضایی بر اساس قرارگیری فیزیکی عوارض با ویژگی مشترک است.

```python
mapclust=esda.Moran_Local(gdf['pm25'],w)
gdf['local_moran']=mapclust.q

gdf.plot(column='local_moran')
```
