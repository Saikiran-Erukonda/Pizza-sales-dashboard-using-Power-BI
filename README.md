# Pizza-sales-dashboard-using-Power-BI
![Pizza sales dashboard_page-0001](https://github.com/user-attachments/assets/26e59313-31b5-4839-ba4a-69bc4f5173aa)
The working video Link is here https://youtu.be/QhAN13yV-r8?si=WPtd2hkf4nFBIOkz

---

# Pizza-sales-dashboard-using-Google_Colab (Python)

## Project Overview
This project performs **Exploratory Data Analysis (EDA)** on [`Pizza sales dataset`](https://github.com/Saikiran-Erukonda/Pizza-sales-dashboard-using-Power-BI/tree/main/pizza%20sales%20dataset) to uncover trends and patterns in Customer Order preferences. We use libraries like **Pandas, Numpy, Matplotlib, Seaborn**for cleaning, visualization, and analysis. 

  -  [`pizza_sales_descriptive_analysis.ipynb`](https://github.com/Saikiran-Erukonda/Pizza-sales-dashboard-using-Power-BI/blob/main/pizza_sales_descriptive_analysis.ipynb)
---

## Objective
The goal of this project is to:
1. Track the orders received by times **(Morning,Afternoon,Evening,Night)**
2. Identify which **day** of the week we are getting the most orders ?
3. The total no of sales ?
4. which **pizza ordered most** and the **revenue** generated by it ?
5. Top selling category ?
6. Track the sales Month on Month?

---

## Datasets
The `pizzas.csv` dataset contains **96 entries and  4 features** including:
- **pizza_id** : The shortform of pizza name and size.
- **pizza_type_id** : The shortform of pizza name.
- **size** : Different size of pizza **S,M,L,XL,XXL**. 
- **price** : Respective pizza price based on Pizza name and size.

The  `orders.csv` dataset contains **21350 entries and  3 features** including:
- **order_id** :The order_id for order given by a customer
- **date** : The date of order
- **time** : The time of order

The `order_details.csv` dataset contains **48620 entries and 4 features** including: 
- **order_details_id** : The ID for internal items of given `order_id` in `orders.csv` 
- **order_id** : `order_id` from table `orders.csv`
- **pizza_id** : The `pizza_id` matches with '`pizza_id` which is in `pizzas.csv`
- **quantity** : The quantity ordered by customer

The `pizza_types.csv` dataset contains **32 entries and 4 features** including:
- This dataset has 4 features,in that ingredients feature has data which is seperated by commas(,) and all are inbetween two inverted commas("). The loading of dataset needs delimiter
- **pizza_type_id** : This is same as `pizza_type_id` in `pizzas.csv` dataset
- **name**          : Pizza's Full name
- **category**      : Reflects the category like **Classic,Supreme,Veggie,Chicken** 
- **ingredients**   : The ingredients were used in making the pizza.
---

## Steps and Workflow

### 1. Mounting google Drive into Google Colab
```python
from google.colab import drive
drive.mount('/content/drive/')
#change working directory
import os
os.chdir('/content/drive/MyDrive/pizza_sales_analysis_py/')
print(os.getcwd())
```
### 2. Datasets Loading
```python
import pandas as pd
pizzas  = pd.read_csv("./pizzas.csv")
order_details = pd.read_csv("./order_details.csv")
orders = pd.read_csv("./orders.csv")
# pizza_types = pd.read_csv("./pizza_types.csv")

#printing the table characteristics
print(pizzas.head())
print(pizzas.info())

print(orders.head())
print(orders.info())

print(order_details.head())
print(order_details.info())

# Change ingredients as shown "ingredients" in csv file and Load the CSV file; ingredients are separated with commas so loaded using delimiter
file_path = "./pizza_types.csv"
pizza_types = pd.read_csv(file_path, delimiter=',', quotechar='"')
#displaying the dataset
print(pizza_types.head())
print(pizza_types.info())
```
### 3. Data Cleaning
- **Handle missing data**: Clear missing values nulls from the data or replace them
- **Fix data types**: In `orders.csv` Converted `date` to a **datetime** object.
  ```python
  #converting date column to datetime object
  orders['date'] = pd.to_datetime(orders['date'])
  ```
- **Remove outliers**: Like ingredients in `Pizza_type`
  ```python
  pizza_types = pizza_types.drop('ingredients',axis = 1)
  ```
### 4. EDA (Exploratory Data Analysis) by finding Objectives
1. Top 5 selling  pizzas
```python
# Group by 'pizza_id' and sum the 'quantity' for each pizza
pizza_sales = order_details.groupby('pizza_id')['quantity'].sum().reset_index()

# Rename columns for clarity
pizza_sales.columns = ['pizza_id', 'total_quantity_sold']
pizza_sales = pizza_sales.sort_values(by='total_quantity_sold', ascending=False)
# Display the pizza sales data
print("The top 5 selling pizzas: \n",pizza_sales.head(5))
```
---
- **The top 5 selling pizzas:** 
-  **pizza_id          **|** total_quantity_sold**
-   big_meat_s         **|**        1914
-   thai_ckn_l         **|**        1410
-   five_cheese_l       **|**          1409
-   four_cheese_l        **|**         1316
-   classic_dlx_m         **|**        1181

2. Least  5 selling pizzas
```python
#accesing  lowest 20 rows in pizza sales
lowest_20 = (pizza_sales.tail(20)).sort_values(by='total_quantity_sold', ascending=True)
                                   #taken lowest 20 and sorted in ascending order wrt total_quantity sold
print("The least selling pizzas: \n", lowest_20.head(5))
```
- **The least selling pizzas:** 
  -   **pizza_id   **|** total_quantity_sold**
  -   the_greek_xxl **|**                  28
  -  green_garden_l **|**                  95
  -  ckn_alfredo_s   **|**                96
  -    calabrese_s    **|**               99
  - mexicana_s         **|**         162
1. Track the orders received by times **(Morning,Afternoon,Evening,Night)**
```python
"""Track the orders by time"""
print(orders.head(10))

import pandas as pd
# Define a function to categorize time into morning, afternoon, and evening
def categorize_time(row):
    hour = int(row.split(':')[0])  # Extract the hour part from the time column
    if 5 <= hour < 12:
        return 'Morning'
    elif 12 <= hour < 16:
        return 'Afternoon'
    elif 16 <= hour < 20:
        return 'Evening'
    else:
        return 'Night'

# Apply the function to create a new column 'time_period'
orders['time_period'] = orders['time'].apply(categorize_time)

# Ensure that the 'time_period' follows the desired order
time_period_order = ['Morning','Afternoon','Evening','Night']
orders['time_period'] = pd.Categorical(orders['time_period'], categories=time_period_order, ordered=True)

# Group by 'time_period' and count the number of orders for each period
orders_by_time_period = orders.groupby('time_period')['order_id'].count().reset_index()

# Rename columns for clarity
orders_by_time_period.columns = ['time_period', 'number_of_orders']

# Display the results
print(orders_by_time_period)
```
---
-   **time_period  number_of_orders**
- 0     **Morning**              1240
- 1   **Afternoon**              7915
- 2     **Evening**              8664
- 3       **Night**              3531
---

2.  Identify which **day** of the week we are getting the most orders ?
```python
#Sales by Day of the week
#converting date column to datetime object
orders['date'] = pd.to_datetime(orders['date'])

# Extract the day of the week (0=Monday, 6=Sunday)
orders['day_of_week']=orders['date'].dt.day_name()

# Group by 'day_of_week' and count the number of orders for each day
orders_by_day = orders.groupby('day_of_week')['order_id'].count().reset_index()


# Rename columns for clarity
orders_by_day.columns=['day_of_week','number_of_orders']

# To ensure the days of the week are in order (Monday to Sunday)
days_order = ['Monday','Tuesday','Wednesday','Thursday','Friday','Saturday','Sunday']
orders_by_day['day_of_week'] = pd.Categorical(orders_by_day['day_of_week'],categories=days_order,ordered=True)

# Sort the DataFrame by the correct day order
orders_by_day = orders_by_day.sort_values('day_of_week')

#display the result
print(orders_by_day)
```
---
- **day_of_week   ,  number_of_orders**
- **Monday**        ,      2794
- **Tuesday**      ,      2973
- **Wednesday**     ,      3024
- **Thursday**      ,      3239
- **Friday**        ,      3538
- **Saturday**      ,      3158
- **Sunday**        ,      2624
---
    
3. The total number of sales ?
```python
# 3. total sales
total_sales = order_details["quantity"].sum()
print("The total pizza sales : ",total_sales)
```
- The total pizza **sales** :  49574
- 
4. which **pizza ordered most** and the **revenue** generated by it ?
```python
#merging the pizza sales and pizzas table to know the price for each pizza
sales_revenue = pd.merge(pizza_sales,pizzas,left_on = "pizza_id",right_on = "pizza_id",how ="inner")

# calculatIng revenue generated
sales_revenue["Revenue"] = sales_revenue["total_quantity_sold"] * sales_revenue["price"]
# print(sales_revenue.head(3))

#slicing the first row , 5th column element
print("The most ordered pizza : ",sales_revenue.iloc[0,0])
print("The total revenue generated by most sold pizza: ",sales_revenue.iloc[0,5],"$")
```
---
 - The **most ordered pizza** :  big_meat_s
 - The total **revenue generated** by most sold pizza:  22968.0 $
---
5. Top selling category ?
```python
# print(order_details.head())
# print(pizzas.head())
# print(pizza_types.head())
# print(sales_revenue.head())

#first merging order_details and pizzas datasets
join1 = pd.merge(order_details,pizzas,left_on = "pizza_id",right_on = "pizza_id",how = 'inner')

#removing the columns price,pizza_id, Order_id, size
join1 = join1.drop('price',axis = 1)
join1 = join1.drop ('pizza_id',axis=1)
join1 = join1.drop ('order_id',axis=1)
join1 = join1.drop ('size',axis=1)
# print(join1.head())

#join 1st merged table with pizza Types
join2 = pd.merge(join1,pizza_types,left_on = 'pizza_type_id',right_on= 'pizza_type_id',how='inner')

#drop ingredients column
join2 = join2.drop('ingredients',axis = 1)
# print(join2.head())

# group by category and calculate no_of ordered pizzas
orders_by_category = join2.groupby('category')['order_details_id'].count().reset_index()

#name the columns
orders_by_category.columns = ['category','No_of_pizza_ordered']

#sort by descending order of NO_of_pizza_ordered
orders_by_category = orders_by_category.sort_values(by='No_of_pizza_ordered', ascending=False)

#displaying Hot selling category and orders by category table
print("The Hot selling category :",orders_by_category.iloc[0,0])
print("\n",orders_by_category)
```
---
### The Hot selling category : Classic

- **category , No_of_pizza_ordered**
-  Classic ,              14579
-  Supreme ,              11777
-   Veggie ,              11449
-  Chicken ,              10815
---
6. Month on Month sales revenue
```python
orders['Month_of_year']=orders['date'].dt.month_name()

import pandas as pd

o_revenue1 = pd.merge(order_details,orders,left_on = "order_id",right_on="order_id",how ="inner")
# print(o_revenue1.head())

o_revenue2 = pd.merge(o_revenue1,pizzas,left_on = "pizza_id",right_on = 'pizza_id',how = 'inner')
# print(o_revenue2.head())

o_revenue2['Revenue'] = o_revenue2['quantity'] * o_revenue2['price']

monthly_revenue = o_revenue2.groupby('Month_of_year')['Revenue'].sum().reset_index()
monthly_revenue.columns = ['Month_name','Revenue']

# To ensure the days of the week are in order (Monday to Sunday)
Month_order = ['January','February','March','April','May','June','July','August','September','October','November','December']
monthly_revenue['Month_name'] = pd.Categorical(monthly_revenue['Month_name'],categories=Month_order,ordered=True)

# Sort the DataFrame by the correct day order
monthly_revenue = monthly_revenue.sort_values('Month_name')

print(monthly_revenue)
print("\nThe average revenue in year(in $): ",monthly_revenue["Revenue"].mean())
```

### 4. Data Visualization
1. Track the orders received by times **(Morning,Afternoon,Evening,Night)**
```python
import pandas as pd
# Define a function to categorize time into morning, afternoon, and evening
def categorize_time(row):
    hour = int(row.split(':')[0])  # Extract the hour part from the time column
    if 5 <= hour < 12:
        return 'Morning'
    elif 12 <= hour < 16:
        return 'Afternoon'
    elif 16 <= hour < 20:
        return 'Evening'
    else:
        return 'Night'

# Apply the function to create a new column 'time_period'
orders['time_period'] = orders['time'].apply(categorize_time)

# Ensure that the 'time_period' follows the desired order
time_period_order = ['Morning','Afternoon','Evening','Night']
orders['time_period'] = pd.Categorical(orders['time_period'], categories=time_period_order, ordered=True)

# Group by 'time_period' and count the number of orders for each period
orders_by_time_period = orders.groupby('time_period')['order_id'].count().reset_index()

# Rename columns for clarity
orders_by_time_period.columns = ['time_period', 'number_of_orders']

# Plot the results
import seaborn as sns
import matplotlib.pyplot as plt

plt.figure(figsize=(8, 5))
o = sns.barplot(hue='time_period', y='number_of_orders',data=orders_by_time_period, palette="coolwarm")

for i in o.containers:
  o.bar_label(i,label_type = "edge")

plt.title('Number of Orders Placed by Time Period', fontsize=14)
plt.xlabel('Time Period', fontsize=12)
plt.ylabel('Number of Orders', fontsize=12)
plt.show()
```
![TIME BY SALES](https://github.com/user-attachments/assets/69a9008f-595f-498e-ab16-112aff27d072)

2. Identify which **day** of the week we are getting the most orders ?
```python
#Sales by Day of the week
#converting date column to datetime object
orders['date'] = pd.to_datetime(orders['date'])

# Extract the day of the week (0=Monday, 6=Sunday)
orders['day_of_week']=orders['date'].dt.day_name()

# Group by 'day_of_week' and count the number of orders for each day
orders_by_day = orders.groupby('day_of_week')['order_id'].count().reset_index()

# Rename columns for clarity
orders_by_day.columns=['day_of_week','number_of_orders']

# To ensure the days of the week are in order (Monday to Sunday)
days_order = ['Monday','Tuesday','Wednesday','Thursday','Friday','Saturday','Sunday']
orders_by_day['day_of_week'] = pd.Categorical(orders_by_day['day_of_week'],categories=days_order,ordered=True)

# Sort the DataFrame by the correct day order
orders_by_day = orders_by_day.sort_values('day_of_week')

import matplotlib.pyplot as plt
import seaborn as sns

plt.figure(figsize=(10,6))
g = sns.barplot(x = 'day_of_week',y='number_of_orders',data = orders_by_day,palette = "magma")

# Add data labels to the bars
for i in g.containers:
    g.bar_label(i, label_type='edge')

plt.title("Orders by day")
plt.xlabel("Day Name---->")
plt.ylabel("No.of orders--->")
plt.show()
```
![download](https://github.com/user-attachments/assets/59de12f3-908c-42d4-be8a-fb00308eaeee)

3. Top selling category ?
```python
# print(order_details.head())
# print(pizzas.head())
# print(pizza_types.head())
# print(sales_revenue.head())

#first merging order_details and pizzas datasets
join1 = pd.merge(order_details,pizzas,left_on = "pizza_id",right_on = "pizza_id",how = 'inner')

#removing the columns price,pizza_id, Order_id, size
join1 = join1.drop('price',axis = 1)
join1 = join1.drop ('pizza_id',axis=1)
join1 = join1.drop ('order_id',axis=1)
join1 = join1.drop ('size',axis=1)
# print(join1.head())

#join 1st merged table with pizza Types
join2 = pd.merge(join1,pizza_types,left_on = 'pizza_type_id',right_on= 'pizza_type_id',how='inner')

#drop ingredients column
join2 = join2.drop('ingredients',axis = 1)
# print(join2.head())

# group by category and calculate no_of ordered pizzas
orders_by_category = join2.groupby('category')['order_details_id'].count().reset_index()

#name the columns
orders_by_category.columns = ['category','No_of_pizza_ordered']

#sort by descending order of NO_of_pizza_ordered
orders_by_category = orders_by_category.sort_values(by='No_of_pizza_ordered', ascending=False)

#displaying Hot selling category and orders by category table
print("The Hot selling category :",orders_by_category.iloc[0,0])
print("\n",orders_by_category)

import matplotlib.pyplot as plt
import seaborn as sns
import numpy as np

 #plot with bargraph
plt.figure(figsize=(10,6))
g = sns.barplot(x = 'category',y='No_of_pizza_ordered',data = orders_by_category,palette = "magma")

#adding data labels ,title,xlabel and ylabel
for i in g.containers:
    g.bar_label(i,label_type='edge')
plt.title("Orders by category")
plt.xlabel("category---->")
plt.ylabel("No.of orders-->")
plt.show()

""" New pie chart"""
# Data for pie chart
labels = orders_by_category['category']
sizes = orders_by_category['No_of_pizza_ordered']
# Generate colors using the coolwarm colormap
colors = plt.cm.coolwarm(np.linspace(0, 1, len(labels)))

# Create the pie chart
plt.figure(figsize=(5,5))
plt.pie(sizes, labels=labels, autopct='%1.1f%%', startangle=90, colors = colors)

# Equal aspect ratio ensures the pie is drawn as a circle.
plt.axis('equal')

# Title for the plot
plt.title('Proportion of pizza_ordered by Category', fontsize=14)

# Display the plot
plt.show()
```
![category](https://github.com/user-attachments/assets/09be5953-882e-4505-a346-60ceb2128ad8)
![image](https://github.com/user-attachments/assets/f4028573-15cf-4796-8de3-834cb51d8925)

4. Top 20 selling pizzas
```python
import seaborn as sns
import matplotlib.pyplot as plt


# Group by 'pizza_id' and sum the 'quantity' for each pizza
pizza_sales = order_details.groupby('pizza_id')['quantity'].sum().reset_index()

# Rename columns for clarity
pizza_sales.columns = ['pizza_id', 'total_quantity_sold']
pizza_sales = pizza_sales.sort_values(by='total_quantity_sold', ascending=False)

# Descending order by total sales
top_20 = pizza_sales.head(20)

# Display the pizza sales data
print("The top 5 selling pizzas: \n",pizza_sales.head(5))

# Create a barplot using seaborn
plt.figure(figsize=(15, 6))
h  = sns.barplot(x='pizza_id', y='total_quantity_sold', data = top_20, palette='viridis')
for  k  in h.containers;
      h.bar_label(k,label_type="edge")

# Set plot labels and title
plt.title('Most ordered Pizzas', fontsize=14)
plt.xlabel('Pizza ID', fontsize=12)
plt.ylabel('Total Quantity Sold', fontsize=12)

# Show the plot
plt.xticks(rotation=45)  # Rotate the x labels if necessary
plt.show()
```
![top20](https://github.com/user-attachments/assets/85332772-5a95-4727-bdf4-ba2dfbf7bda3)

5. Lowest 20 selling pizzas
```python
#accesing  lowest 20 rows in pizza sales
lowest_20 = (pizza_sales.tail(20)).sort_values(by='total_quantity_sold', ascending=True)
                                   #taken lowest 20 and sorted in ascending order wrt total_quantity sold
print("The least selling pizzas: \n", lowest_20.head(5))
# Create a barplot using seaborn
plt.figure(figsize=(15, 6))
l = sns.barplot(x='pizza_id', y='total_quantity_sold', data = lowest_20, palette='crest')
for i in l.containers:
  l.bar_label(i,label_type= "edge")
# Set plot labels and title
plt.title('Least ordered Pizzas', fontsize=14)
plt.xlabel('Pizza ID', fontsize=12)
plt.ylabel('Total Quantity Sold', fontsize=12)

# Show the plot
plt.xticks(rotation=45)  # Rotate the x labels if necessary
plt.show()

```
![lowest20](https://github.com/user-attachments/assets/ded825d9-f272-4794-a3fb-c0523f384b72)

6. Month on month revenue
```python
plt.figure(figsize=(13,5))
m = sns.barplot(x = "Month_name",y = "Revenue",data = monthly_revenue,palette="coolwarm")

for j in m.containers:
  m.bar_label(j,label_type= "center")

plt.title("Monthly revenue Stats")
plt.xlabel("Months")
plt.ylabel("Revenue(in $)")
plt.show()
```
![download (1)](https://github.com/user-attachments/assets/d70457f3-2ab8-4a40-81a4-d4528fd7dd07)

7. Sales based on Size
 ```python
sales_by_size = o_revenue2.groupby("size")["quantity"].sum().reset_index()
sales_by_size.columns =  ["size","sales"]
sales_by_size = sales_by_size.sort_values(by='sales', ascending=True)

print(sales_by_size)

plt.figure(figsize = (10,6))
s = plt.barh(sales_by_size["size"],sales_by_size["sales"],color = 'skyblue')

for bar in s:
  plt.text(bar.get_width() + 1,  # x-coordinate for the text (slightly offset to the right)
             bar.get_y() + bar.get_height()/2,  # y-coordinate for the text (centered vertically)
             f'{bar.get_width():.0f}',  # The label showing the value of the bar
             va='center',  # Vertical alignment of the text
             ha='center',  # Horizontal alignment of the text
             fontsize=10)

plt.title("sales by size")
plt.xlabel("sales")
plt.ylabel("Size")
plt.show()
 ```
![size](https://github.com/user-attachments/assets/8e9c4bf6-80e4-43ad-aef7-82bdca3ae888)

---

## Key Findings and Insights
1. **Most selling category**:
   "Classic" category pizzas.

3. **Average revenue of year**:  
   - 68155$ .

4. **Most ordered pizza**:  
   - big_meat_s , Revenue generated : 22968.0 $

5. **Selling pattern of the week**:  
   - Most of the orders are received on Friday,Thursday,Saturday of the week

6. **Receiving Orders Behavior based on time()Evening ,Af**:  
   - Most orders received In the **Evening** time of the day

7. **Receiving Orders Behavior based on Size**:  
   - Most orders received **L** size ,**M** size, **S** size
---

## How to Run This Project

1. Clone the repository:
   ```bash
   git clone https://github.com/Saikiran-Erukonda/Pizza-sales-dashboard-using-Power-BI
   ```
2. Open with Google Colab
   ```bash
   https://colab.research.google.com/
   ```
2. Install the required libraries:
   ```bash
   pip install pandas numpy matplotlib seaborn
   ```
3. Run the **Jupyter notebook** or **Python script**:
   ```bash
   jupyter notebook pizza_sales_descriptive_analysis.ipynb
   ```



## Future Work
- implementing **machine learning** for prediction of dependent variables in different datasets
- Create an **interactive dashboard** using Plotly or Tableau for live monitoring.

---

## Conclusion
This project offers valuable insights in Pizza selling restaurant, helping both owners and customers make informed decisions. By using **EDA techniques**, we identified key trends and developed actionable recommendations. Future improvements can involve advanced analytics and predictive modeling to further enhance the findings.

---

## License
This project is open-source. Feel free to use and modify the code.

---

## Contact
For any queries, feel free to reach out at:
- **YouTube**: [@Twinkling_deals_edu](https://www.youtube.com/@twinkling_deals)  
- **GitHub**: [My GitHub](https://github.com/Saikiran-Erukonda)  
- **LinkedIn**: [My LinkedIn](https://www.linkedin.com/in/erukonda-saikiran-4379911a3/)  

---
