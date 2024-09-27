
### Step 1: Setting up the Database in Colab

1. **Install SQLite and Create a Database:**

   ```python
   import sqlite3
   import pandas as pd

   # Connect to a database (or create one if it doesn't exist)
   conn = sqlite3.connect('ecommerce.db')
   cursor = conn.cursor()

   # Create customer table
   cursor.execute('''
   CREATE TABLE IF NOT EXISTS customer_dim (
       customer_id INT PRIMARY KEY, 
       customer_name VARCHAR(255), 
       location VARCHAR(255)
   );
   ''')

   # Create product table
   cursor.execute('''
   CREATE TABLE IF NOT EXISTS product_dim (
       product_id INT PRIMARY KEY, 
       product_name VARCHAR(255), 
       category VARCHAR(255)
   );
   ''')

   # Create order table
   cursor.execute('''
   CREATE TABLE IF NOT EXISTS order_fact (
       order_id INT PRIMARY KEY, 
       product_id INT, 
       customer_id INT, 
       quantity INT, 
       order_amount DECIMAL(10, 2), 
       order_date TIMESTAMP
   );
   ''')

   # Insert sample data into the tables
   cursor.execute("INSERT OR IGNORE INTO product_dim VALUES (1, 'Smartphone', 'Electronics');")
   cursor.execute("INSERT OR IGNORE INTO product_dim VALUES (2, 'Laptop', 'Electronics');")
   cursor.execute("INSERT OR IGNORE INTO product_dim VALUES (3, 'Headphones', 'Accessories');")
   cursor.execute("INSERT OR IGNORE INTO customer_dim VALUES (2001, 'Bob', 'California');")
   cursor.execute("INSERT OR IGNORE INTO customer_dim VALUES (2002, 'Alice', 'New York');")
   cursor.execute("INSERT OR IGNORE INTO customer_dim VALUES (2003, 'John', 'Texas');")

   # Sample orders
   cursor.execute("INSERT OR IGNORE INTO order_fact VALUES (1001, 1, 2001, 2, 500.00, '2024-09-25 10:00:00');")
   cursor.execute("INSERT OR IGNORE INTO order_fact VALUES (1002, 2, 2002, 1, 1000.00, '2024-09-26 12:30:00');")
   cursor.execute("INSERT OR IGNORE INTO order_fact VALUES (1003, 3, 2003, 3, 150.00, '2024-09-27 15:00:00');")
   cursor.execute("INSERT OR IGNORE INTO order_fact VALUES (1004, 1, 2002, 1, 250.00, '2024-09-27 16:00:00');")

   # Commit the changes and close the connection
   conn.commit()
   ```

2. **Load the Data into Pandas DataFrames (Optional):**

   You can visualize the data stored in the tables using Pandas:

   ```python
   # Load data into Pandas DataFrame for visualization
   customer_df = pd.read_sql_query("SELECT * FROM customer_dim;", conn)
   product_df = pd.read_sql_query("SELECT * FROM product_dim;", conn)
   order_df = pd.read_sql_query("SELECT * FROM order_fact;", conn)

   print("Customers:")
   print(customer_df)
   print("\nProducts:")
   print(product_df)
   print("\nOrders:")
   print(order_df)
   ```

---

### Step 2: Analyzing Customer Behavior, Order Data, and Product Interactions

1. **1. Analyzing Total Sales per Customer:**

   ```python
   cursor.execute('''
   SELECT 
       customer_dim.customer_name, 
       SUM(order_fact.order_amount) AS total_spent
   FROM order_fact
   JOIN customer_dim ON order_fact.customer_id = customer_dim.customer_id
   GROUP BY customer_dim.customer_name
   ORDER BY total_spent DESC;
   ''')
   customer_sales = cursor.fetchall()
   print("\nTotal Sales per Customer:")
   for row in customer_sales:
       print(row)
   ```

   **Explanation:**
   - This query calculates the total amount each customer has spent.

2. **2. Analyzing Number of Orders per Customer:**

   ```python
   cursor.execute('''
   SELECT 
       customer_dim.customer_name, 
       COUNT(order_fact.order_id) AS number_of_orders
   FROM order_fact
   JOIN customer_dim ON order_fact.customer_id = customer_dim.customer_id
   GROUP BY customer_dim.customer_name
   ORDER BY number_of_orders DESC;
   ''')
   order_count = cursor.fetchall()
   print("\nNumber of Orders per Customer:")
   for row in order_count:
       print(row)
   ```

   **Explanation:**
   - This query counts the number of orders each customer has placed.

3. **3. Analyzing Most Popular Products (by Quantity Sold):**

   ```python
   cursor.execute('''
   SELECT 
       product_dim.product_name, 
       SUM(order_fact.quantity) AS total_quantity_sold
   FROM order_fact
   JOIN product_dim ON order_fact.product_id = product_dim.product_id
   GROUP BY product_dim.product_name
   ORDER BY total_quantity_sold DESC;
   ''')
   popular_products = cursor.fetchall()
   print("\nMost Popular Products (by Quantity Sold):")
   for row in popular_products:
       print(row)
   ```

   **Explanation:**
   - This query finds out which products have been sold the most in terms of quantity.

4. **4. Analyzing Sales Distribution by Product Category:**

   ```python
   cursor.execute('''
   SELECT 
       product_dim.category, 
       SUM(order_fact.order_amount) AS total_sales
   FROM order_fact
   JOIN product_dim ON order_fact.product_id = product_dim.product_id
   GROUP BY product_dim.category
   ORDER BY total_sales DESC;
   ''')
   category_sales = cursor.fetchall()
   print("\nSales Distribution by Product Category:")
   for row in category_sales:
       print(row)
   ```

   **Explanation:**
   - This query calculates the total sales for each product category.

5. **5. Finding the Highest Sales Day:**

   ```python
   cursor.execute('''
   SELECT 
       DATE(order_fact.order_date) AS order_day, 
       SUM(order_fact.order_amount) AS total_sales
   FROM order_fact
   GROUP BY order_day
   ORDER BY total_sales DESC
   LIMIT 1;
   ''')
   highest_sales_day = cursor.fetchone()
   print("\nHighest Sales Day:")
   print(highest_sales_day)
   ```

   **Explanation:**
   - This query identifies the day with the highest total sales.

---

### Step 3: Close the Connection

Make sure to close the database connection after you're done:

```python
conn.close()
```

### Running these Queries in Colab
- Ensure that the database and tables are created and populated as shown in Step 1.
- Execute the SQL queries in Step 2 using the `cursor.execute()` function.
- Print or visualize the results to analyze the data.

