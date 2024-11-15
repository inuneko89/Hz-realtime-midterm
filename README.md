import streamlit as st
import pandas as pd
import pinotdb
import matplotlib.pyplot as plt

# การเชื่อมต่อกับ Apache Pinot
def connect_to_pinot():
    conn = pinotdb.connect(
        host='13.229.112.104',  # หรือที่อยู่ของ Pinot
        port=9000,         # พอร์ตของ Pinot Broker
        path='/query/sql', # พาธของ SQL
        scheme='http',     # ใช้ http หรือ https ขึ้นอยู่กับการตั้งค่า
    )
    return conn

# ดึงข้อมูลจาก Pinot
def get_data_from_pinot(query):
    conn = connect_to_pinot()
    df = pd.read_sql(query, conn)
    return df

# กราฟยอดขายรวมตามประเภทกาแฟ (COFFEE_TYPES)
def plot_total_sales(df):
    plt.figure(figsize=(10, 6))
    coffee_sales = df.groupby('COFFEE_TYPES')['TOTAL_PRICE'].sum().reset_index()
    plt.bar(coffee_sales['COFFEE_TYPES'], coffee_sales['TOTAL_PRICE'], color='skyblue')
    plt.xlabel('Coffee Types')
    plt.ylabel('Total Sales ($)')
    plt.title('Total Sales by Coffee Type')
    st.pyplot(plt)

# กราฟจำนวนคำสั่งซื้อตามผู้ใช้ (USERID)
def plot_quantity_by_user(df):
    plt.figure(figsize=(10, 6))
    user_quantity = df.groupby('USERID')['QUANTITY'].sum().reset_index()
    plt.bar(user_quantity['USERID'], user_quantity['QUANTITY'], color='orange')
    plt.xlabel('User ID')
    plt.ylabel('Total Quantity')
    plt.title('Total Quantity by User')
    st.pyplot(plt)

# กราฟสถานะคำสั่ง (STATUS)
def plot_order_status(df):
    plt.figure(figsize=(8, 6))
    status_count = df['STATUS'].value_counts().reset_index()
    plt.bar(status_count['index'], status_count['STATUS'], color='lightgreen')
    plt.xlabel('Order Status')
    plt.ylabel('Count')
    plt.title('Order Status Distribution')
    st.pyplot(plt)

# Streamlit UI
st.title('Coffee City Orders Analysis')

# SQL Query to fetch data (replace with your actual query)
query = """
SELECT ORDERID, USERID, ORDER_TIMESTAMP, COFFEE_TYPES, QUANTITY, TOTAL_PRICE, STATUS 
FROM COFFEECITY
"""

# ดึงข้อมูลจาก Pinot
df = get_data_from_pinot(query)

# แสดงข้อมูลเบื้องต้น
st.write("Sample Data:", df.head())

# สร้างกราฟต่าง ๆ
plot_total_sales(df)
plot_quantity_by_user(df)
plot_order_status(df)
