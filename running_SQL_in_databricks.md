```python
# Assume we have a DataFrame 'df' created from some source
df = spark.read.csv("/path/to/data.csv", header=True, inferSchema=True)

# Register the DataFrame as a temporary SQL view
df.createOrReplaceTempView("employee_data")

# Now you can run SQL queries on the view
result = spark.sql("""
    SELECT employee_id, name, salary
    FROM employee_data
    WHERE salary > 50000
    ORDER BY salary DESC
""")

# Show the result of the query
result.show()
```
