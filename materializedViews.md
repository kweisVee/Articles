# Materialized Views? What are they?

I have come across materialized views for the first time at my job. This was something that was never discussed in school as my courses covering databases just simply focused on creating tables, deleting tables, adding data, and my ever-so-favorite JOINS. However, there are yet more concepts that we have to discover which could eventually help the efficiency on how we use these databases, and learning about them is a great way to start.

### First, what are views?

Before I rush to the definition of materialized views, we first will discuss the definition of views. Views are a virtual table that aren't stored in any disk - hence, it doesn't occupy any type of memory which will eventually have zero costs. When accessing data from a view, the time complexity depends on the efficiency of the underlying query. If the query involves multiple JOIN statements, it's better to avoid using a view since it will execute the query each time the view is accessed. Instead, views are most suitable for quick and easy access to infrequent data. Here is an example of creating a view with the number of employees from each department of a company:
`    CREATE VIEW num_employees AS 
    SELECT emp_dept_id, employee_dept_name, COUNT(*)
    FROM emp_details
    GROUP BY emp_dept;`
