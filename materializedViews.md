# Materialized Views? What are they?

I have come across materialized views for the first time at my job. This was something that was never discussed in school as my courses covering databases just simply focused on creating tables, deleting tables, adding data, and my ever-so-favorite JOINS. However, there are yet more concepts that we have to discover which could eventually help the efficiency on how we use these databases, and learning about them is a great way to start.

### First, what are views?

Before I rush to the definition of materialized views, we first will discuss the definition of a VIEW. Views are virtual tables that aren't stored in any disk hence, it doesn't occupy any type of memory which will eventually have zero costs. When accessing data from a VIEW, the time complexity depends on the efficiency of the underlying query. If the query involves multiple JOIN statements, it's better to avoid using a view since it will execute the query each time the view is accessed. Instead, views are most suitable for infrequently accessed data. Here is an example of creating a view that displays the order details of a specific order:

```
CREATE OR REPLACE VIEW order_details AS
SELECT o.order_id, o.order_date, o.amount, c.customer_id, c.customer_name
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
WHERE o.order_id = order_id;
```

Views are best used for this query as we might need to access an order detail during packing and delivery, and once it's successfully delivered, we rarely need to access this type of information.

Here's a small tip: if you really want to make use of views but queries that you may have used seem to be complicated, it would be best to use indexed foreign keys most especially when using JOINS. Also, one could consider caching these views avoid repeated computation.

### Materialized Views

Now that we have some background knowledge with views, we're off to Materialized Views. Materialized views are normally viewed as a cache as it is a virtual table within memory. And when is it best used? You guessed it - it's for data that are frequently accessed and yet infrequently changing. These are essential for cutting cost as they are only updated when needed and querying is much more efficient when compared to a View or when queried directly. Though, one must keep in mind that these copies are read-only and cannot be changed. Here is an example of creating a materialized view that displays the employees and on which department they belong in:

```
CREATE MATERIALIZED VIEW employee_details AS
SELECT e.employee_id, e.employee_name, d.department_name
FROM employees e
JOIN departments d ON e.department_id = d.department_id;
```

Materialized views could be used for this scenarios as we don't hire and let go of employees in a daily basis, unless you have an unstable company which means you need to do something quick! But, there will always be situations wherein we will be hiring more employees and we'll let go of some this leads to changing the employees table from which we are selecting from. Then, how do we update our own materialized views?

### Refreshing materialized views

Refreshing a materialized view is essential when you want to update a materialized view when changes have been made to its underlying table. For example, we will be hiring two more employees for two different departments. So if we initially run,

```
SELECT COUNT(*) FROM employee_details;
```

This will be returning 70 which is the number of employees within the company when the materialized view `employee_details` was created.

But upon checking the number of employees we have within the database after adding the two newly hired employees with this query:

```
SELECT COUNT(*) FROM employees;
```

It returns 72 which is the actual number of employees within the company.

#### Types of refreshes

We can refresh a materialized view in three different ways:

-   COMPLETE
    This creates the materialized view straight from scratch by dropping all the contents within the materialized view, and reruning the query all over again in order to repopulate the materialized view with the right data. This however, can reduce efficiency as it will be slow most especially when working with large databases.
-   FAST
    This only updates the rows in the materialized view that have changed based on the last refresh. This can be found through the materialized view logs provided by Oracle that store information about the changes that were made to the base tables. When a fast refresh has been requested, the materialized view logs is accessed in order to identify the rows that have been inserted, updated or deleted.
-   FORCE
    This performs a FAST refresh on the materialized view if possible but will perform a COMPLETE refresh if needed based on the nature of the data or the query.

For Oracle, we can instantly refresh a materialized view with the use of `DBMS_MVIEW.REFRESH()` to refresh a materialized view on demand. It accepts two parameters - _list_ which is the name of the materialized view, and _method_ to specify what type of refresh we want to perform. An example below is how we can perform a COMPLETE refresh on the `employee_details` materialized view:

```
BEGIN
  DBMS_MVIEW.REFRESH(
    list => 'employee_details',
    method => 'COMPLETE'
  );
  DBMS_OUTPUT.PUT_LINE('Materialized view employee_details has been refreshed.');
EXCEPTION
  WHEN OTHERS THEN
    DBMS_OUTPUT.PUT_LINE('Error occurred during refresh: ' || SQLERRM);
END;
```

But this is not ideal, as we won't know the exact time to refresh these materialized views. This is where the ON COMMIT keyword comes in and is used if we want the materialized view to be refreshed every time changes are made to its underlying tables or data. The materialized view is then refreshed once the changes have been **committed**. This is most useful as we want to our materialized views to up-to-date to the latest changes made within our database. An example below is how to create a materialized view and performs a FAST refresh whenever new employee data has been committed to the `employees` table:

```
CREATE MATERIALIZED VIEW employee_details
REFRESH FAST ON COMMIT AS
SELECT e.employee_id, e.employee_name, d.department_name
FROM employees e
JOIN departments d ON e.department_id = d.department_id;
```

By understanding the differences and capabilities of views and materialized views, database administrators and developers can leverage these features effectively to optimize data retrieval, enhance application performance, and ultimately unlock the full potential of their Oracle databases. Embracing the right combination of views and materialized views empowers users to harness the power of data, making informed decisions and driving business success.
