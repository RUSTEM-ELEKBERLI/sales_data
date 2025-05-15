# sales_data 
import pandas as pd
df = pd.read_csv("satislar_data.csv")

discount_impact = df.groupby('discount_pct')['quantity'].sum()

weekday_sales = df.groupby('weekday')['total_amount'].sum()

category_quantity = df.groupby('product_category')['quantity'].sum().sort_values(ascending=False)
category_revenue = df.groupby('product_category')['total_amount'].sum().sort_values(ascending=False)

vip_analysis = df.groupby('is_vip_customer').agg({
    'total_amount': ['sum', 'mean'],
    'sale_id': 'count',
    'customer_id': 'nunique'
})

if 'is_returned' in df.columns:
    # Ümumi qaytarma dərəcəsi
    return_rate = df['is_returned'].mean() * 100

    
    return_reasons = df[df['is_returned'] == True]['return_reason'].value_counts()

   
    product_return_rates = df.groupby('product_name')['is_returned'].mean().sort_values(ascending=False) * 100



import pandas as pd
import numpy as np


cat_discount_analysis = df.pivot_table(
    index='product_category',
    columns='discount_pct',
    values='quantity',
    aggfunc='sum',
    fill_value=0
)


import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns


plt.style.use('seaborn-v0_8-whitegrid') 
sns.set(font_scale=1.2)


df = pd.read_csv("satislar_data.csv")


df['date'] = pd.to_datetime(df['date'])


import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
import matplotlib.dates as mdates

# Qrafiklərin daha gözəl görünməsi üçün
plt.style.use('seaborn-v0_8-whitegrid')
sns.set(font_scale=1.2)
plt.rcParams['figure.figsize'] = (12, 7)

# Faylı oxuyaq
df = pd.read_csv("satislar_data.csv")

# Tarix sütununu düzgün formata çevirək
df['date'] = pd.to_datetime(df['date'])
if 'return_date' in df.columns:
    df['return_date'] = pd.to_datetime(df['return_date'])

# 1. Endirim Analizi - Bar Qrafiki
discount_impact = df.groupby('discount_pct')['quantity'].sum()

plt.figure()
discount_impact.plot(kind='bar', color='skyblue')
plt.title('Endirim Səviyyələrinə görə Satılan Məhsul Sayı', fontsize=14)
plt.xlabel('Endirim Dərəcəsi')
plt.ylabel('Satılan Məhsul Sayı')
plt.xticks(rotation=0)
plt.grid(axis='y', alpha=0.3)
for i, v in enumerate(discount_impact):
    plt.text(i, v + 50, f"{v}", ha='center', fontweight='bold')
plt.tight_layout()
plt.savefig('discount_bar.png')
plt.close()

# 2. Aylıq Satış Trendi - Xətt Qrafiki
monthly_sales = df.groupby(pd.Grouper(key='date', freq='ME'))['total_amount'].sum().reset_index()

plt.figure()
plt.plot(monthly_sales['date'], monthly_sales['total_amount'], marker='o', linestyle='-', linewidth=2, color='#2E86C1')
plt.title('Aylıq Satış Trendi', fontsize=14)
plt.xlabel('Tarix')
plt.ylabel('Ümumi Satış Məbləği')
plt.grid(True, alpha=0.3)
plt.gca().xaxis.set_major_formatter(mdates.DateFormatter('%b %Y'))
plt.gca().xaxis.set_major_locator(mdates.MonthLocator(interval=1))
plt.xticks(rotation=45)
plt.tight_layout()
plt.savefig('monthly_sales_line.png')
plt.close()

# 3. Məhsul Kateqoriyaları - Dairə Qrafiki
category_sales = df.groupby('product_category')['total_amount'].sum().sort_values(ascending=False)
top_categories = category_sales.head(5)
other_categories = pd.Series({'Digər': category_sales[5:].sum()})
pie_data = pd.concat([top_categories, other_categories])

plt.figure(figsize=(10, 8))
plt.pie(pie_data, labels=pie_data.index, autopct='%1.1f%%',
        startangle=90, shadow=False,
        colors=['#3498DB', '#E74C3C', '#2ECC71', '#F39C12', '#9B59B6', '#95A5A6'])
plt.title('Məhsul Kateqoriyalarının Ümumi Satışda Payı', fontsize=14)
plt.axis('equal')
plt.tight_layout()
plt.savefig('category_pie.png')
plt.close()

# 4. Həftənin Günü və Saat üzrə Satışlar - İstilik Xəritəsi
weekday_order = ['Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday', 'Sunday']
hour_day_sales = df.pivot_table(index='weekday', columns='hour',
                               values='total_amount', aggfunc='sum')
hour_day_sales = hour_day_sales.reindex(weekday_order)

plt.figure(figsize=(14, 8))
sns.heatmap(hour_day_sales, cmap='YlGnBu', annot=False, linewidths=.5)
plt.title('Həftənin Günləri və Saatlara görə Satışlar', fontsize=14)
plt.xlabel('Saat')
plt.ylabel('Gün')
plt.tight_layout()
plt.savefig('hour_day_heatmap.png')
plt.close()

# 5. VIP və Regular Müştərilər - Müqayisəli Bar
vip_analysis = df.groupby('is_vip_customer').agg({
    'total_amount': ['mean', 'sum'],
    'sale_id': 'count'
})
vip_analysis.columns = ['Orta Çek', 'Ümumi Gəlir', 'Satış Sayı']
vip_analysis = vip_analysis.rename(index={False: 'Adi Müştəri', True: 'VIP Müştəri'})

plt.figure(figsize=(10, 6))
vip_analysis['Orta Çek'].plot(kind='bar', color=['blue', 'gold'])
plt.title('VIP və Adi Müştərilərin Orta Çeki', fontsize=14)
plt.xlabel('Müştəri Tipi')
plt.ylabel('Orta Çek Məbləği')
plt.grid(axis='y', alpha=0.3)
for i, v in enumerate(vip_analysis['Orta Çek']):
    plt.text(i, v + 5, f"{v:.2f}", ha='center')
plt.tight_layout()
plt.savefig('vip_regular_bar.png')
plt.close()

# 6. Ödəniş Metodları - Dairə Qrafiki
payment_methods = df.groupby('payment_method')['total_amount'].sum()

plt.figure(figsize=(10, 8))
plt.pie(payment_methods, labels=payment_methods.index, autopct='%1.1f%%',
        startangle=90, explode=[0.05]*len(payment_methods),
        colors=sns.color_palette('pastel'))
plt.title('Ödəniş Metodlarının Payı', fontsize=14)
plt.axis('equal')
plt.tight_layout()
plt.savefig('payment_pie.png')
plt.close()

# 7. Şəhərlərə görə Satışlar - Horizontal Bar Chart
city_sales = df.groupby('customer_city')['total_amount'].sum().sort_values(ascending=False).head(10)

plt.figure(figsize=(12, 8))
city_sales.plot(kind='barh', color=sns.color_palette('viridis', len(city_sales)))
plt.title('Top 10 Şəhər üzrə Satışlar', fontsize=14)
plt.xlabel('Ümumi Satış Məbləği')
plt.ylabel('Şəhər')
plt.grid(axis='x', alpha=0.3)
for i, v in enumerate(city_sales):
    plt.text(v + 100, i, f"{v:.0f}", va='center')
plt.tight_layout()
plt.savefig('city_sales_bar.png')
plt.close()

# 8. Kombinə edilmiş vizuallaşdırma - Endirim analizinin daha dərin versiyası
discount_quantity = df.groupby('discount_pct')['quantity'].sum()
discount_revenue = df.groupby('discount_pct')['total_amount'].sum()
discount_avg_check = df.groupby('discount_pct')['total_amount'].mean()

fig, (ax1, ax2) = plt.subplots(2, 1, figsize=(12, 12), sharex=True)

# Yuxarıdakı qrafik - Satış Həcmi
bars1 = ax1.bar(discount_quantity.index, discount_quantity.values, color='skyblue', alpha=0.7)
ax1.set_ylabel('Satış Miqdarı')
ax1.set_title('Endirim Səviyyələrinin Satış Həcminə Təsiri', fontsize=14)
ax1.grid(axis='y', alpha=0.3)
for bar in bars1:
    height = bar.get_height()
    ax1.annotate(f'{height}',
                xy=(bar.get_x() + bar.get_width() / 2, height),
                xytext=(0, 3), textcoords="offset points",
                ha='center', va='bottom')

# Aşağıdakı qrafik - Orta Çek
bars2 = ax2.bar(discount_avg_check.index, discount_avg_check.values, color='salmon', alpha=0.7)
ax2.set_xlabel('Endirim Dərəcəsi')
ax2.set_ylabel('Orta Çek Məbləği')
ax2.set_title('Endirim Səviyyələrinin Orta Çekə Təsiri', fontsize=14)
ax2.grid(axis='y', alpha=0.3)
for bar in bars2:
    height = bar.get_height()
    ax2.annotate(f'{height:.2f}',
                xy=(bar.get_x() + bar.get_width() / 2, height),
                xytext=(0, 3), textcoords="offset points",
                ha='center', va='bottom')

plt.tight_layout()
plt.savefig('discount_combined.png')
plt.close()

print("Bütün qrafiklər yaradıldı və saxlanıldı!")


