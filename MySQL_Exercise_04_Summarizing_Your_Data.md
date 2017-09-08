
Copyright Jana Schaich Borg/Attribution-NonCommercial 4.0 International (CC BY-NC 4.0)

# MySQL Exercise 4: Summarizing your Data

Last week you practiced retrieving and formatting selected subsets of raw data from individual tables in a database.  In this lesson we are going to learn how to use SQL to run calculations that summarize your data without having to output all the raw rows or entries.  These calculations will serve as building blocks for the queries that will address our business questions about how to improve Dognition test completion rates.

These are the five most common aggregate functions used to summarize information stored in tables:

<img src="https://duke.box.com/shared/static/bc3yclxtwmv8dffis09hwsvskx18u1mc.jpg" width=400 alt="AGGREGATE FUNCTIONS" />

You will use COUNT and SUM very frequently.

COUNT is the only aggregate function that can work on any type of variable.  The other four aggregate functions are only appropriate for numerical data.

All aggregate functions require you to enter either a column name or a "\*" in the parentheses after the function word.   

Let's begin by exploring the COUNT function.
    

## 1. The COUNT function

**First, load the sql library and the Dognition database, and set dognition as the default database.**


```python
%load_ext sql
%sql mysql://studentuser:studentpw@mysqlserver/dognitiondb
%sql USE dognitiondb;
```

    The sql extension is already loaded. To reload it, use:
      %reload_ext sql
    0 rows affected.





    []



The Jupyter interface conveniently tells us how many rows are in our query output, so we can compare the results of the COUNT function to the results of our SELECT function.  If you run:

```mySQL
SELECT breed
FROM dogs 
```

Jupyter tells that 35050 rows are "affected", meaning there are 35050 rows in the output of the query (although, of course, we have limited the display to only 1000 rows at a time).    

**Now try running:**

```mySQL
SELECT COUNT(breed)
FROM dogs 
```


```python
%%sql
SELECT COUNT(breed)
FROM dogs
```

    1 rows affected.





<table>
    <tr>
        <th>COUNT(breed)</th>
    </tr>
    <tr>
        <td>35050</td>
    </tr>
</table>



COUNT is reporting how many rows are in the breed column in total.  COUNT should give you the same output as Jupyter's output without displaying the actual rows of data that are being aggregated.  

You can use DISTINCT (which you learned about in MySQL Exercise 3) with COUNT to count all the unique values in a column, but it must be placed inside the parentheses, immediately before the column that is being counted.  For example, to count the number of distinct breed names contained within all the entries in the breed column you could query:  

```SQL
SELECT COUNT(DISTINCT breed)
  FROM dogs
```

What if you wanted to know how many indivdual dogs successfully completed at least one test?

Since every row in the complete_tests table represents a completed test and we learned earlier that there are no NULL values in the created_at column of the complete_tests table, any non-null Dog_Guid in the complete_tests table will have completed at least one test.  When a column is included in the parentheses, null values are automatically ignored.  Therefore, you could use:

```SQL
SELECT COUNT(DISTINCT Dog_Guid)
  FROM complete_tests
```

**Question 1: Try combining this query with a WHERE clause to find how many individual dogs completed tests after March 1, 2014 (the answer should be 13,289):**


```python
%%sql 
SELECT COUNT(DISTINCT Dog_Guid)
FROM complete_tests
WHERE created_at > '2014_03_01';
```

    1 rows affected.





<table>
    <tr>
        <th>COUNT(DISTINCT Dog_Guid)</th>
    </tr>
    <tr>
        <td>13289</td>
    </tr>
</table>



You can use the "\*" in the parentheses of a COUNT function to count how many rows are in the entire table (or subtable).  There are two fundamental difference between COUNT(\*) and COUNT(column_name), though.  

The first difference is that you cannot use DISTINCT with COUNT(\*). 

**Question 2: To observe the second difference yourself first, count the number of rows in the dogs table using COUNT(\*):**    



```python
%%sql
SELECT COUNT(*)
FROM complete_tests
```

    1 rows affected.





<table>
    <tr>
        <th>COUNT(*)</th>
    </tr>
    <tr>
        <td>193246</td>
    </tr>
</table>



**Question 3: Now count the number of rows in the exclude column of the dogs table:**


```python
%%sql
SELECT exclude
FROM dogs;
```

    35050 rows affected.





<table>
    <tr>
        <th>exclude</th>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
</table>
<span style="font-style:italic;text-align:center;">35050 rows, truncated to displaylimit of 1000</span>



The output of the second query should return a much smaller number than the output of the first query.  That's because:

><mark> When a column is included in a count function, null values are ignored in the count. When an asterisk is included in a count function, nulls are included in the count.</mark>

This will be both useful and important to remember in future queries where you might want to use SELECT(\*) to count items in multiple groups at once.  


**Question 4: How many distinct dogs have an exclude flag in the dogs table (value will be "1")? (the answer should be 853)**


```python
%%sql
SELECT COUNT(DISTINCT dog_guid)
FROM dogs
WHERE exclude = 1;
```

    1 rows affected.





<table>
    <tr>
        <th>COUNT(DISTINCT dog_guid)</th>
    </tr>
    <tr>
        <td>853</td>
    </tr>
</table>



## 2. The SUM Function

The fact that the output of:
```mySQL
SELECT COUNT(exclude) 
FROM dogs
```
was so much lower than:
```mySQL
SELECT COUNT(*)
FROM dogs
```

suggests that there must be many NULL values in the exclude column.  Conveniently, we can combine the SUM function with ISNULL to count exactly how many NULL values there are.  Look up "ISNULL" at this link to MySQL functions I included in an earlier lesson: 

http://www.w3resource.com/mysql/mysql-functions-and-operators.php

You will see that ISNULL is a logical function that returns a 1 for every row that has a NULL value in the specified column, and a 0 for everything else.  If we sum up the number of 1s outputted by ISNULL(exclude), then, we should get the total number of NULL values in the column.  Here's what that query would look like:

```mySQL
SELECT SUM(ISNULL(exclude))
FROM dogs
```

It might be tempting to treat SQL like a calculator and leave out the SELECT statement, but you will quickly see that doesn't work.  

><mark>*Every SQL query that extracts data from a database MUST contain a SELECT statement.*  <mark\>

**Try counting the number of NULL values in the exclude column:**
      
              
     


```python
%%sql
SELECT SUM(ISNULL(exclude))
FROM dogs;
```

    1 rows affected.





<table>
    <tr>
        <th>SUM(ISNULL(exclude))</th>
    </tr>
    <tr>
        <td>34025</td>
    </tr>
</table>



The output should return a value of 34,025. When you add that number to the 1025 entries that have an exclude flag, you get a total of 35,050, which is the number of rows reported by SELECT COUNT(\*) from dogs.

     
     
## 3. The AVG, MIN, and MAX Functions

AVG, MIN, and MAX all work very similarly to SUM.

During the Dognition test, customers were asked the question: "How surprising were [your dog’s name]’s choices?” after completing a test.  Users could choose any number between 1 (not surprising) to 9 (very surprising).  We could retrieve the average, minimum, and maximum rating customers gave to this question after completing the "Eye Contact Game" with the following query:

```mySQL
SELECT test_name, 
AVG(rating) AS AVG_Rating, 
MIN(rating) AS MIN_Rating, 
MAX(rating) AS MAX_Rating
FROM reviews
WHERE test_name="Eye Contact Game";
```

This would give us an output with 4 columns.  The last three columns would have titles reflecting the names inputted after the AS clauses.  Recall that if you want to title a column with a string of text that contains a space, that string will need to be enclosed in quotation marks after the AS clause in your query.
   
    
**Question 5: What is the average, minimum, and maximum ratings given to "Memory versus Pointing" game? (Your answer should be 3.5584, 0, and 9, respectively)**


```python
%%sql
SELECT test_name, AVG(rating) AS AVG_Rating, MIN(rating) AS MIN_Rating, MAX(rating) AS MAX_Rating
FROM reviews
WHERE test_name= "Memory versus Pointing";
```

    1 rows affected.





<table>
    <tr>
        <th>test_name</th>
        <th>AVG_Rating</th>
        <th>MIN_Rating</th>
        <th>MAX_Rating</th>
    </tr>
    <tr>
        <td>Memory versus Pointing</td>
        <td>3.5584</td>
        <td>0</td>
        <td>9</td>
    </tr>
</table>



What if you wanted the average rating for each of the 40 tests in the Reviews table?  One way to do that with the tools you know already is to write 40 separate queries like the ones you wrote above for each test, and then copy or transcribe the results into a separate table in another program like Excel to assemble all the results in one place.  That would be a very tedious and time-consuming exercise.  Fortunately, there is a very simple way to produce the results you want within one query.  That's what we will learn how to do in MySQL Exercise 5.  However, it is important that you feel comfortable with the syntax we have learned thus far before we start taking advantage of that functionality. Practice is the best way to become comfortable!


## Practice incorporating aggregate functions with everything else you've learned so far in your own queries.

**Question 6: How would you query how much time it took to complete each test provided in the exam_answers table, in minutes?  Title the column that represents this data "Duration."**  Note that the exam_answers table has over 2 million rows, so if you don't limit your output, it will take longer than usual to run this query.  (HINT: use the TIMESTAMPDIFF function described at: http://www.w3resource.com/mysql/date-and-time-functions/date-and-time-functions.php.  It might seem unkind of me to keep suggesting you look up and use new functions I haven't demonstrated for you, but I really want you to become confident that you know how to look up and use new functions when you need them!  It will give you a very competitive edge in the business world.)  



```python
%%sql
SELECT TIMESTAMPDIFF(minute,start_time,end_time) AS Duration 
FROM exam_answers
LIMIT 2000
```

    2000 rows affected.





<table>
    <tr>
        <th>Duration</th>
    </tr>
    <tr>
        <td>345139</td>
    </tr>
    <tr>
        <td>345139</td>
    </tr>
    <tr>
        <td>345139</td>
    </tr>
    <tr>
        <td>345138</td>
    </tr>
    <tr>
        <td>345138</td>
    </tr>
    <tr>
        <td>345138</td>
    </tr>
    <tr>
        <td>345138</td>
    </tr>
    <tr>
        <td>345138</td>
    </tr>
    <tr>
        <td>345137</td>
    </tr>
    <tr>
        <td>345137</td>
    </tr>
    <tr>
        <td>345137</td>
    </tr>
    <tr>
        <td>345137</td>
    </tr>
    <tr>
        <td>345137</td>
    </tr>
    <tr>
        <td>345136</td>
    </tr>
    <tr>
        <td>345136</td>
    </tr>
    <tr>
        <td>345136</td>
    </tr>
    <tr>
        <td>345135</td>
    </tr>
    <tr>
        <td>345135</td>
    </tr>
    <tr>
        <td>345134</td>
    </tr>
    <tr>
        <td>345134</td>
    </tr>
    <tr>
        <td>345134</td>
    </tr>
    <tr>
        <td>345134</td>
    </tr>
    <tr>
        <td>345133</td>
    </tr>
    <tr>
        <td>345132</td>
    </tr>
    <tr>
        <td>345132</td>
    </tr>
    <tr>
        <td>345132</td>
    </tr>
    <tr>
        <td>345132</td>
    </tr>
    <tr>
        <td>345131</td>
    </tr>
    <tr>
        <td>345130</td>
    </tr>
    <tr>
        <td>345129</td>
    </tr>
    <tr>
        <td>345129</td>
    </tr>
    <tr>
        <td>345128</td>
    </tr>
    <tr>
        <td>345128</td>
    </tr>
    <tr>
        <td>345128</td>
    </tr>
    <tr>
        <td>345127</td>
    </tr>
    <tr>
        <td>345127</td>
    </tr>
    <tr>
        <td>345127</td>
    </tr>
    <tr>
        <td>345126</td>
    </tr>
    <tr>
        <td>345126</td>
    </tr>
    <tr>
        <td>345126</td>
    </tr>
    <tr>
        <td>345126</td>
    </tr>
    <tr>
        <td>345126</td>
    </tr>
    <tr>
        <td>345125</td>
    </tr>
    <tr>
        <td>345125</td>
    </tr>
    <tr>
        <td>345125</td>
    </tr>
    <tr>
        <td>345125</td>
    </tr>
    <tr>
        <td>345124</td>
    </tr>
    <tr>
        <td>345124</td>
    </tr>
    <tr>
        <td>345124</td>
    </tr>
    <tr>
        <td>345124</td>
    </tr>
    <tr>
        <td>345123</td>
    </tr>
    <tr>
        <td>345123</td>
    </tr>
    <tr>
        <td>345123</td>
    </tr>
    <tr>
        <td>345123</td>
    </tr>
    <tr>
        <td>345122</td>
    </tr>
    <tr>
        <td>345122</td>
    </tr>
    <tr>
        <td>345122</td>
    </tr>
    <tr>
        <td>345122</td>
    </tr>
    <tr>
        <td>345121</td>
    </tr>
    <tr>
        <td>345121</td>
    </tr>
    <tr>
        <td>345121</td>
    </tr>
    <tr>
        <td>345121</td>
    </tr>
    <tr>
        <td>345121</td>
    </tr>
    <tr>
        <td>345120</td>
    </tr>
    <tr>
        <td>345120</td>
    </tr>
    <tr>
        <td>345120</td>
    </tr>
    <tr>
        <td>345120</td>
    </tr>
    <tr>
        <td>345119</td>
    </tr>
    <tr>
        <td>345119</td>
    </tr>
    <tr>
        <td>345119</td>
    </tr>
    <tr>
        <td>345119</td>
    </tr>
    <tr>
        <td>345119</td>
    </tr>
    <tr>
        <td>345118</td>
    </tr>
    <tr>
        <td>345118</td>
    </tr>
    <tr>
        <td>345118</td>
    </tr>
    <tr>
        <td>345118</td>
    </tr>
    <tr>
        <td>345118</td>
    </tr>
    <tr>
        <td>345118</td>
    </tr>
    <tr>
        <td>345117</td>
    </tr>
    <tr>
        <td>344442</td>
    </tr>
    <tr>
        <td>344442</td>
    </tr>
    <tr>
        <td>344442</td>
    </tr>
    <tr>
        <td>344442</td>
    </tr>
    <tr>
        <td>344442</td>
    </tr>
    <tr>
        <td>344441</td>
    </tr>
    <tr>
        <td>344441</td>
    </tr>
    <tr>
        <td>344441</td>
    </tr>
    <tr>
        <td>344441</td>
    </tr>
    <tr>
        <td>344441</td>
    </tr>
    <tr>
        <td>344441</td>
    </tr>
    <tr>
        <td>344441</td>
    </tr>
    <tr>
        <td>344441</td>
    </tr>
    <tr>
        <td>344440</td>
    </tr>
    <tr>
        <td>344440</td>
    </tr>
    <tr>
        <td>344440</td>
    </tr>
    <tr>
        <td>344440</td>
    </tr>
    <tr>
        <td>344440</td>
    </tr>
    <tr>
        <td>344440</td>
    </tr>
    <tr>
        <td>344440</td>
    </tr>
    <tr>
        <td>344440</td>
    </tr>
    <tr>
        <td>344440</td>
    </tr>
    <tr>
        <td>344440</td>
    </tr>
    <tr>
        <td>344440</td>
    </tr>
    <tr>
        <td>344440</td>
    </tr>
    <tr>
        <td>344440</td>
    </tr>
    <tr>
        <td>344440</td>
    </tr>
    <tr>
        <td>344440</td>
    </tr>
    <tr>
        <td>344440</td>
    </tr>
    <tr>
        <td>344440</td>
    </tr>
    <tr>
        <td>344439</td>
    </tr>
    <tr>
        <td>344439</td>
    </tr>
    <tr>
        <td>344439</td>
    </tr>
    <tr>
        <td>344439</td>
    </tr>
    <tr>
        <td>344439</td>
    </tr>
    <tr>
        <td>344439</td>
    </tr>
    <tr>
        <td>344439</td>
    </tr>
    <tr>
        <td>344439</td>
    </tr>
    <tr>
        <td>344439</td>
    </tr>
    <tr>
        <td>344439</td>
    </tr>
    <tr>
        <td>344439</td>
    </tr>
    <tr>
        <td>344439</td>
    </tr>
    <tr>
        <td>344439</td>
    </tr>
    <tr>
        <td>344439</td>
    </tr>
    <tr>
        <td>344438</td>
    </tr>
    <tr>
        <td>344438</td>
    </tr>
    <tr>
        <td>344438</td>
    </tr>
    <tr>
        <td>344438</td>
    </tr>
    <tr>
        <td>344438</td>
    </tr>
    <tr>
        <td>344438</td>
    </tr>
    <tr>
        <td>344438</td>
    </tr>
    <tr>
        <td>344438</td>
    </tr>
    <tr>
        <td>344438</td>
    </tr>
    <tr>
        <td>344438</td>
    </tr>
    <tr>
        <td>344437</td>
    </tr>
    <tr>
        <td>344437</td>
    </tr>
    <tr>
        <td>344437</td>
    </tr>
    <tr>
        <td>344437</td>
    </tr>
    <tr>
        <td>344437</td>
    </tr>
    <tr>
        <td>344437</td>
    </tr>
    <tr>
        <td>344437</td>
    </tr>
    <tr>
        <td>344437</td>
    </tr>
    <tr>
        <td>344437</td>
    </tr>
    <tr>
        <td>344437</td>
    </tr>
    <tr>
        <td>344437</td>
    </tr>
    <tr>
        <td>344436</td>
    </tr>
    <tr>
        <td>344436</td>
    </tr>
    <tr>
        <td>344436</td>
    </tr>
    <tr>
        <td>344436</td>
    </tr>
    <tr>
        <td>344436</td>
    </tr>
    <tr>
        <td>344436</td>
    </tr>
    <tr>
        <td>344436</td>
    </tr>
    <tr>
        <td>344436</td>
    </tr>
    <tr>
        <td>344436</td>
    </tr>
    <tr>
        <td>344436</td>
    </tr>
    <tr>
        <td>344436</td>
    </tr>
    <tr>
        <td>344436</td>
    </tr>
    <tr>
        <td>344436</td>
    </tr>
    <tr>
        <td>344436</td>
    </tr>
    <tr>
        <td>344335</td>
    </tr>
    <tr>
        <td>344335</td>
    </tr>
    <tr>
        <td>344335</td>
    </tr>
    <tr>
        <td>344335</td>
    </tr>
    <tr>
        <td>344335</td>
    </tr>
    <tr>
        <td>344334</td>
    </tr>
    <tr>
        <td>344334</td>
    </tr>
    <tr>
        <td>344334</td>
    </tr>
    <tr>
        <td>344334</td>
    </tr>
    <tr>
        <td>344334</td>
    </tr>
    <tr>
        <td>344334</td>
    </tr>
    <tr>
        <td>344334</td>
    </tr>
    <tr>
        <td>344334</td>
    </tr>
    <tr>
        <td>344334</td>
    </tr>
    <tr>
        <td>344333</td>
    </tr>
    <tr>
        <td>344333</td>
    </tr>
    <tr>
        <td>344333</td>
    </tr>
    <tr>
        <td>344333</td>
    </tr>
    <tr>
        <td>344333</td>
    </tr>
    <tr>
        <td>344333</td>
    </tr>
    <tr>
        <td>344333</td>
    </tr>
    <tr>
        <td>344333</td>
    </tr>
    <tr>
        <td>344333</td>
    </tr>
    <tr>
        <td>344333</td>
    </tr>
    <tr>
        <td>344333</td>
    </tr>
    <tr>
        <td>344333</td>
    </tr>
    <tr>
        <td>344332</td>
    </tr>
    <tr>
        <td>344332</td>
    </tr>
    <tr>
        <td>344332</td>
    </tr>
    <tr>
        <td>344332</td>
    </tr>
    <tr>
        <td>344332</td>
    </tr>
    <tr>
        <td>344332</td>
    </tr>
    <tr>
        <td>344332</td>
    </tr>
    <tr>
        <td>344332</td>
    </tr>
    <tr>
        <td>344332</td>
    </tr>
    <tr>
        <td>344332</td>
    </tr>
    <tr>
        <td>344332</td>
    </tr>
    <tr>
        <td>344332</td>
    </tr>
    <tr>
        <td>344332</td>
    </tr>
    <tr>
        <td>344331</td>
    </tr>
    <tr>
        <td>344331</td>
    </tr>
    <tr>
        <td>344331</td>
    </tr>
    <tr>
        <td>344346</td>
    </tr>
    <tr>
        <td>344346</td>
    </tr>
    <tr>
        <td>344346</td>
    </tr>
    <tr>
        <td>344346</td>
    </tr>
    <tr>
        <td>344346</td>
    </tr>
    <tr>
        <td>344346</td>
    </tr>
    <tr>
        <td>344346</td>
    </tr>
    <tr>
        <td>344346</td>
    </tr>
    <tr>
        <td>344346</td>
    </tr>
    <tr>
        <td>344345</td>
    </tr>
    <tr>
        <td>344345</td>
    </tr>
    <tr>
        <td>344345</td>
    </tr>
    <tr>
        <td>344345</td>
    </tr>
    <tr>
        <td>344345</td>
    </tr>
    <tr>
        <td>344345</td>
    </tr>
    <tr>
        <td>344345</td>
    </tr>
    <tr>
        <td>344345</td>
    </tr>
    <tr>
        <td>344345</td>
    </tr>
    <tr>
        <td>344344</td>
    </tr>
    <tr>
        <td>344344</td>
    </tr>
    <tr>
        <td>344344</td>
    </tr>
    <tr>
        <td>344344</td>
    </tr>
    <tr>
        <td>344344</td>
    </tr>
    <tr>
        <td>344344</td>
    </tr>
    <tr>
        <td>344344</td>
    </tr>
    <tr>
        <td>344344</td>
    </tr>
    <tr>
        <td>344344</td>
    </tr>
    <tr>
        <td>344344</td>
    </tr>
    <tr>
        <td>344343</td>
    </tr>
    <tr>
        <td>344343</td>
    </tr>
    <tr>
        <td>344343</td>
    </tr>
    <tr>
        <td>344343</td>
    </tr>
    <tr>
        <td>344343</td>
    </tr>
    <tr>
        <td>344343</td>
    </tr>
    <tr>
        <td>344343</td>
    </tr>
    <tr>
        <td>344343</td>
    </tr>
    <tr>
        <td>344343</td>
    </tr>
    <tr>
        <td>344332</td>
    </tr>
    <tr>
        <td>344332</td>
    </tr>
    <tr>
        <td>344332</td>
    </tr>
    <tr>
        <td>344332</td>
    </tr>
    <tr>
        <td>344332</td>
    </tr>
    <tr>
        <td>344332</td>
    </tr>
    <tr>
        <td>344332</td>
    </tr>
    <tr>
        <td>344331</td>
    </tr>
    <tr>
        <td>344331</td>
    </tr>
    <tr>
        <td>344331</td>
    </tr>
    <tr>
        <td>344331</td>
    </tr>
    <tr>
        <td>344331</td>
    </tr>
    <tr>
        <td>344331</td>
    </tr>
    <tr>
        <td>344331</td>
    </tr>
    <tr>
        <td>344331</td>
    </tr>
    <tr>
        <td>344330</td>
    </tr>
    <tr>
        <td>344330</td>
    </tr>
    <tr>
        <td>344330</td>
    </tr>
    <tr>
        <td>344330</td>
    </tr>
    <tr>
        <td>344330</td>
    </tr>
    <tr>
        <td>344330</td>
    </tr>
    <tr>
        <td>344330</td>
    </tr>
    <tr>
        <td>344330</td>
    </tr>
    <tr>
        <td>344330</td>
    </tr>
    <tr>
        <td>344330</td>
    </tr>
    <tr>
        <td>344330</td>
    </tr>
    <tr>
        <td>344330</td>
    </tr>
    <tr>
        <td>344329</td>
    </tr>
    <tr>
        <td>344329</td>
    </tr>
    <tr>
        <td>344329</td>
    </tr>
    <tr>
        <td>344329</td>
    </tr>
    <tr>
        <td>344329</td>
    </tr>
    <tr>
        <td>344329</td>
    </tr>
    <tr>
        <td>344329</td>
    </tr>
    <tr>
        <td>344329</td>
    </tr>
    <tr>
        <td>344329</td>
    </tr>
    <tr>
        <td>344329</td>
    </tr>
    <tr>
        <td>344329</td>
    </tr>
    <tr>
        <td>344329</td>
    </tr>
    <tr>
        <td>344328</td>
    </tr>
    <tr>
        <td>344328</td>
    </tr>
    <tr>
        <td>344328</td>
    </tr>
    <tr>
        <td>344328</td>
    </tr>
    <tr>
        <td>344328</td>
    </tr>
    <tr>
        <td>344328</td>
    </tr>
    <tr>
        <td>344328</td>
    </tr>
    <tr>
        <td>344328</td>
    </tr>
    <tr>
        <td>344328</td>
    </tr>
    <tr>
        <td>344328</td>
    </tr>
    <tr>
        <td>344328</td>
    </tr>
    <tr>
        <td>344327</td>
    </tr>
    <tr>
        <td>344327</td>
    </tr>
    <tr>
        <td>344327</td>
    </tr>
    <tr>
        <td>344327</td>
    </tr>
    <tr>
        <td>344327</td>
    </tr>
    <tr>
        <td>344327</td>
    </tr>
    <tr>
        <td>344327</td>
    </tr>
    <tr>
        <td>344327</td>
    </tr>
    <tr>
        <td>344327</td>
    </tr>
    <tr>
        <td>344327</td>
    </tr>
    <tr>
        <td>344326</td>
    </tr>
    <tr>
        <td>344326</td>
    </tr>
    <tr>
        <td>344326</td>
    </tr>
    <tr>
        <td>344326</td>
    </tr>
    <tr>
        <td>344326</td>
    </tr>
    <tr>
        <td>344326</td>
    </tr>
    <tr>
        <td>344326</td>
    </tr>
    <tr>
        <td>344326</td>
    </tr>
    <tr>
        <td>344326</td>
    </tr>
    <tr>
        <td>344326</td>
    </tr>
    <tr>
        <td>344326</td>
    </tr>
    <tr>
        <td>344326</td>
    </tr>
    <tr>
        <td>344326</td>
    </tr>
    <tr>
        <td>344326</td>
    </tr>
    <tr>
        <td>344326</td>
    </tr>
    <tr>
        <td>344326</td>
    </tr>
    <tr>
        <td>344325</td>
    </tr>
    <tr>
        <td>344325</td>
    </tr>
    <tr>
        <td>344325</td>
    </tr>
    <tr>
        <td>344301</td>
    </tr>
    <tr>
        <td>344301</td>
    </tr>
    <tr>
        <td>344301</td>
    </tr>
    <tr>
        <td>344301</td>
    </tr>
    <tr>
        <td>344301</td>
    </tr>
    <tr>
        <td>344301</td>
    </tr>
    <tr>
        <td>344301</td>
    </tr>
    <tr>
        <td>344301</td>
    </tr>
    <tr>
        <td>344301</td>
    </tr>
    <tr>
        <td>344301</td>
    </tr>
    <tr>
        <td>344301</td>
    </tr>
    <tr>
        <td>344301</td>
    </tr>
    <tr>
        <td>344300</td>
    </tr>
    <tr>
        <td>344300</td>
    </tr>
    <tr>
        <td>344300</td>
    </tr>
    <tr>
        <td>344300</td>
    </tr>
    <tr>
        <td>344300</td>
    </tr>
    <tr>
        <td>344300</td>
    </tr>
    <tr>
        <td>344300</td>
    </tr>
    <tr>
        <td>344300</td>
    </tr>
    <tr>
        <td>344300</td>
    </tr>
    <tr>
        <td>344300</td>
    </tr>
    <tr>
        <td>344300</td>
    </tr>
    <tr>
        <td>344300</td>
    </tr>
    <tr>
        <td>344300</td>
    </tr>
    <tr>
        <td>344300</td>
    </tr>
    <tr>
        <td>344300</td>
    </tr>
    <tr>
        <td>344300</td>
    </tr>
    <tr>
        <td>344300</td>
    </tr>
    <tr>
        <td>344300</td>
    </tr>
    <tr>
        <td>344300</td>
    </tr>
    <tr>
        <td>344300</td>
    </tr>
    <tr>
        <td>344300</td>
    </tr>
    <tr>
        <td>344300</td>
    </tr>
    <tr>
        <td>344300</td>
    </tr>
    <tr>
        <td>344300</td>
    </tr>
    <tr>
        <td>344300</td>
    </tr>
    <tr>
        <td>344300</td>
    </tr>
    <tr>
        <td>344300</td>
    </tr>
    <tr>
        <td>344300</td>
    </tr>
    <tr>
        <td>344300</td>
    </tr>
    <tr>
        <td>344300</td>
    </tr>
    <tr>
        <td>344300</td>
    </tr>
    <tr>
        <td>344300</td>
    </tr>
    <tr>
        <td>344300</td>
    </tr>
    <tr>
        <td>344300</td>
    </tr>
    <tr>
        <td>344300</td>
    </tr>
    <tr>
        <td>344300</td>
    </tr>
    <tr>
        <td>344300</td>
    </tr>
    <tr>
        <td>344300</td>
    </tr>
    <tr>
        <td>344300</td>
    </tr>
    <tr>
        <td>344300</td>
    </tr>
    <tr>
        <td>344300</td>
    </tr>
    <tr>
        <td>344300</td>
    </tr>
    <tr>
        <td>344300</td>
    </tr>
    <tr>
        <td>344300</td>
    </tr>
    <tr>
        <td>344300</td>
    </tr>
    <tr>
        <td>344300</td>
    </tr>
    <tr>
        <td>344300</td>
    </tr>
    <tr>
        <td>344300</td>
    </tr>
    <tr>
        <td>344300</td>
    </tr>
    <tr>
        <td>344300</td>
    </tr>
    <tr>
        <td>344300</td>
    </tr>
    <tr>
        <td>344300</td>
    </tr>
    <tr>
        <td>344300</td>
    </tr>
    <tr>
        <td>344300</td>
    </tr>
    <tr>
        <td>344300</td>
    </tr>
    <tr>
        <td>344300</td>
    </tr>
    <tr>
        <td>344300</td>
    </tr>
    <tr>
        <td>344300</td>
    </tr>
    <tr>
        <td>344300</td>
    </tr>
    <tr>
        <td>344300</td>
    </tr>
    <tr>
        <td>344300</td>
    </tr>
    <tr>
        <td>344300</td>
    </tr>
    <tr>
        <td>344300</td>
    </tr>
    <tr>
        <td>344300</td>
    </tr>
    <tr>
        <td>344300</td>
    </tr>
    <tr>
        <td>344300</td>
    </tr>
    <tr>
        <td>344300</td>
    </tr>
    <tr>
        <td>344299</td>
    </tr>
    <tr>
        <td>344297</td>
    </tr>
    <tr>
        <td>344296</td>
    </tr>
    <tr>
        <td>344296</td>
    </tr>
    <tr>
        <td>344296</td>
    </tr>
    <tr>
        <td>344296</td>
    </tr>
    <tr>
        <td>344296</td>
    </tr>
    <tr>
        <td>344296</td>
    </tr>
    <tr>
        <td>344296</td>
    </tr>
    <tr>
        <td>344296</td>
    </tr>
    <tr>
        <td>344296</td>
    </tr>
    <tr>
        <td>344296</td>
    </tr>
    <tr>
        <td>344292</td>
    </tr>
    <tr>
        <td>344292</td>
    </tr>
    <tr>
        <td>344292</td>
    </tr>
    <tr>
        <td>344292</td>
    </tr>
    <tr>
        <td>344292</td>
    </tr>
    <tr>
        <td>344292</td>
    </tr>
    <tr>
        <td>344292</td>
    </tr>
    <tr>
        <td>344292</td>
    </tr>
    <tr>
        <td>344291</td>
    </tr>
    <tr>
        <td>344291</td>
    </tr>
    <tr>
        <td>344291</td>
    </tr>
    <tr>
        <td>344291</td>
    </tr>
    <tr>
        <td>344291</td>
    </tr>
    <tr>
        <td>344291</td>
    </tr>
    <tr>
        <td>344291</td>
    </tr>
    <tr>
        <td>344291</td>
    </tr>
    <tr>
        <td>344291</td>
    </tr>
    <tr>
        <td>344291</td>
    </tr>
    <tr>
        <td>344291</td>
    </tr>
    <tr>
        <td>344291</td>
    </tr>
    <tr>
        <td>344290</td>
    </tr>
    <tr>
        <td>344290</td>
    </tr>
    <tr>
        <td>344290</td>
    </tr>
    <tr>
        <td>344290</td>
    </tr>
    <tr>
        <td>344290</td>
    </tr>
    <tr>
        <td>344290</td>
    </tr>
    <tr>
        <td>344290</td>
    </tr>
    <tr>
        <td>344290</td>
    </tr>
    <tr>
        <td>344290</td>
    </tr>
    <tr>
        <td>344290</td>
    </tr>
    <tr>
        <td>344290</td>
    </tr>
    <tr>
        <td>344290</td>
    </tr>
    <tr>
        <td>344290</td>
    </tr>
    <tr>
        <td>344290</td>
    </tr>
    <tr>
        <td>344290</td>
    </tr>
    <tr>
        <td>344289</td>
    </tr>
    <tr>
        <td>344289</td>
    </tr>
    <tr>
        <td>344289</td>
    </tr>
    <tr>
        <td>344289</td>
    </tr>
    <tr>
        <td>344289</td>
    </tr>
    <tr>
        <td>344289</td>
    </tr>
    <tr>
        <td>344289</td>
    </tr>
    <tr>
        <td>344289</td>
    </tr>
    <tr>
        <td>344289</td>
    </tr>
    <tr>
        <td>344289</td>
    </tr>
    <tr>
        <td>344288</td>
    </tr>
    <tr>
        <td>344288</td>
    </tr>
    <tr>
        <td>344288</td>
    </tr>
    <tr>
        <td>344288</td>
    </tr>
    <tr>
        <td>344288</td>
    </tr>
    <tr>
        <td>344288</td>
    </tr>
    <tr>
        <td>344288</td>
    </tr>
    <tr>
        <td>344288</td>
    </tr>
    <tr>
        <td>344288</td>
    </tr>
    <tr>
        <td>344288</td>
    </tr>
    <tr>
        <td>344288</td>
    </tr>
    <tr>
        <td>344288</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>344288</td>
    </tr>
    <tr>
        <td>344288</td>
    </tr>
    <tr>
        <td>344288</td>
    </tr>
    <tr>
        <td>344287</td>
    </tr>
    <tr>
        <td>344287</td>
    </tr>
    <tr>
        <td>344287</td>
    </tr>
    <tr>
        <td>344287</td>
    </tr>
    <tr>
        <td>344287</td>
    </tr>
    <tr>
        <td>344287</td>
    </tr>
    <tr>
        <td>344287</td>
    </tr>
    <tr>
        <td>344287</td>
    </tr>
    <tr>
        <td>344287</td>
    </tr>
    <tr>
        <td>344287</td>
    </tr>
    <tr>
        <td>344287</td>
    </tr>
    <tr>
        <td>344287</td>
    </tr>
    <tr>
        <td>344287</td>
    </tr>
    <tr>
        <td>344286</td>
    </tr>
    <tr>
        <td>344286</td>
    </tr>
    <tr>
        <td>344286</td>
    </tr>
    <tr>
        <td>344286</td>
    </tr>
    <tr>
        <td>344286</td>
    </tr>
    <tr>
        <td>344286</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>344284</td>
    </tr>
    <tr>
        <td>344283</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>344277</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>344277</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>344277</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>344277</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>344277</td>
    </tr>
    <tr>
        <td>344276</td>
    </tr>
    <tr>
        <td>344276</td>
    </tr>
    <tr>
        <td>344276</td>
    </tr>
    <tr>
        <td>344276</td>
    </tr>
    <tr>
        <td>344276</td>
    </tr>
    <tr>
        <td>344276</td>
    </tr>
    <tr>
        <td>344275</td>
    </tr>
    <tr>
        <td>344275</td>
    </tr>
    <tr>
        <td>344275</td>
    </tr>
    <tr>
        <td>344275</td>
    </tr>
    <tr>
        <td>344272</td>
    </tr>
    <tr>
        <td>344272</td>
    </tr>
    <tr>
        <td>344272</td>
    </tr>
    <tr>
        <td>344271</td>
    </tr>
    <tr>
        <td>344271</td>
    </tr>
    <tr>
        <td>344271</td>
    </tr>
    <tr>
        <td>344271</td>
    </tr>
    <tr>
        <td>344271</td>
    </tr>
    <tr>
        <td>344271</td>
    </tr>
    <tr>
        <td>344271</td>
    </tr>
    <tr>
        <td>344271</td>
    </tr>
    <tr>
        <td>344270</td>
    </tr>
    <tr>
        <td>344270</td>
    </tr>
    <tr>
        <td>344270</td>
    </tr>
    <tr>
        <td>344270</td>
    </tr>
    <tr>
        <td>344269</td>
    </tr>
    <tr>
        <td>344269</td>
    </tr>
    <tr>
        <td>344269</td>
    </tr>
    <tr>
        <td>344269</td>
    </tr>
    <tr>
        <td>344269</td>
    </tr>
    <tr>
        <td>344269</td>
    </tr>
    <tr>
        <td>344269</td>
    </tr>
    <tr>
        <td>344269</td>
    </tr>
    <tr>
        <td>344269</td>
    </tr>
    <tr>
        <td>344269</td>
    </tr>
    <tr>
        <td>344269</td>
    </tr>
    <tr>
        <td>344268</td>
    </tr>
    <tr>
        <td>344268</td>
    </tr>
    <tr>
        <td>344268</td>
    </tr>
    <tr>
        <td>344268</td>
    </tr>
    <tr>
        <td>344268</td>
    </tr>
    <tr>
        <td>344268</td>
    </tr>
    <tr>
        <td>344268</td>
    </tr>
    <tr>
        <td>344268</td>
    </tr>
    <tr>
        <td>344268</td>
    </tr>
    <tr>
        <td>344268</td>
    </tr>
    <tr>
        <td>344268</td>
    </tr>
    <tr>
        <td>344268</td>
    </tr>
    <tr>
        <td>344268</td>
    </tr>
    <tr>
        <td>344268</td>
    </tr>
    <tr>
        <td>344268</td>
    </tr>
    <tr>
        <td>344267</td>
    </tr>
    <tr>
        <td>344267</td>
    </tr>
    <tr>
        <td>344267</td>
    </tr>
    <tr>
        <td>344267</td>
    </tr>
    <tr>
        <td>344267</td>
    </tr>
    <tr>
        <td>344267</td>
    </tr>
    <tr>
        <td>344267</td>
    </tr>
    <tr>
        <td>344267</td>
    </tr>
    <tr>
        <td>344267</td>
    </tr>
    <tr>
        <td>344267</td>
    </tr>
    <tr>
        <td>344267</td>
    </tr>
    <tr>
        <td>344267</td>
    </tr>
    <tr>
        <td>344267</td>
    </tr>
    <tr>
        <td>344267</td>
    </tr>
    <tr>
        <td>344267</td>
    </tr>
    <tr>
        <td>344267</td>
    </tr>
    <tr>
        <td>344267</td>
    </tr>
    <tr>
        <td>344267</td>
    </tr>
    <tr>
        <td>344267</td>
    </tr>
    <tr>
        <td>344267</td>
    </tr>
    <tr>
        <td>344267</td>
    </tr>
    <tr>
        <td>344266</td>
    </tr>
    <tr>
        <td>344266</td>
    </tr>
    <tr>
        <td>344266</td>
    </tr>
    <tr>
        <td>344266</td>
    </tr>
    <tr>
        <td>344266</td>
    </tr>
    <tr>
        <td>344266</td>
    </tr>
    <tr>
        <td>344266</td>
    </tr>
    <tr>
        <td>344266</td>
    </tr>
    <tr>
        <td>344266</td>
    </tr>
    <tr>
        <td>344266</td>
    </tr>
    <tr>
        <td>344266</td>
    </tr>
    <tr>
        <td>344266</td>
    </tr>
    <tr>
        <td>344266</td>
    </tr>
    <tr>
        <td>344266</td>
    </tr>
    <tr>
        <td>344266</td>
    </tr>
    <tr>
        <td>344266</td>
    </tr>
    <tr>
        <td>344266</td>
    </tr>
    <tr>
        <td>344266</td>
    </tr>
    <tr>
        <td>344266</td>
    </tr>
    <tr>
        <td>344265</td>
    </tr>
    <tr>
        <td>344265</td>
    </tr>
    <tr>
        <td>344265</td>
    </tr>
    <tr>
        <td>344265</td>
    </tr>
    <tr>
        <td>344265</td>
    </tr>
    <tr>
        <td>344265</td>
    </tr>
    <tr>
        <td>344265</td>
    </tr>
    <tr>
        <td>344265</td>
    </tr>
    <tr>
        <td>344265</td>
    </tr>
    <tr>
        <td>344265</td>
    </tr>
    <tr>
        <td>344265</td>
    </tr>
    <tr>
        <td>344265</td>
    </tr>
    <tr>
        <td>344265</td>
    </tr>
    <tr>
        <td>344265</td>
    </tr>
    <tr>
        <td>344265</td>
    </tr>
    <tr>
        <td>344265</td>
    </tr>
    <tr>
        <td>344265</td>
    </tr>
    <tr>
        <td>344265</td>
    </tr>
    <tr>
        <td>344265</td>
    </tr>
    <tr>
        <td>344265</td>
    </tr>
    <tr>
        <td>344265</td>
    </tr>
    <tr>
        <td>344265</td>
    </tr>
    <tr>
        <td>344265</td>
    </tr>
    <tr>
        <td>344265</td>
    </tr>
    <tr>
        <td>344264</td>
    </tr>
    <tr>
        <td>344264</td>
    </tr>
    <tr>
        <td>344264</td>
    </tr>
    <tr>
        <td>344264</td>
    </tr>
    <tr>
        <td>344264</td>
    </tr>
    <tr>
        <td>344264</td>
    </tr>
    <tr>
        <td>344264</td>
    </tr>
    <tr>
        <td>344264</td>
    </tr>
    <tr>
        <td>344264</td>
    </tr>
    <tr>
        <td>344264</td>
    </tr>
    <tr>
        <td>344264</td>
    </tr>
    <tr>
        <td>344264</td>
    </tr>
    <tr>
        <td>344264</td>
    </tr>
    <tr>
        <td>344264</td>
    </tr>
    <tr>
        <td>344264</td>
    </tr>
    <tr>
        <td>344264</td>
    </tr>
    <tr>
        <td>344264</td>
    </tr>
    <tr>
        <td>344264</td>
    </tr>
    <tr>
        <td>344264</td>
    </tr>
    <tr>
        <td>344264</td>
    </tr>
    <tr>
        <td>344264</td>
    </tr>
    <tr>
        <td>344264</td>
    </tr>
    <tr>
        <td>344264</td>
    </tr>
    <tr>
        <td>344264</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>344263</td>
    </tr>
    <tr>
        <td>344263</td>
    </tr>
    <tr>
        <td>344263</td>
    </tr>
    <tr>
        <td>344263</td>
    </tr>
    <tr>
        <td>344263</td>
    </tr>
    <tr>
        <td>344263</td>
    </tr>
    <tr>
        <td>344263</td>
    </tr>
    <tr>
        <td>344263</td>
    </tr>
    <tr>
        <td>344263</td>
    </tr>
    <tr>
        <td>344263</td>
    </tr>
    <tr>
        <td>344263</td>
    </tr>
    <tr>
        <td>344263</td>
    </tr>
    <tr>
        <td>344263</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>344214</td>
    </tr>
    <tr>
        <td>344214</td>
    </tr>
    <tr>
        <td>344214</td>
    </tr>
    <tr>
        <td>344214</td>
    </tr>
    <tr>
        <td>344214</td>
    </tr>
    <tr>
        <td>344214</td>
    </tr>
    <tr>
        <td>344214</td>
    </tr>
    <tr>
        <td>344214</td>
    </tr>
    <tr>
        <td>344214</td>
    </tr>
    <tr>
        <td>344214</td>
    </tr>
    <tr>
        <td>344214</td>
    </tr>
    <tr>
        <td>344213</td>
    </tr>
    <tr>
        <td>344213</td>
    </tr>
    <tr>
        <td>344213</td>
    </tr>
    <tr>
        <td>344213</td>
    </tr>
    <tr>
        <td>344213</td>
    </tr>
    <tr>
        <td>344213</td>
    </tr>
    <tr>
        <td>344213</td>
    </tr>
    <tr>
        <td>344213</td>
    </tr>
    <tr>
        <td>344213</td>
    </tr>
    <tr>
        <td>344212</td>
    </tr>
    <tr>
        <td>344212</td>
    </tr>
    <tr>
        <td>344212</td>
    </tr>
    <tr>
        <td>344212</td>
    </tr>
    <tr>
        <td>344212</td>
    </tr>
    <tr>
        <td>344212</td>
    </tr>
    <tr>
        <td>344212</td>
    </tr>
    <tr>
        <td>344212</td>
    </tr>
    <tr>
        <td>344212</td>
    </tr>
    <tr>
        <td>344212</td>
    </tr>
    <tr>
        <td>344212</td>
    </tr>
    <tr>
        <td>344212</td>
    </tr>
    <tr>
        <td>344212</td>
    </tr>
    <tr>
        <td>344212</td>
    </tr>
    <tr>
        <td>344212</td>
    </tr>
    <tr>
        <td>344211</td>
    </tr>
    <tr>
        <td>344211</td>
    </tr>
    <tr>
        <td>344210</td>
    </tr>
    <tr>
        <td>344208</td>
    </tr>
    <tr>
        <td>344208</td>
    </tr>
    <tr>
        <td>344208</td>
    </tr>
    <tr>
        <td>344208</td>
    </tr>
    <tr>
        <td>344208</td>
    </tr>
    <tr>
        <td>344208</td>
    </tr>
    <tr>
        <td>344208</td>
    </tr>
    <tr>
        <td>344208</td>
    </tr>
    <tr>
        <td>344207</td>
    </tr>
    <tr>
        <td>344207</td>
    </tr>
    <tr>
        <td>344207</td>
    </tr>
    <tr>
        <td>344207</td>
    </tr>
    <tr>
        <td>344207</td>
    </tr>
    <tr>
        <td>344207</td>
    </tr>
    <tr>
        <td>344207</td>
    </tr>
    <tr>
        <td>344207</td>
    </tr>
    <tr>
        <td>344207</td>
    </tr>
    <tr>
        <td>344207</td>
    </tr>
    <tr>
        <td>344207</td>
    </tr>
    <tr>
        <td>344207</td>
    </tr>
    <tr>
        <td>344207</td>
    </tr>
    <tr>
        <td>344207</td>
    </tr>
    <tr>
        <td>344207</td>
    </tr>
    <tr>
        <td>344206</td>
    </tr>
    <tr>
        <td>344206</td>
    </tr>
    <tr>
        <td>344206</td>
    </tr>
    <tr>
        <td>344206</td>
    </tr>
    <tr>
        <td>344206</td>
    </tr>
    <tr>
        <td>344206</td>
    </tr>
    <tr>
        <td>344206</td>
    </tr>
    <tr>
        <td>344206</td>
    </tr>
    <tr>
        <td>344206</td>
    </tr>
    <tr>
        <td>344206</td>
    </tr>
    <tr>
        <td>344206</td>
    </tr>
    <tr>
        <td>344206</td>
    </tr>
    <tr>
        <td>344206</td>
    </tr>
    <tr>
        <td>344206</td>
    </tr>
    <tr>
        <td>344206</td>
    </tr>
    <tr>
        <td>344206</td>
    </tr>
    <tr>
        <td>344206</td>
    </tr>
    <tr>
        <td>344205</td>
    </tr>
    <tr>
        <td>2</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>2</td>
    </tr>
    <tr>
        <td>3</td>
    </tr>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>344169</td>
    </tr>
    <tr>
        <td>344169</td>
    </tr>
    <tr>
        <td>344168</td>
    </tr>
    <tr>
        <td>344168</td>
    </tr>
    <tr>
        <td>344168</td>
    </tr>
    <tr>
        <td>344168</td>
    </tr>
    <tr>
        <td>344168</td>
    </tr>
    <tr>
        <td>344168</td>
    </tr>
    <tr>
        <td>344167</td>
    </tr>
    <tr>
        <td>344167</td>
    </tr>
    <tr>
        <td>344167</td>
    </tr>
    <tr>
        <td>344167</td>
    </tr>
    <tr>
        <td>344167</td>
    </tr>
    <tr>
        <td>344167</td>
    </tr>
    <tr>
        <td>344167</td>
    </tr>
    <tr>
        <td>344167</td>
    </tr>
    <tr>
        <td>344166</td>
    </tr>
    <tr>
        <td>344166</td>
    </tr>
    <tr>
        <td>344166</td>
    </tr>
    <tr>
        <td>344166</td>
    </tr>
    <tr>
        <td>344166</td>
    </tr>
    <tr>
        <td>344166</td>
    </tr>
    <tr>
        <td>344166</td>
    </tr>
    <tr>
        <td>344165</td>
    </tr>
    <tr>
        <td>344165</td>
    </tr>
    <tr>
        <td>344165</td>
    </tr>
    <tr>
        <td>344165</td>
    </tr>
    <tr>
        <td>344165</td>
    </tr>
    <tr>
        <td>344165</td>
    </tr>
    <tr>
        <td>344165</td>
    </tr>
    <tr>
        <td>344164</td>
    </tr>
    <tr>
        <td>344164</td>
    </tr>
    <tr>
        <td>344164</td>
    </tr>
    <tr>
        <td>344164</td>
    </tr>
    <tr>
        <td>344164</td>
    </tr>
    <tr>
        <td>344164</td>
    </tr>
    <tr>
        <td>344163</td>
    </tr>
    <tr>
        <td>344163</td>
    </tr>
    <tr>
        <td>344163</td>
    </tr>
    <tr>
        <td>344163</td>
    </tr>
    <tr>
        <td>344163</td>
    </tr>
    <tr>
        <td>344163</td>
    </tr>
    <tr>
        <td>344163</td>
    </tr>
    <tr>
        <td>344163</td>
    </tr>
    <tr>
        <td>344163</td>
    </tr>
    <tr>
        <td>344162</td>
    </tr>
    <tr>
        <td>344162</td>
    </tr>
    <tr>
        <td>344162</td>
    </tr>
    <tr>
        <td>344162</td>
    </tr>
    <tr>
        <td>344162</td>
    </tr>
    <tr>
        <td>344162</td>
    </tr>
    <tr>
        <td>344162</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344161</td>
    </tr>
    <tr>
        <td>344160</td>
    </tr>
    <tr>
        <td>344160</td>
    </tr>
    <tr>
        <td>344160</td>
    </tr>
    <tr>
        <td>344160</td>
    </tr>
    <tr>
        <td>344160</td>
    </tr>
    <tr>
        <td>344160</td>
    </tr>
    <tr>
        <td>344160</td>
    </tr>
    <tr>
        <td>344160</td>
    </tr>
    <tr>
        <td>344159</td>
    </tr>
    <tr>
        <td>344159</td>
    </tr>
    <tr>
        <td>344159</td>
    </tr>
    <tr>
        <td>344159</td>
    </tr>
    <tr>
        <td>344159</td>
    </tr>
    <tr>
        <td>344159</td>
    </tr>
    <tr>
        <td>344159</td>
    </tr>
    <tr>
        <td>344159</td>
    </tr>
    <tr>
        <td>0</td>
    </tr>
    <tr>
        <td>344158</td>
    </tr>
    <tr>
        <td>344157</td>
    </tr>
    <tr>
        <td>344157</td>
    </tr>
    <tr>
        <td>344157</td>
    </tr>
    <tr>
        <td>344157</td>
    </tr>
    <tr>
        <td>344157</td>
    </tr>
    <tr>
        <td>344157</td>
    </tr>
    <tr>
        <td>344157</td>
    </tr>
    <tr>
        <td>344157</td>
    </tr>
    <tr>
        <td>344157</td>
    </tr>
    <tr>
        <td>344156</td>
    </tr>
    <tr>
        <td>344156</td>
    </tr>
    <tr>
        <td>344156</td>
    </tr>
    <tr>
        <td>344156</td>
    </tr>
    <tr>
        <td>344156</td>
    </tr>
    <tr>
        <td>344155</td>
    </tr>
    <tr>
        <td>344155</td>
    </tr>
    <tr>
        <td>8</td>
    </tr>
    <tr>
        <td>344155</td>
    </tr>
    <tr>
        <td>344155</td>
    </tr>
    <tr>
        <td>344155</td>
    </tr>
    <tr>
        <td>344155</td>
    </tr>
    <tr>
        <td>344155</td>
    </tr>
    <tr>
        <td>344155</td>
    </tr>
    <tr>
        <td>344155</td>
    </tr>
    <tr>
        <td>344155</td>
    </tr>
    <tr>
        <td>344154</td>
    </tr>
    <tr>
        <td>344154</td>
    </tr>
    <tr>
        <td>344154</td>
    </tr>
    <tr>
        <td>344154</td>
    </tr>
    <tr>
        <td>344154</td>
    </tr>
    <tr>
        <td>344154</td>
    </tr>
    <tr>
        <td>344154</td>
    </tr>
    <tr>
        <td>344153</td>
    </tr>
    <tr>
        <td>344153</td>
    </tr>
    <tr>
        <td>344153</td>
    </tr>
    <tr>
        <td>344153</td>
    </tr>
    <tr>
        <td>344153</td>
    </tr>
    <tr>
        <td>344153</td>
    </tr>
    <tr>
        <td>344153</td>
    </tr>
    <tr>
        <td>344153</td>
    </tr>
    <tr>
        <td>344153</td>
    </tr>
</table>
<span style="font-style:italic;text-align:center;">2000 rows, truncated to displaylimit of 1000</span>



**Question 7: Include a column for Dog_Guid, start_time, and end_time in your query, and examine the output.  Do you notice anything strange?**  


```python
%%sql
SELECT Dog_guid, start_time, end_time, TIMESTAMPDIFF(minute, start_time, end_time) AS Duration
FROM exam_answers
LIMIT 2000;
```

    2000 rows affected.





<table>
    <tr>
        <th>Dog_guid</th>
        <th>start_time</th>
        <th>end_time</th>
        <th>Duration</th>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 03:58:13</td>
        <td>2013-10-02 20:18:06</td>
        <td>345139</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 03:58:31</td>
        <td>2013-10-02 20:18:06</td>
        <td>345139</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 03:59:03</td>
        <td>2013-10-02 20:18:06</td>
        <td>345139</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 03:59:10</td>
        <td>2013-10-02 20:18:06</td>
        <td>345138</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 03:59:22</td>
        <td>2013-10-02 20:18:06</td>
        <td>345138</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 03:59:36</td>
        <td>2013-10-02 20:18:06</td>
        <td>345138</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 03:59:41</td>
        <td>2013-10-02 20:18:06</td>
        <td>345138</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:00:00</td>
        <td>2013-10-02 20:18:06</td>
        <td>345138</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:00:16</td>
        <td>2013-10-02 20:18:06</td>
        <td>345137</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:00:35</td>
        <td>2013-10-02 20:18:06</td>
        <td>345137</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:00:46</td>
        <td>2013-10-02 20:18:06</td>
        <td>345137</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:00:54</td>
        <td>2013-10-02 20:18:06</td>
        <td>345137</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:01:01</td>
        <td>2013-10-02 20:18:06</td>
        <td>345137</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:01:15</td>
        <td>2013-10-02 20:18:06</td>
        <td>345136</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:01:40</td>
        <td>2013-10-02 20:18:06</td>
        <td>345136</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:02:02</td>
        <td>2013-10-02 20:18:06</td>
        <td>345136</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:02:30</td>
        <td>2013-10-02 20:18:06</td>
        <td>345135</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:03:00</td>
        <td>2013-10-02 20:18:06</td>
        <td>345135</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:03:29</td>
        <td>2013-10-02 20:18:06</td>
        <td>345134</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:03:37</td>
        <td>2013-10-02 20:18:06</td>
        <td>345134</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:03:48</td>
        <td>2013-10-02 20:18:06</td>
        <td>345134</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:03:56</td>
        <td>2013-10-02 20:18:06</td>
        <td>345134</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:04:07</td>
        <td>2013-10-02 20:18:06</td>
        <td>345133</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:05:31</td>
        <td>2013-10-02 20:18:06</td>
        <td>345132</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:05:47</td>
        <td>2013-10-02 20:18:06</td>
        <td>345132</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:05:57</td>
        <td>2013-10-02 20:18:06</td>
        <td>345132</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:06:06</td>
        <td>2013-10-02 20:18:06</td>
        <td>345132</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:06:30</td>
        <td>2013-10-02 20:18:06</td>
        <td>345131</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:07:38</td>
        <td>2013-10-02 20:18:06</td>
        <td>345130</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:08:58</td>
        <td>2013-10-02 20:18:06</td>
        <td>345129</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:09:05</td>
        <td>2013-10-02 20:18:06</td>
        <td>345129</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:09:40</td>
        <td>2013-10-02 20:18:06</td>
        <td>345128</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:09:44</td>
        <td>2013-10-02 20:18:06</td>
        <td>345128</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:10:03</td>
        <td>2013-10-02 20:18:06</td>
        <td>345128</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:10:11</td>
        <td>2013-10-02 20:18:06</td>
        <td>345127</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:10:16</td>
        <td>2013-10-02 20:18:06</td>
        <td>345127</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:10:26</td>
        <td>2013-10-02 20:18:06</td>
        <td>345127</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:11:12</td>
        <td>2013-10-02 20:18:06</td>
        <td>345126</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:11:21</td>
        <td>2013-10-02 20:18:06</td>
        <td>345126</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:11:34</td>
        <td>2013-10-02 20:18:06</td>
        <td>345126</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:11:40</td>
        <td>2013-10-02 20:18:06</td>
        <td>345126</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:11:54</td>
        <td>2013-10-02 20:18:06</td>
        <td>345126</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:12:20</td>
        <td>2013-10-02 20:18:06</td>
        <td>345125</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:12:28</td>
        <td>2013-10-02 20:18:06</td>
        <td>345125</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:12:49</td>
        <td>2013-10-02 20:18:06</td>
        <td>345125</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:13:05</td>
        <td>2013-10-02 20:18:06</td>
        <td>345125</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:13:14</td>
        <td>2013-10-02 20:18:06</td>
        <td>345124</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:13:30</td>
        <td>2013-10-02 20:18:06</td>
        <td>345124</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:13:55</td>
        <td>2013-10-02 20:18:07</td>
        <td>345124</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:14:02</td>
        <td>2013-10-02 20:18:07</td>
        <td>345124</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:14:14</td>
        <td>2013-10-02 20:18:07</td>
        <td>345123</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:14:22</td>
        <td>2013-10-02 20:18:07</td>
        <td>345123</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:14:40</td>
        <td>2013-10-02 20:18:07</td>
        <td>345123</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:15:04</td>
        <td>2013-10-02 20:18:07</td>
        <td>345123</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:15:10</td>
        <td>2013-10-02 20:18:07</td>
        <td>345122</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:15:38</td>
        <td>2013-10-02 20:18:07</td>
        <td>345122</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:15:56</td>
        <td>2013-10-02 20:18:07</td>
        <td>345122</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:16:02</td>
        <td>2013-10-02 20:18:07</td>
        <td>345122</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:16:18</td>
        <td>2013-10-02 20:18:07</td>
        <td>345121</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:16:27</td>
        <td>2013-10-02 20:18:07</td>
        <td>345121</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:16:36</td>
        <td>2013-10-02 20:18:07</td>
        <td>345121</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:16:55</td>
        <td>2013-10-02 20:18:07</td>
        <td>345121</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:17:02</td>
        <td>2013-10-02 20:18:07</td>
        <td>345121</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:17:20</td>
        <td>2013-10-02 20:18:07</td>
        <td>345120</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:17:27</td>
        <td>2013-10-02 20:18:07</td>
        <td>345120</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:17:43</td>
        <td>2013-10-02 20:18:07</td>
        <td>345120</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:18:06</td>
        <td>2013-10-02 20:18:07</td>
        <td>345120</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:18:15</td>
        <td>2013-10-02 20:18:07</td>
        <td>345119</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:18:21</td>
        <td>2013-10-02 20:18:07</td>
        <td>345119</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:18:31</td>
        <td>2013-10-02 20:18:07</td>
        <td>345119</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:18:44</td>
        <td>2013-10-02 20:18:07</td>
        <td>345119</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:18:50</td>
        <td>2013-10-02 20:18:07</td>
        <td>345119</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:19:11</td>
        <td>2013-10-02 20:18:07</td>
        <td>345118</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:19:20</td>
        <td>2013-10-02 20:18:07</td>
        <td>345118</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:19:26</td>
        <td>2013-10-02 20:18:07</td>
        <td>345118</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:19:32</td>
        <td>2013-10-02 20:18:07</td>
        <td>345118</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:19:53</td>
        <td>2013-10-02 20:18:07</td>
        <td>345118</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:20:02</td>
        <td>2013-10-02 20:18:07</td>
        <td>345118</td>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 04:20:24</td>
        <td>2013-10-02 20:18:07</td>
        <td>345117</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:35:28</td>
        <td>2013-10-02 20:18:07</td>
        <td>344442</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:35:38</td>
        <td>2013-10-02 20:18:07</td>
        <td>344442</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:35:47</td>
        <td>2013-10-02 20:18:07</td>
        <td>344442</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:35:54</td>
        <td>2013-10-02 20:18:07</td>
        <td>344442</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:36:01</td>
        <td>2013-10-02 20:18:07</td>
        <td>344442</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:36:10</td>
        <td>2013-10-02 20:18:07</td>
        <td>344441</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:36:24</td>
        <td>2013-10-02 20:18:07</td>
        <td>344441</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:36:33</td>
        <td>2013-10-02 20:18:07</td>
        <td>344441</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:36:37</td>
        <td>2013-10-02 20:18:07</td>
        <td>344441</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:36:42</td>
        <td>2013-10-02 20:18:07</td>
        <td>344441</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:36:48</td>
        <td>2013-10-02 20:18:07</td>
        <td>344441</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:36:54</td>
        <td>2013-10-02 20:18:07</td>
        <td>344441</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:36:59</td>
        <td>2013-10-02 20:18:07</td>
        <td>344441</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:37:08</td>
        <td>2013-10-02 20:18:07</td>
        <td>344440</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:37:13</td>
        <td>2013-10-02 20:18:07</td>
        <td>344440</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:37:20</td>
        <td>2013-10-02 20:18:07</td>
        <td>344440</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:37:27</td>
        <td>2013-10-02 20:18:07</td>
        <td>344440</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:37:33</td>
        <td>2013-10-02 20:18:07</td>
        <td>344440</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:37:37</td>
        <td>2013-10-02 20:18:07</td>
        <td>344440</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:37:42</td>
        <td>2013-10-02 20:18:07</td>
        <td>344440</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:37:45</td>
        <td>2013-10-02 20:18:07</td>
        <td>344440</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:37:48</td>
        <td>2013-10-02 20:18:37</td>
        <td>344440</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:37:50</td>
        <td>2013-10-02 20:18:37</td>
        <td>344440</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:37:55</td>
        <td>2013-10-02 20:18:37</td>
        <td>344440</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:38:05</td>
        <td>2013-10-02 20:18:37</td>
        <td>344440</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:38:09</td>
        <td>2013-10-02 20:18:37</td>
        <td>344440</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:38:15</td>
        <td>2013-10-02 20:18:37</td>
        <td>344440</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:38:25</td>
        <td>2013-10-02 20:18:37</td>
        <td>344440</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:38:30</td>
        <td>2013-10-02 20:18:37</td>
        <td>344440</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:38:34</td>
        <td>2013-10-02 20:18:37</td>
        <td>344440</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:38:38</td>
        <td>2013-10-02 20:18:37</td>
        <td>344439</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:38:41</td>
        <td>2013-10-02 20:18:37</td>
        <td>344439</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:38:46</td>
        <td>2013-10-02 20:18:37</td>
        <td>344439</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:38:49</td>
        <td>2013-10-02 20:18:37</td>
        <td>344439</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:38:55</td>
        <td>2013-10-02 20:18:37</td>
        <td>344439</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:39:01</td>
        <td>2013-10-02 20:18:37</td>
        <td>344439</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:39:07</td>
        <td>2013-10-02 20:18:37</td>
        <td>344439</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:39:12</td>
        <td>2013-10-02 20:18:37</td>
        <td>344439</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:39:16</td>
        <td>2013-10-02 20:18:37</td>
        <td>344439</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:39:20</td>
        <td>2013-10-02 20:18:37</td>
        <td>344439</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:39:23</td>
        <td>2013-10-02 20:18:37</td>
        <td>344439</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:39:29</td>
        <td>2013-10-02 20:18:37</td>
        <td>344439</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:39:31</td>
        <td>2013-10-02 20:18:37</td>
        <td>344439</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:39:34</td>
        <td>2013-10-02 20:18:37</td>
        <td>344439</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:39:42</td>
        <td>2013-10-02 20:18:38</td>
        <td>344438</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:39:50</td>
        <td>2013-10-02 20:18:38</td>
        <td>344438</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:39:55</td>
        <td>2013-10-02 20:18:38</td>
        <td>344438</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:39:58</td>
        <td>2013-10-02 20:18:38</td>
        <td>344438</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:40:09</td>
        <td>2013-10-02 20:18:38</td>
        <td>344438</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:40:13</td>
        <td>2013-10-02 20:18:38</td>
        <td>344438</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:40:18</td>
        <td>2013-10-02 20:18:38</td>
        <td>344438</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:40:25</td>
        <td>2013-10-02 20:18:38</td>
        <td>344438</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:40:30</td>
        <td>2013-10-02 20:18:38</td>
        <td>344438</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:40:35</td>
        <td>2013-10-02 20:18:38</td>
        <td>344438</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:40:41</td>
        <td>2013-10-02 20:18:38</td>
        <td>344437</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:40:45</td>
        <td>2013-10-02 20:18:38</td>
        <td>344437</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:40:50</td>
        <td>2013-10-02 20:18:38</td>
        <td>344437</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:40:54</td>
        <td>2013-10-02 20:18:38</td>
        <td>344437</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:40:56</td>
        <td>2013-10-02 20:18:38</td>
        <td>344437</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:41:01</td>
        <td>2013-10-02 20:18:38</td>
        <td>344437</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:41:06</td>
        <td>2013-10-02 20:18:38</td>
        <td>344437</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:41:10</td>
        <td>2013-10-02 20:18:38</td>
        <td>344437</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:41:13</td>
        <td>2013-10-02 20:18:38</td>
        <td>344437</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:41:20</td>
        <td>2013-10-02 20:18:38</td>
        <td>344437</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:41:22</td>
        <td>2013-10-02 20:18:38</td>
        <td>344437</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:41:43</td>
        <td>2013-10-02 20:18:38</td>
        <td>344436</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:41:48</td>
        <td>2013-10-02 20:18:38</td>
        <td>344436</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:41:51</td>
        <td>2013-10-02 20:18:38</td>
        <td>344436</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:41:53</td>
        <td>2013-10-02 20:18:38</td>
        <td>344436</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:42:01</td>
        <td>2013-10-02 20:18:38</td>
        <td>344436</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:42:05</td>
        <td>2013-10-02 20:18:38</td>
        <td>344436</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:42:11</td>
        <td>2013-10-02 20:18:38</td>
        <td>344436</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:42:16</td>
        <td>2013-10-02 20:18:38</td>
        <td>344436</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:42:18</td>
        <td>2013-10-02 20:18:38</td>
        <td>344436</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:42:23</td>
        <td>2013-10-02 20:18:38</td>
        <td>344436</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:42:26</td>
        <td>2013-10-02 20:18:38</td>
        <td>344436</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:42:29</td>
        <td>2013-10-02 20:18:38</td>
        <td>344436</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:42:34</td>
        <td>2013-10-02 20:18:38</td>
        <td>344436</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 15:42:38</td>
        <td>2013-10-02 20:18:38</td>
        <td>344436</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:23:09</td>
        <td>2013-10-02 20:18:38</td>
        <td>344335</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:23:16</td>
        <td>2013-10-02 20:18:38</td>
        <td>344335</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:23:24</td>
        <td>2013-10-02 20:18:38</td>
        <td>344335</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:23:29</td>
        <td>2013-10-02 20:18:38</td>
        <td>344335</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:23:35</td>
        <td>2013-10-02 20:18:38</td>
        <td>344335</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:23:41</td>
        <td>2013-10-02 20:18:38</td>
        <td>344334</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:23:48</td>
        <td>2013-10-02 20:18:38</td>
        <td>344334</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:23:53</td>
        <td>2013-10-02 20:18:38</td>
        <td>344334</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:23:58</td>
        <td>2013-10-02 20:18:38</td>
        <td>344334</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:24:03</td>
        <td>2013-10-02 20:18:38</td>
        <td>344334</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:24:13</td>
        <td>2013-10-02 20:18:38</td>
        <td>344334</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:24:24</td>
        <td>2013-10-02 20:18:38</td>
        <td>344334</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:24:30</td>
        <td>2013-10-02 20:18:38</td>
        <td>344334</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:24:35</td>
        <td>2013-10-02 20:18:38</td>
        <td>344334</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:24:40</td>
        <td>2013-10-02 20:18:38</td>
        <td>344333</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:24:46</td>
        <td>2013-10-02 20:18:38</td>
        <td>344333</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:24:52</td>
        <td>2013-10-02 20:18:38</td>
        <td>344333</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:25:00</td>
        <td>2013-10-02 20:18:38</td>
        <td>344333</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:25:02</td>
        <td>2013-10-02 20:18:38</td>
        <td>344333</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:25:07</td>
        <td>2013-10-02 20:18:38</td>
        <td>344333</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:25:12</td>
        <td>2013-10-02 20:18:38</td>
        <td>344333</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:25:14</td>
        <td>2013-10-02 20:18:38</td>
        <td>344333</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:25:16</td>
        <td>2013-10-02 20:18:38</td>
        <td>344333</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:25:22</td>
        <td>2013-10-02 20:18:38</td>
        <td>344333</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:25:27</td>
        <td>2013-10-02 20:18:38</td>
        <td>344333</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:25:33</td>
        <td>2013-10-02 20:18:38</td>
        <td>344333</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:25:40</td>
        <td>2013-10-02 20:18:38</td>
        <td>344332</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:25:46</td>
        <td>2013-10-02 20:18:38</td>
        <td>344332</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:25:54</td>
        <td>2013-10-02 20:18:38</td>
        <td>344332</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:25:59</td>
        <td>2013-10-02 20:18:38</td>
        <td>344332</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:26:02</td>
        <td>2013-10-02 20:18:38</td>
        <td>344332</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:26:07</td>
        <td>2013-10-02 20:18:39</td>
        <td>344332</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:26:11</td>
        <td>2013-10-02 20:18:39</td>
        <td>344332</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:26:13</td>
        <td>2013-10-02 20:18:39</td>
        <td>344332</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:26:20</td>
        <td>2013-10-02 20:18:39</td>
        <td>344332</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:26:24</td>
        <td>2013-10-02 20:18:39</td>
        <td>344332</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:26:29</td>
        <td>2013-10-02 20:18:39</td>
        <td>344332</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:26:35</td>
        <td>2013-10-02 20:18:39</td>
        <td>344332</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:26:37</td>
        <td>2013-10-02 20:18:39</td>
        <td>344332</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:26:41</td>
        <td>2013-10-02 20:18:39</td>
        <td>344331</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:26:43</td>
        <td>2013-10-02 20:18:39</td>
        <td>344331</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:26:52</td>
        <td>2013-10-02 20:18:39</td>
        <td>344331</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:26:58</td>
        <td>2013-10-02 20:33:44</td>
        <td>344346</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:27:02</td>
        <td>2013-10-02 20:33:44</td>
        <td>344346</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:27:08</td>
        <td>2013-10-02 20:33:44</td>
        <td>344346</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:27:15</td>
        <td>2013-10-02 20:33:44</td>
        <td>344346</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:27:18</td>
        <td>2013-10-02 20:33:44</td>
        <td>344346</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:27:23</td>
        <td>2013-10-02 20:33:44</td>
        <td>344346</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:27:28</td>
        <td>2013-10-02 20:33:44</td>
        <td>344346</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:27:35</td>
        <td>2013-10-02 20:33:44</td>
        <td>344346</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:27:42</td>
        <td>2013-10-02 20:33:44</td>
        <td>344346</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:27:52</td>
        <td>2013-10-02 20:33:44</td>
        <td>344345</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:27:57</td>
        <td>2013-10-02 20:33:44</td>
        <td>344345</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:28:02</td>
        <td>2013-10-02 20:33:44</td>
        <td>344345</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:28:10</td>
        <td>2013-10-02 20:33:44</td>
        <td>344345</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:28:16</td>
        <td>2013-10-02 20:33:44</td>
        <td>344345</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:28:21</td>
        <td>2013-10-02 20:33:44</td>
        <td>344345</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:28:27</td>
        <td>2013-10-02 20:33:44</td>
        <td>344345</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:28:37</td>
        <td>2013-10-02 20:33:44</td>
        <td>344345</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:28:44</td>
        <td>2013-10-02 20:33:44</td>
        <td>344345</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:28:50</td>
        <td>2013-10-02 20:33:44</td>
        <td>344344</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:28:55</td>
        <td>2013-10-02 20:33:44</td>
        <td>344344</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:29:00</td>
        <td>2013-10-02 20:33:44</td>
        <td>344344</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:29:13</td>
        <td>2013-10-02 20:33:44</td>
        <td>344344</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:29:16</td>
        <td>2013-10-02 20:33:44</td>
        <td>344344</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:29:21</td>
        <td>2013-10-02 20:33:44</td>
        <td>344344</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:29:28</td>
        <td>2013-10-02 20:33:44</td>
        <td>344344</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:29:32</td>
        <td>2013-10-02 20:33:44</td>
        <td>344344</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:29:36</td>
        <td>2013-10-02 20:33:44</td>
        <td>344344</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:29:44</td>
        <td>2013-10-02 20:33:44</td>
        <td>344344</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:29:49</td>
        <td>2013-10-02 20:33:44</td>
        <td>344343</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:29:54</td>
        <td>2013-10-02 20:33:44</td>
        <td>344343</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:30:03</td>
        <td>2013-10-02 20:33:44</td>
        <td>344343</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:30:07</td>
        <td>2013-10-02 20:33:44</td>
        <td>344343</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:30:10</td>
        <td>2013-10-02 20:33:44</td>
        <td>344343</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:30:12</td>
        <td>2013-10-02 20:33:44</td>
        <td>344343</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:30:16</td>
        <td>2013-10-02 20:33:44</td>
        <td>344343</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:30:20</td>
        <td>2013-10-02 20:33:44</td>
        <td>344343</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:30:26</td>
        <td>2013-10-02 20:33:44</td>
        <td>344343</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:41:09</td>
        <td>2013-10-02 20:33:44</td>
        <td>344332</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:41:15</td>
        <td>2013-10-02 20:33:44</td>
        <td>344332</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:41:19</td>
        <td>2013-10-02 20:33:44</td>
        <td>344332</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:41:25</td>
        <td>2013-10-02 20:33:44</td>
        <td>344332</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:41:29</td>
        <td>2013-10-02 20:33:45</td>
        <td>344332</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:41:36</td>
        <td>2013-10-02 20:33:45</td>
        <td>344332</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:41:39</td>
        <td>2013-10-02 20:33:45</td>
        <td>344332</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:41:46</td>
        <td>2013-10-02 20:33:45</td>
        <td>344331</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:41:53</td>
        <td>2013-10-02 20:33:45</td>
        <td>344331</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:41:56</td>
        <td>2013-10-02 20:33:45</td>
        <td>344331</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:42:09</td>
        <td>2013-10-02 20:33:45</td>
        <td>344331</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:42:15</td>
        <td>2013-10-02 20:33:45</td>
        <td>344331</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:42:31</td>
        <td>2013-10-02 20:33:45</td>
        <td>344331</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:42:36</td>
        <td>2013-10-02 20:33:45</td>
        <td>344331</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:42:43</td>
        <td>2013-10-02 20:33:45</td>
        <td>344331</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:42:48</td>
        <td>2013-10-02 20:33:45</td>
        <td>344330</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:42:55</td>
        <td>2013-10-02 20:33:45</td>
        <td>344330</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:43:01</td>
        <td>2013-10-02 20:33:45</td>
        <td>344330</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:43:04</td>
        <td>2013-10-02 20:33:45</td>
        <td>344330</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:43:08</td>
        <td>2013-10-02 20:33:45</td>
        <td>344330</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:43:12</td>
        <td>2013-10-02 20:33:45</td>
        <td>344330</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:43:14</td>
        <td>2013-10-02 20:33:45</td>
        <td>344330</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:43:17</td>
        <td>2013-10-02 20:33:45</td>
        <td>344330</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:43:23</td>
        <td>2013-10-02 20:33:45</td>
        <td>344330</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:43:27</td>
        <td>2013-10-02 20:33:45</td>
        <td>344330</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:43:39</td>
        <td>2013-10-02 20:33:45</td>
        <td>344330</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:43:45</td>
        <td>2013-10-02 20:33:45</td>
        <td>344330</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:43:55</td>
        <td>2013-10-02 20:33:45</td>
        <td>344329</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:43:59</td>
        <td>2013-10-02 20:33:45</td>
        <td>344329</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:44:02</td>
        <td>2013-10-02 20:33:45</td>
        <td>344329</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:44:08</td>
        <td>2013-10-02 20:33:45</td>
        <td>344329</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:44:11</td>
        <td>2013-10-02 20:33:45</td>
        <td>344329</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:44:17</td>
        <td>2013-10-02 20:33:45</td>
        <td>344329</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:44:24</td>
        <td>2013-10-02 20:33:45</td>
        <td>344329</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:44:28</td>
        <td>2013-10-02 20:33:45</td>
        <td>344329</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:44:32</td>
        <td>2013-10-02 20:33:45</td>
        <td>344329</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:44:37</td>
        <td>2013-10-02 20:33:45</td>
        <td>344329</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:44:40</td>
        <td>2013-10-02 20:33:45</td>
        <td>344329</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:44:43</td>
        <td>2013-10-02 20:33:45</td>
        <td>344329</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:44:48</td>
        <td>2013-10-02 20:33:45</td>
        <td>344328</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:44:54</td>
        <td>2013-10-02 20:33:45</td>
        <td>344328</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:45:01</td>
        <td>2013-10-02 20:33:45</td>
        <td>344328</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:45:07</td>
        <td>2013-10-02 20:33:45</td>
        <td>344328</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:45:11</td>
        <td>2013-10-02 20:33:45</td>
        <td>344328</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:45:19</td>
        <td>2013-10-02 20:33:45</td>
        <td>344328</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:45:26</td>
        <td>2013-10-02 20:33:45</td>
        <td>344328</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:45:29</td>
        <td>2013-10-02 20:33:45</td>
        <td>344328</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:45:37</td>
        <td>2013-10-02 20:33:45</td>
        <td>344328</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:45:41</td>
        <td>2013-10-02 20:33:45</td>
        <td>344328</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:45:45</td>
        <td>2013-10-02 20:33:45</td>
        <td>344328</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:45:54</td>
        <td>2013-10-02 20:33:45</td>
        <td>344327</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:46:02</td>
        <td>2013-10-02 20:33:45</td>
        <td>344327</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:46:08</td>
        <td>2013-10-02 20:33:45</td>
        <td>344327</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:46:12</td>
        <td>2013-10-02 20:33:45</td>
        <td>344327</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:46:20</td>
        <td>2013-10-02 20:33:45</td>
        <td>344327</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:46:24</td>
        <td>2013-10-02 20:33:45</td>
        <td>344327</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:46:29</td>
        <td>2013-10-02 20:33:45</td>
        <td>344327</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:46:32</td>
        <td>2013-10-02 20:33:45</td>
        <td>344327</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:46:36</td>
        <td>2013-10-02 20:33:45</td>
        <td>344327</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:46:45</td>
        <td>2013-10-02 20:33:45</td>
        <td>344327</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:46:50</td>
        <td>2013-10-02 20:33:45</td>
        <td>344326</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:46:54</td>
        <td>2013-10-02 20:33:45</td>
        <td>344326</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:46:59</td>
        <td>2013-10-02 20:33:46</td>
        <td>344326</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:47:05</td>
        <td>2013-10-02 20:33:56</td>
        <td>344326</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:47:08</td>
        <td>2013-10-02 20:33:56</td>
        <td>344326</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:47:15</td>
        <td>2013-10-02 20:33:56</td>
        <td>344326</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:47:24</td>
        <td>2013-10-02 20:33:56</td>
        <td>344326</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:47:29</td>
        <td>2013-10-02 20:33:56</td>
        <td>344326</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:47:30</td>
        <td>2013-10-02 20:33:56</td>
        <td>344326</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:47:33</td>
        <td>2013-10-02 20:33:56</td>
        <td>344326</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:47:38</td>
        <td>2013-10-02 20:33:56</td>
        <td>344326</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:47:41</td>
        <td>2013-10-02 20:33:56</td>
        <td>344326</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:47:47</td>
        <td>2013-10-02 20:33:56</td>
        <td>344326</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:47:49</td>
        <td>2013-10-02 20:33:56</td>
        <td>344326</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:47:51</td>
        <td>2013-10-02 20:33:56</td>
        <td>344326</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:47:53</td>
        <td>2013-10-02 20:33:56</td>
        <td>344326</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:47:57</td>
        <td>2013-10-02 20:33:56</td>
        <td>344325</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:48:05</td>
        <td>2013-10-02 20:33:56</td>
        <td>344325</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 17:48:13</td>
        <td>2013-10-02 20:33:56</td>
        <td>344325</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:12:39</td>
        <td>2013-10-02 20:33:56</td>
        <td>344301</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:12:49</td>
        <td>2013-10-02 20:33:56</td>
        <td>344301</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:12:50</td>
        <td>2013-10-02 20:33:56</td>
        <td>344301</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:12:51</td>
        <td>2013-10-02 20:33:56</td>
        <td>344301</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:12:52</td>
        <td>2013-10-02 20:33:56</td>
        <td>344301</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:12:52</td>
        <td>2013-10-02 20:33:56</td>
        <td>344301</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:12:53</td>
        <td>2013-10-02 20:33:56</td>
        <td>344301</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:12:54</td>
        <td>2013-10-02 20:33:56</td>
        <td>344301</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:12:54</td>
        <td>2013-10-02 20:33:56</td>
        <td>344301</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:12:55</td>
        <td>2013-10-02 20:33:56</td>
        <td>344301</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:12:56</td>
        <td>2013-10-02 20:33:56</td>
        <td>344301</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:12:56</td>
        <td>2013-10-02 20:33:56</td>
        <td>344301</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:12:57</td>
        <td>2013-10-02 20:33:56</td>
        <td>344300</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:12:57</td>
        <td>2013-10-02 20:33:56</td>
        <td>344300</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:12:58</td>
        <td>2013-10-02 20:33:57</td>
        <td>344300</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:12:59</td>
        <td>2013-10-02 20:33:57</td>
        <td>344300</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:13:00</td>
        <td>2013-10-02 20:33:57</td>
        <td>344300</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:13:01</td>
        <td>2013-10-02 20:33:57</td>
        <td>344300</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:13:01</td>
        <td>2013-10-02 20:33:57</td>
        <td>344300</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:13:02</td>
        <td>2013-10-02 20:33:57</td>
        <td>344300</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:13:03</td>
        <td>2013-10-02 20:33:57</td>
        <td>344300</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:13:03</td>
        <td>2013-10-02 20:33:57</td>
        <td>344300</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:13:04</td>
        <td>2013-10-02 20:33:57</td>
        <td>344300</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:13:05</td>
        <td>2013-10-02 20:33:57</td>
        <td>344300</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:13:06</td>
        <td>2013-10-02 20:33:57</td>
        <td>344300</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:13:06</td>
        <td>2013-10-02 20:33:57</td>
        <td>344300</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:13:07</td>
        <td>2013-10-02 20:33:57</td>
        <td>344300</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:13:07</td>
        <td>2013-10-02 20:33:57</td>
        <td>344300</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:13:09</td>
        <td>2013-10-02 20:33:57</td>
        <td>344300</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:13:10</td>
        <td>2013-10-02 20:33:57</td>
        <td>344300</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:13:10</td>
        <td>2013-10-02 20:33:57</td>
        <td>344300</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:13:11</td>
        <td>2013-10-02 20:33:57</td>
        <td>344300</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:13:12</td>
        <td>2013-10-02 20:33:57</td>
        <td>344300</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:13:12</td>
        <td>2013-10-02 20:33:57</td>
        <td>344300</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:13:13</td>
        <td>2013-10-02 20:33:57</td>
        <td>344300</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:13:13</td>
        <td>2013-10-02 20:33:57</td>
        <td>344300</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:13:14</td>
        <td>2013-10-02 20:33:57</td>
        <td>344300</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:13:16</td>
        <td>2013-10-02 20:33:57</td>
        <td>344300</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:13:16</td>
        <td>2013-10-02 20:33:57</td>
        <td>344300</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:13:17</td>
        <td>2013-10-02 20:33:57</td>
        <td>344300</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:13:17</td>
        <td>2013-10-02 20:33:57</td>
        <td>344300</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:13:18</td>
        <td>2013-10-02 20:33:57</td>
        <td>344300</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:13:20</td>
        <td>2013-10-02 20:33:57</td>
        <td>344300</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:13:21</td>
        <td>2013-10-02 20:33:57</td>
        <td>344300</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:13:22</td>
        <td>2013-10-02 20:33:57</td>
        <td>344300</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:13:23</td>
        <td>2013-10-02 20:33:57</td>
        <td>344300</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:13:23</td>
        <td>2013-10-02 20:33:57</td>
        <td>344300</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:13:24</td>
        <td>2013-10-02 20:33:57</td>
        <td>344300</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:13:24</td>
        <td>2013-10-02 20:33:57</td>
        <td>344300</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:13:25</td>
        <td>2013-10-02 20:33:57</td>
        <td>344300</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:13:27</td>
        <td>2013-10-02 20:33:57</td>
        <td>344300</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:13:27</td>
        <td>2013-10-02 20:33:57</td>
        <td>344300</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:13:28</td>
        <td>2013-10-02 20:33:57</td>
        <td>344300</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:13:29</td>
        <td>2013-10-02 20:33:57</td>
        <td>344300</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:13:30</td>
        <td>2013-10-02 20:33:57</td>
        <td>344300</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:13:30</td>
        <td>2013-10-02 20:33:57</td>
        <td>344300</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:13:31</td>
        <td>2013-10-02 20:33:57</td>
        <td>344300</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:13:31</td>
        <td>2013-10-02 20:33:57</td>
        <td>344300</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:13:33</td>
        <td>2013-10-02 20:33:57</td>
        <td>344300</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:13:34</td>
        <td>2013-10-02 20:33:57</td>
        <td>344300</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:13:35</td>
        <td>2013-10-02 20:33:57</td>
        <td>344300</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:13:35</td>
        <td>2013-10-02 20:33:57</td>
        <td>344300</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:13:36</td>
        <td>2013-10-02 20:33:57</td>
        <td>344300</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:13:36</td>
        <td>2013-10-02 20:33:57</td>
        <td>344300</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:13:37</td>
        <td>2013-10-02 20:33:57</td>
        <td>344300</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:13:38</td>
        <td>2013-10-02 20:33:57</td>
        <td>344300</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:13:39</td>
        <td>2013-10-02 20:33:57</td>
        <td>344300</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:13:40</td>
        <td>2013-10-02 20:33:57</td>
        <td>344300</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:13:40</td>
        <td>2013-10-02 20:33:57</td>
        <td>344300</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:13:41</td>
        <td>2013-10-02 20:33:57</td>
        <td>344300</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:13:42</td>
        <td>2013-10-02 20:33:57</td>
        <td>344300</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:13:42</td>
        <td>2013-10-02 20:33:57</td>
        <td>344300</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:13:43</td>
        <td>2013-10-02 20:33:57</td>
        <td>344300</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:13:44</td>
        <td>2013-10-02 20:33:57</td>
        <td>344300</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:13:45</td>
        <td>2013-10-02 20:33:57</td>
        <td>344300</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:13:45</td>
        <td>2013-10-02 20:33:57</td>
        <td>344300</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:13:46</td>
        <td>2013-10-02 20:33:58</td>
        <td>344300</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:13:46</td>
        <td>2013-10-02 20:33:58</td>
        <td>344300</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:13:47</td>
        <td>2013-10-02 20:33:58</td>
        <td>344300</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:14:29</td>
        <td>2013-10-02 20:33:58</td>
        <td>344299</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:16:33</td>
        <td>2013-10-02 20:33:58</td>
        <td>344297</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:17:35</td>
        <td>2013-10-02 20:33:58</td>
        <td>344296</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:17:37</td>
        <td>2013-10-02 20:33:58</td>
        <td>344296</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:17:39</td>
        <td>2013-10-02 20:33:58</td>
        <td>344296</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:17:42</td>
        <td>2013-10-02 20:33:58</td>
        <td>344296</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:17:44</td>
        <td>2013-10-02 20:33:58</td>
        <td>344296</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:17:47</td>
        <td>2013-10-02 20:33:58</td>
        <td>344296</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:17:49</td>
        <td>2013-10-02 20:33:58</td>
        <td>344296</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:17:51</td>
        <td>2013-10-02 20:33:58</td>
        <td>344296</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:17:54</td>
        <td>2013-10-02 20:33:58</td>
        <td>344296</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:17:57</td>
        <td>2013-10-02 20:33:58</td>
        <td>344296</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:21:09</td>
        <td>2013-10-02 20:33:58</td>
        <td>344292</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:21:18</td>
        <td>2013-10-02 20:33:58</td>
        <td>344292</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:21:23</td>
        <td>2013-10-02 20:33:58</td>
        <td>344292</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:21:31</td>
        <td>2013-10-02 20:33:58</td>
        <td>344292</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:21:35</td>
        <td>2013-10-02 20:33:58</td>
        <td>344292</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:21:47</td>
        <td>2013-10-02 20:33:58</td>
        <td>344292</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:21:52</td>
        <td>2013-10-02 20:33:58</td>
        <td>344292</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:21:56</td>
        <td>2013-10-02 20:33:58</td>
        <td>344292</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:22:02</td>
        <td>2013-10-02 20:33:58</td>
        <td>344291</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:22:08</td>
        <td>2013-10-02 20:33:58</td>
        <td>344291</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:22:13</td>
        <td>2013-10-02 20:33:58</td>
        <td>344291</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:22:17</td>
        <td>2013-10-02 20:33:58</td>
        <td>344291</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:22:22</td>
        <td>2013-10-02 20:33:58</td>
        <td>344291</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:22:25</td>
        <td>2013-10-02 20:33:58</td>
        <td>344291</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:22:31</td>
        <td>2013-10-02 20:33:58</td>
        <td>344291</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:22:34</td>
        <td>2013-10-02 20:33:58</td>
        <td>344291</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:22:40</td>
        <td>2013-10-02 20:33:58</td>
        <td>344291</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:22:48</td>
        <td>2013-10-02 20:33:58</td>
        <td>344291</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:22:52</td>
        <td>2013-10-02 20:33:58</td>
        <td>344291</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:22:56</td>
        <td>2013-10-02 20:33:58</td>
        <td>344291</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:22:59</td>
        <td>2013-10-02 20:33:58</td>
        <td>344290</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:23:04</td>
        <td>2013-10-02 20:33:58</td>
        <td>344290</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:23:06</td>
        <td>2013-10-02 20:33:58</td>
        <td>344290</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:23:13</td>
        <td>2013-10-02 20:33:58</td>
        <td>344290</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:23:18</td>
        <td>2013-10-02 20:33:58</td>
        <td>344290</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:23:24</td>
        <td>2013-10-02 20:33:58</td>
        <td>344290</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:23:29</td>
        <td>2013-10-02 20:33:58</td>
        <td>344290</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:23:33</td>
        <td>2013-10-02 20:33:58</td>
        <td>344290</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:23:38</td>
        <td>2013-10-02 20:33:58</td>
        <td>344290</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:23:42</td>
        <td>2013-10-02 20:33:58</td>
        <td>344290</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:23:45</td>
        <td>2013-10-02 20:33:58</td>
        <td>344290</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:23:49</td>
        <td>2013-10-02 20:33:58</td>
        <td>344290</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:23:52</td>
        <td>2013-10-02 20:33:58</td>
        <td>344290</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:23:55</td>
        <td>2013-10-02 20:33:58</td>
        <td>344290</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:23:58</td>
        <td>2013-10-02 20:33:58</td>
        <td>344290</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:24:01</td>
        <td>2013-10-02 20:33:58</td>
        <td>344289</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:24:04</td>
        <td>2013-10-02 20:33:58</td>
        <td>344289</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:24:15</td>
        <td>2013-10-02 20:33:58</td>
        <td>344289</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:24:23</td>
        <td>2013-10-02 20:33:58</td>
        <td>344289</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:24:26</td>
        <td>2013-10-02 20:33:58</td>
        <td>344289</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:24:29</td>
        <td>2013-10-02 20:33:58</td>
        <td>344289</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:24:37</td>
        <td>2013-10-02 20:33:58</td>
        <td>344289</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:24:44</td>
        <td>2013-10-02 20:33:58</td>
        <td>344289</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:24:48</td>
        <td>2013-10-02 20:33:58</td>
        <td>344289</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:24:54</td>
        <td>2013-10-02 20:33:58</td>
        <td>344289</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:25:01</td>
        <td>2013-10-02 20:33:58</td>
        <td>344288</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:25:03</td>
        <td>2013-10-02 20:33:58</td>
        <td>344288</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:25:09</td>
        <td>2013-10-02 20:33:58</td>
        <td>344288</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:25:13</td>
        <td>2013-10-02 20:33:59</td>
        <td>344288</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:25:19</td>
        <td>2013-10-02 20:33:59</td>
        <td>344288</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:25:22</td>
        <td>2013-10-02 20:33:59</td>
        <td>344288</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:25:29</td>
        <td>2013-10-02 20:33:59</td>
        <td>344288</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:25:33</td>
        <td>2013-10-02 20:33:59</td>
        <td>344288</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:25:37</td>
        <td>2013-10-02 20:33:59</td>
        <td>344288</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:25:40</td>
        <td>2013-10-02 20:33:59</td>
        <td>344288</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:25:44</td>
        <td>2013-10-02 20:33:59</td>
        <td>344288</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:25:49</td>
        <td>2013-10-02 20:33:59</td>
        <td>344288</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:25:51</td>
        <td>2013-02-05 18:26:54</td>
        <td>1</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:25:53</td>
        <td>2013-10-02 20:33:59</td>
        <td>344288</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:25:56</td>
        <td>2013-10-02 20:33:59</td>
        <td>344288</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:25:59</td>
        <td>2013-10-02 20:33:59</td>
        <td>344288</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:26:05</td>
        <td>2013-10-02 20:33:59</td>
        <td>344287</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:26:09</td>
        <td>2013-10-02 20:33:59</td>
        <td>344287</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:26:12</td>
        <td>2013-10-02 20:33:59</td>
        <td>344287</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:26:17</td>
        <td>2013-10-02 20:33:59</td>
        <td>344287</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:26:20</td>
        <td>2013-10-02 20:33:59</td>
        <td>344287</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:26:25</td>
        <td>2013-10-02 20:33:59</td>
        <td>344287</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:26:32</td>
        <td>2013-10-02 20:33:59</td>
        <td>344287</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:26:35</td>
        <td>2013-10-02 20:33:59</td>
        <td>344287</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:26:36</td>
        <td>2013-10-02 20:33:59</td>
        <td>344287</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:26:41</td>
        <td>2013-10-02 20:33:59</td>
        <td>344287</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:26:43</td>
        <td>2013-10-02 20:33:59</td>
        <td>344287</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:26:48</td>
        <td>2013-10-02 20:33:59</td>
        <td>344287</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:26:57</td>
        <td>2013-10-02 20:33:59</td>
        <td>344287</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:27:00</td>
        <td>2013-10-02 20:33:59</td>
        <td>344286</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:27:01</td>
        <td>2013-10-02 20:33:59</td>
        <td>344286</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:27:03</td>
        <td>2013-10-02 20:33:59</td>
        <td>344286</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:27:06</td>
        <td>2013-10-02 20:33:59</td>
        <td>344286</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:27:11</td>
        <td>2013-10-02 20:33:59</td>
        <td>344286</td>
    </tr>
    <tr>
        <td>fd27baec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:27:16</td>
        <td>2013-10-02 20:33:59</td>
        <td>344286</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:29:32</td>
        <td>2013-02-05 18:31:03</td>
        <td>1</td>
    </tr>
    <tr>
        <td>None</td>
        <td>2013-02-05 18:29:54</td>
        <td>2013-10-02 20:33:59</td>
        <td>344284</td>
    </tr>
    <tr>
        <td>None</td>
        <td>2013-02-05 18:30:02</td>
        <td>2013-10-02 20:33:59</td>
        <td>344283</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:31:36</td>
        <td>2013-02-05 18:31:38</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:31:49</td>
        <td>2013-02-05 18:31:51</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:32:02</td>
        <td>2013-02-05 18:32:04</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:32:11</td>
        <td>2013-02-05 18:32:17</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:32:18</td>
        <td>2013-02-05 18:32:20</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:32:22</td>
        <td>2013-02-05 18:32:24</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:32:48</td>
        <td>2013-02-05 18:32:48</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:32:50</td>
        <td>2013-02-05 18:32:51</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:32:52</td>
        <td>2013-02-05 18:32:52</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:32:52</td>
        <td>2013-02-05 18:32:53</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:32:54</td>
        <td>2013-02-05 18:32:54</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:32:55</td>
        <td>2013-02-05 18:32:56</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:33:05</td>
        <td>2013-02-05 18:33:06</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:33:07</td>
        <td>2013-02-05 18:33:08</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:33:09</td>
        <td>2013-02-05 18:33:10</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:33:11</td>
        <td>2013-02-05 18:33:11</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:33:12</td>
        <td>2013-02-05 18:33:13</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:33:12</td>
        <td>2013-02-05 18:33:13</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:33:13</td>
        <td>2013-02-05 18:33:15</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:33:26</td>
        <td>2013-02-05 18:33:26</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:33:28</td>
        <td>2013-02-05 18:33:29</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:33:29</td>
        <td>2013-02-05 18:33:29</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:33:30</td>
        <td>2013-02-05 18:33:30</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:33:32</td>
        <td>2013-02-05 18:33:32</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:33:33</td>
        <td>2013-02-05 18:33:33</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:33:56</td>
        <td>2013-02-05 18:33:59</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:34:17</td>
        <td>2013-02-05 18:34:20</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:34:21</td>
        <td>2013-02-05 18:34:24</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:34:33</td>
        <td>2013-02-05 18:34:35</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:34:36</td>
        <td>2013-02-05 18:34:38</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:34:44</td>
        <td>2013-02-05 18:34:46</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:35:10</td>
        <td>2013-02-05 18:35:11</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:35:13</td>
        <td>2013-02-05 18:35:13</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:35:15</td>
        <td>2013-02-05 18:35:16</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:35:17</td>
        <td>2013-02-05 18:35:18</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:35:36</td>
        <td>2013-02-05 18:35:38</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:35:39</td>
        <td>2013-02-05 18:35:41</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:35:43</td>
        <td>2013-02-05 18:35:45</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:35:46</td>
        <td>2013-02-05 18:35:47</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:36:09</td>
        <td>2013-10-02 20:33:59</td>
        <td>344277</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:36:17</td>
        <td>2013-02-05 18:36:17</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:36:17</td>
        <td>2013-10-02 20:33:59</td>
        <td>344277</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:36:19</td>
        <td>2013-02-05 18:36:19</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:36:21</td>
        <td>2013-02-05 18:36:22</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:36:23</td>
        <td>2013-02-05 18:36:24</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:36:25</td>
        <td>2013-10-02 20:33:59</td>
        <td>344277</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:36:25</td>
        <td>2013-02-05 18:36:26</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:36:28</td>
        <td>2013-02-05 18:36:28</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:36:34</td>
        <td>2013-10-02 20:33:59</td>
        <td>344277</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:36:37</td>
        <td>2013-02-05 18:36:38</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:36:40</td>
        <td>2013-02-05 18:36:40</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:36:41</td>
        <td>2013-02-05 18:36:42</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:36:43</td>
        <td>2013-02-05 18:36:44</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:36:50</td>
        <td>2013-10-02 20:33:59</td>
        <td>344277</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:37:10</td>
        <td>2013-10-02 20:33:59</td>
        <td>344276</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:37:17</td>
        <td>2013-10-02 20:33:59</td>
        <td>344276</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:37:22</td>
        <td>2013-10-02 20:33:59</td>
        <td>344276</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:37:27</td>
        <td>2013-10-02 20:33:59</td>
        <td>344276</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:37:35</td>
        <td>2013-10-02 20:33:59</td>
        <td>344276</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:37:45</td>
        <td>2013-10-02 20:33:59</td>
        <td>344276</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:38:33</td>
        <td>2013-10-02 20:33:59</td>
        <td>344275</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:38:40</td>
        <td>2013-10-02 20:33:59</td>
        <td>344275</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:38:44</td>
        <td>2013-10-02 20:33:59</td>
        <td>344275</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:38:51</td>
        <td>2013-10-02 20:33:59</td>
        <td>344275</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:41:40</td>
        <td>2013-10-02 20:33:59</td>
        <td>344272</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:41:52</td>
        <td>2013-10-02 20:33:59</td>
        <td>344272</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:41:55</td>
        <td>2013-10-02 20:33:59</td>
        <td>344272</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:42:02</td>
        <td>2013-10-02 20:33:59</td>
        <td>344271</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:42:05</td>
        <td>2013-10-02 20:33:59</td>
        <td>344271</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:42:10</td>
        <td>2013-10-02 20:33:59</td>
        <td>344271</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:42:17</td>
        <td>2013-10-02 20:33:59</td>
        <td>344271</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:42:25</td>
        <td>2013-10-02 20:33:59</td>
        <td>344271</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:42:31</td>
        <td>2013-10-02 20:33:59</td>
        <td>344271</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:42:35</td>
        <td>2013-10-02 20:33:59</td>
        <td>344271</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:42:42</td>
        <td>2013-10-02 20:33:59</td>
        <td>344271</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:43:12</td>
        <td>2013-10-02 20:33:59</td>
        <td>344270</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:43:20</td>
        <td>2013-10-02 20:33:59</td>
        <td>344270</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:43:26</td>
        <td>2013-10-02 20:33:59</td>
        <td>344270</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:43:29</td>
        <td>2013-10-02 20:33:59</td>
        <td>344270</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:44:07</td>
        <td>2013-10-02 20:34:00</td>
        <td>344269</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:44:11</td>
        <td>2013-10-02 20:34:00</td>
        <td>344269</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:44:13</td>
        <td>2013-10-02 20:34:00</td>
        <td>344269</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:44:17</td>
        <td>2013-10-02 20:34:00</td>
        <td>344269</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:44:21</td>
        <td>2013-10-02 20:34:00</td>
        <td>344269</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:44:26</td>
        <td>2013-10-02 20:34:00</td>
        <td>344269</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:44:39</td>
        <td>2013-10-02 20:34:00</td>
        <td>344269</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:44:45</td>
        <td>2013-10-02 20:34:00</td>
        <td>344269</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:44:51</td>
        <td>2013-10-02 20:34:00</td>
        <td>344269</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:44:54</td>
        <td>2013-10-02 20:34:00</td>
        <td>344269</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:44:58</td>
        <td>2013-10-02 20:34:00</td>
        <td>344269</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:45:02</td>
        <td>2013-10-02 20:34:00</td>
        <td>344268</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:45:07</td>
        <td>2013-10-02 20:34:00</td>
        <td>344268</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:45:13</td>
        <td>2013-10-02 20:34:00</td>
        <td>344268</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:45:15</td>
        <td>2013-10-02 20:34:00</td>
        <td>344268</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:45:17</td>
        <td>2013-10-02 20:34:00</td>
        <td>344268</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:45:21</td>
        <td>2013-10-02 20:34:00</td>
        <td>344268</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:45:24</td>
        <td>2013-10-02 20:34:00</td>
        <td>344268</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:45:28</td>
        <td>2013-10-02 20:34:00</td>
        <td>344268</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:45:30</td>
        <td>2013-10-02 20:34:00</td>
        <td>344268</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:45:33</td>
        <td>2013-10-02 20:34:00</td>
        <td>344268</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:45:34</td>
        <td>2013-10-02 20:34:00</td>
        <td>344268</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:45:37</td>
        <td>2013-10-02 20:34:00</td>
        <td>344268</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:45:38</td>
        <td>2013-10-02 20:34:00</td>
        <td>344268</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:45:45</td>
        <td>2013-10-02 20:34:00</td>
        <td>344268</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:45:56</td>
        <td>2013-10-02 20:34:00</td>
        <td>344268</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:46:03</td>
        <td>2013-10-02 20:34:00</td>
        <td>344267</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:46:07</td>
        <td>2013-10-02 20:34:00</td>
        <td>344267</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:46:13</td>
        <td>2013-10-02 20:34:00</td>
        <td>344267</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:46:19</td>
        <td>2013-10-02 20:34:00</td>
        <td>344267</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:46:24</td>
        <td>2013-10-02 20:34:00</td>
        <td>344267</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:46:25</td>
        <td>2013-10-02 20:34:00</td>
        <td>344267</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:46:29</td>
        <td>2013-10-02 20:34:00</td>
        <td>344267</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:46:30</td>
        <td>2013-10-02 20:34:00</td>
        <td>344267</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:46:37</td>
        <td>2013-10-02 20:34:00</td>
        <td>344267</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:46:37</td>
        <td>2013-10-02 20:34:00</td>
        <td>344267</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:46:41</td>
        <td>2013-10-02 20:34:00</td>
        <td>344267</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:46:43</td>
        <td>2013-10-02 20:34:00</td>
        <td>344267</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:46:43</td>
        <td>2013-10-02 20:34:00</td>
        <td>344267</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:46:46</td>
        <td>2013-10-02 20:34:00</td>
        <td>344267</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:46:48</td>
        <td>2013-10-02 20:34:00</td>
        <td>344267</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:46:49</td>
        <td>2013-10-02 20:34:00</td>
        <td>344267</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:46:52</td>
        <td>2013-10-02 20:34:00</td>
        <td>344267</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:46:52</td>
        <td>2013-10-02 20:34:00</td>
        <td>344267</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:46:57</td>
        <td>2013-10-02 20:34:00</td>
        <td>344267</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:46:58</td>
        <td>2013-10-02 20:34:00</td>
        <td>344267</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:47:00</td>
        <td>2013-10-02 20:34:00</td>
        <td>344267</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:47:06</td>
        <td>2013-10-02 20:34:00</td>
        <td>344266</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:47:13</td>
        <td>2013-10-02 20:34:00</td>
        <td>344266</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:47:13</td>
        <td>2013-10-02 20:34:00</td>
        <td>344266</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:47:17</td>
        <td>2013-10-02 20:34:00</td>
        <td>344266</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:47:18</td>
        <td>2013-10-02 20:34:00</td>
        <td>344266</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:47:25</td>
        <td>2013-10-02 20:34:00</td>
        <td>344266</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:47:33</td>
        <td>2013-10-02 20:34:00</td>
        <td>344266</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:47:38</td>
        <td>2013-10-02 20:34:00</td>
        <td>344266</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:47:41</td>
        <td>2013-10-02 20:34:00</td>
        <td>344266</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:47:44</td>
        <td>2013-10-02 20:34:00</td>
        <td>344266</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:47:47</td>
        <td>2013-10-02 20:34:00</td>
        <td>344266</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:47:50</td>
        <td>2013-10-02 20:34:00</td>
        <td>344266</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:47:52</td>
        <td>2013-10-02 20:34:00</td>
        <td>344266</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:47:54</td>
        <td>2013-10-02 20:34:00</td>
        <td>344266</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:47:55</td>
        <td>2013-10-02 20:34:00</td>
        <td>344266</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:47:58</td>
        <td>2013-10-02 20:34:00</td>
        <td>344266</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:47:58</td>
        <td>2013-10-02 20:34:00</td>
        <td>344266</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:48:01</td>
        <td>2013-10-02 20:34:01</td>
        <td>344266</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:48:01</td>
        <td>2013-10-02 20:34:01</td>
        <td>344266</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:48:06</td>
        <td>2013-10-02 20:34:01</td>
        <td>344265</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:48:10</td>
        <td>2013-10-02 20:34:01</td>
        <td>344265</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:48:10</td>
        <td>2013-10-02 20:34:01</td>
        <td>344265</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:48:15</td>
        <td>2013-10-02 20:34:01</td>
        <td>344265</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:48:15</td>
        <td>2013-10-02 20:34:01</td>
        <td>344265</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:48:17</td>
        <td>2013-10-02 20:34:01</td>
        <td>344265</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:48:19</td>
        <td>2013-10-02 20:34:01</td>
        <td>344265</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:48:24</td>
        <td>2013-10-02 20:34:01</td>
        <td>344265</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:48:24</td>
        <td>2013-10-02 20:34:01</td>
        <td>344265</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:48:28</td>
        <td>2013-10-02 20:34:01</td>
        <td>344265</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:48:30</td>
        <td>2013-10-02 20:34:01</td>
        <td>344265</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:48:31</td>
        <td>2013-10-02 20:34:01</td>
        <td>344265</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:48:33</td>
        <td>2013-10-02 20:34:01</td>
        <td>344265</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:48:37</td>
        <td>2013-10-02 20:34:01</td>
        <td>344265</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:48:43</td>
        <td>2013-10-02 20:34:01</td>
        <td>344265</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:48:43</td>
        <td>2013-10-02 20:34:01</td>
        <td>344265</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:48:45</td>
        <td>2013-10-02 20:34:01</td>
        <td>344265</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:48:48</td>
        <td>2013-10-02 20:34:01</td>
        <td>344265</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:48:50</td>
        <td>2013-10-02 20:34:01</td>
        <td>344265</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:48:52</td>
        <td>2013-10-02 20:34:01</td>
        <td>344265</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:48:55</td>
        <td>2013-10-02 20:34:01</td>
        <td>344265</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:48:56</td>
        <td>2013-10-02 20:34:01</td>
        <td>344265</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:49:00</td>
        <td>2013-10-02 20:34:01</td>
        <td>344265</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:49:00</td>
        <td>2013-10-02 20:34:01</td>
        <td>344265</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:49:05</td>
        <td>2013-10-02 20:34:01</td>
        <td>344264</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:49:06</td>
        <td>2013-10-02 20:34:01</td>
        <td>344264</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:49:10</td>
        <td>2013-10-02 20:34:01</td>
        <td>344264</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:49:11</td>
        <td>2013-10-02 20:34:01</td>
        <td>344264</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:49:12</td>
        <td>2013-10-02 20:34:01</td>
        <td>344264</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:49:14</td>
        <td>2013-10-02 20:34:01</td>
        <td>344264</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:49:14</td>
        <td>2013-10-02 20:34:01</td>
        <td>344264</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:49:16</td>
        <td>2013-10-02 20:34:01</td>
        <td>344264</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:49:17</td>
        <td>2013-10-02 20:34:01</td>
        <td>344264</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:49:18</td>
        <td>2013-10-02 20:34:01</td>
        <td>344264</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:49:21</td>
        <td>2013-10-02 20:34:01</td>
        <td>344264</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:49:25</td>
        <td>2013-10-02 20:34:01</td>
        <td>344264</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:49:28</td>
        <td>2013-10-02 20:34:01</td>
        <td>344264</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:49:29</td>
        <td>2013-10-02 20:34:01</td>
        <td>344264</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:49:31</td>
        <td>2013-10-02 20:34:01</td>
        <td>344264</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:49:35</td>
        <td>2013-10-02 20:34:01</td>
        <td>344264</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:49:40</td>
        <td>2013-10-02 20:34:01</td>
        <td>344264</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:49:42</td>
        <td>2013-10-02 20:34:01</td>
        <td>344264</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:49:44</td>
        <td>2013-10-02 20:34:01</td>
        <td>344264</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:49:46</td>
        <td>2013-10-02 20:34:01</td>
        <td>344264</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:49:48</td>
        <td>2013-10-02 20:34:01</td>
        <td>344264</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:49:52</td>
        <td>2013-10-02 20:34:01</td>
        <td>344264</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:49:54</td>
        <td>2013-10-02 20:34:01</td>
        <td>344264</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:49:55</td>
        <td>2013-10-02 20:34:01</td>
        <td>344264</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:49:57</td>
        <td>2013-02-05 18:49:58</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:49:57</td>
        <td>2013-02-05 18:49:57</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:50:02</td>
        <td>2013-10-02 20:34:01</td>
        <td>344263</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:50:06</td>
        <td>2013-10-02 20:34:01</td>
        <td>344263</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:50:07</td>
        <td>2013-10-02 20:34:01</td>
        <td>344263</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:50:12</td>
        <td>2013-10-02 20:34:01</td>
        <td>344263</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:50:16</td>
        <td>2013-10-02 20:34:01</td>
        <td>344263</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:50:20</td>
        <td>2013-10-02 20:34:01</td>
        <td>344263</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:50:27</td>
        <td>2013-10-02 20:34:01</td>
        <td>344263</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:50:31</td>
        <td>2013-10-02 20:34:01</td>
        <td>344263</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:50:33</td>
        <td>2013-10-02 20:34:01</td>
        <td>344263</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:50:35</td>
        <td>2013-10-02 20:34:01</td>
        <td>344263</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:50:40</td>
        <td>2013-10-02 20:34:01</td>
        <td>344263</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:50:45</td>
        <td>2013-10-02 20:34:01</td>
        <td>344263</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:50:49</td>
        <td>2013-10-02 20:34:01</td>
        <td>344263</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 18:56:57</td>
        <td>2013-02-05 18:57:05</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:01:23</td>
        <td>2013-02-05 19:01:31</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:03:41</td>
        <td>2013-02-05 19:03:54</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:04:07</td>
        <td>2013-02-05 19:04:14</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:04:34</td>
        <td>2013-02-05 19:04:42</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:05:12</td>
        <td>2013-02-05 19:05:12</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:05:39</td>
        <td>2013-02-05 19:06:28</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:06:35</td>
        <td>2013-02-05 19:07:01</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:07:10</td>
        <td>2013-02-05 19:07:31</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:12:00</td>
        <td>2013-02-05 19:12:03</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:12:36</td>
        <td>2013-02-05 19:12:38</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:13:18</td>
        <td>2013-02-05 19:13:19</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:13:49</td>
        <td>2013-02-05 19:13:50</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:14:25</td>
        <td>2013-02-05 19:14:26</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:15:00</td>
        <td>2013-02-05 19:15:01</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:17:10</td>
        <td>2013-02-05 19:17:12</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:18:18</td>
        <td>2013-02-05 19:18:21</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:19:11</td>
        <td>2013-02-05 19:19:13</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:19:47</td>
        <td>2013-02-05 19:19:48</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:20:33</td>
        <td>2013-02-05 19:20:34</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:20:39</td>
        <td>2013-02-05 19:21:15</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:24:09</td>
        <td>2013-02-05 19:24:11</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:24:55</td>
        <td>2013-02-05 19:24:57</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:25:42</td>
        <td>2013-02-05 19:25:44</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:26:28</td>
        <td>2013-02-05 19:26:30</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:27:15</td>
        <td>2013-02-05 19:27:16</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:28:26</td>
        <td>2013-02-05 19:28:27</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:39:06</td>
        <td>2013-10-02 20:34:01</td>
        <td>344214</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:39:11</td>
        <td>2013-10-02 20:34:01</td>
        <td>344214</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:39:14</td>
        <td>2013-10-02 20:34:01</td>
        <td>344214</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:39:17</td>
        <td>2013-10-02 20:34:02</td>
        <td>344214</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:39:23</td>
        <td>2013-10-02 20:34:02</td>
        <td>344214</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:39:27</td>
        <td>2013-10-02 20:34:02</td>
        <td>344214</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:39:31</td>
        <td>2013-10-02 20:34:02</td>
        <td>344214</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:39:35</td>
        <td>2013-10-02 20:34:02</td>
        <td>344214</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:39:38</td>
        <td>2013-10-02 20:34:02</td>
        <td>344214</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:39:40</td>
        <td>2013-10-02 20:34:02</td>
        <td>344214</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:39:46</td>
        <td>2013-10-02 20:34:02</td>
        <td>344214</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:40:17</td>
        <td>2013-10-02 20:34:02</td>
        <td>344213</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:40:25</td>
        <td>2013-10-02 20:34:02</td>
        <td>344213</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:40:38</td>
        <td>2013-10-02 20:34:02</td>
        <td>344213</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:40:43</td>
        <td>2013-10-02 20:34:02</td>
        <td>344213</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:40:48</td>
        <td>2013-10-02 20:34:02</td>
        <td>344213</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:40:53</td>
        <td>2013-10-02 20:34:02</td>
        <td>344213</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:40:56</td>
        <td>2013-10-02 20:34:02</td>
        <td>344213</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:40:59</td>
        <td>2013-10-02 20:34:02</td>
        <td>344213</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:41:02</td>
        <td>2013-10-02 20:34:02</td>
        <td>344213</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:41:04</td>
        <td>2013-10-02 20:34:02</td>
        <td>344212</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:41:06</td>
        <td>2013-10-02 20:34:02</td>
        <td>344212</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:41:08</td>
        <td>2013-10-02 20:34:02</td>
        <td>344212</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:41:14</td>
        <td>2013-10-02 20:34:02</td>
        <td>344212</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:41:17</td>
        <td>2013-10-02 20:34:02</td>
        <td>344212</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:41:27</td>
        <td>2013-10-02 20:34:02</td>
        <td>344212</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:41:30</td>
        <td>2013-10-02 20:34:02</td>
        <td>344212</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:41:34</td>
        <td>2013-10-02 20:34:02</td>
        <td>344212</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:41:38</td>
        <td>2013-10-02 20:34:02</td>
        <td>344212</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:41:41</td>
        <td>2013-10-02 20:34:02</td>
        <td>344212</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:41:46</td>
        <td>2013-10-02 20:34:02</td>
        <td>344212</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:41:49</td>
        <td>2013-10-02 20:34:02</td>
        <td>344212</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:41:51</td>
        <td>2013-10-02 20:34:02</td>
        <td>344212</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:41:53</td>
        <td>2013-10-02 20:34:02</td>
        <td>344212</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:42:02</td>
        <td>2013-10-02 20:34:02</td>
        <td>344212</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:42:07</td>
        <td>2013-10-02 20:34:02</td>
        <td>344211</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:42:11</td>
        <td>2013-10-02 20:34:02</td>
        <td>344211</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:43:56</td>
        <td>2013-10-02 20:34:02</td>
        <td>344210</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:45:43</td>
        <td>2013-10-02 20:34:02</td>
        <td>344208</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:45:45</td>
        <td>2013-10-02 20:34:02</td>
        <td>344208</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:45:47</td>
        <td>2013-10-02 20:34:02</td>
        <td>344208</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:45:50</td>
        <td>2013-10-02 20:34:02</td>
        <td>344208</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:45:54</td>
        <td>2013-10-02 20:34:02</td>
        <td>344208</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:45:57</td>
        <td>2013-10-02 20:34:02</td>
        <td>344208</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:45:59</td>
        <td>2013-10-02 20:34:02</td>
        <td>344208</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:46:02</td>
        <td>2013-10-02 20:34:02</td>
        <td>344208</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:46:05</td>
        <td>2013-10-02 20:34:02</td>
        <td>344207</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:46:08</td>
        <td>2013-10-02 20:34:02</td>
        <td>344207</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:46:12</td>
        <td>2013-10-02 20:34:02</td>
        <td>344207</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:46:19</td>
        <td>2013-10-02 20:34:02</td>
        <td>344207</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:46:25</td>
        <td>2013-10-02 20:34:02</td>
        <td>344207</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:46:32</td>
        <td>2013-10-02 20:34:02</td>
        <td>344207</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:46:35</td>
        <td>2013-10-02 20:34:02</td>
        <td>344207</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:46:37</td>
        <td>2013-10-02 20:34:02</td>
        <td>344207</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:46:40</td>
        <td>2013-10-02 20:34:02</td>
        <td>344207</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:46:44</td>
        <td>2013-10-02 20:34:02</td>
        <td>344207</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:46:47</td>
        <td>2013-10-02 20:34:02</td>
        <td>344207</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:46:50</td>
        <td>2013-10-02 20:34:02</td>
        <td>344207</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:46:52</td>
        <td>2013-10-02 20:34:02</td>
        <td>344207</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:46:55</td>
        <td>2013-10-02 20:34:02</td>
        <td>344207</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:47:01</td>
        <td>2013-10-02 20:34:02</td>
        <td>344207</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:47:03</td>
        <td>2013-10-02 20:34:02</td>
        <td>344206</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:47:06</td>
        <td>2013-10-02 20:34:02</td>
        <td>344206</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:47:09</td>
        <td>2013-10-02 20:34:02</td>
        <td>344206</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:47:12</td>
        <td>2013-10-02 20:34:03</td>
        <td>344206</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:47:18</td>
        <td>2013-10-02 20:34:03</td>
        <td>344206</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:47:23</td>
        <td>2013-10-02 20:34:03</td>
        <td>344206</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:47:27</td>
        <td>2013-10-02 20:34:03</td>
        <td>344206</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:47:30</td>
        <td>2013-10-02 20:34:03</td>
        <td>344206</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:47:33</td>
        <td>2013-10-02 20:34:03</td>
        <td>344206</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:47:39</td>
        <td>2013-10-02 20:34:03</td>
        <td>344206</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:47:42</td>
        <td>2013-10-02 20:34:03</td>
        <td>344206</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:47:49</td>
        <td>2013-10-02 20:34:03</td>
        <td>344206</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:47:53</td>
        <td>2013-10-02 20:34:03</td>
        <td>344206</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:47:55</td>
        <td>2013-10-02 20:34:03</td>
        <td>344206</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:47:57</td>
        <td>2013-10-02 20:34:03</td>
        <td>344206</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:48:00</td>
        <td>2013-10-02 20:34:03</td>
        <td>344206</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:48:03</td>
        <td>2013-10-02 20:34:03</td>
        <td>344206</td>
    </tr>
    <tr>
        <td>fd27be84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:48:06</td>
        <td>2013-10-02 20:34:03</td>
        <td>344205</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:49:51</td>
        <td>2013-02-05 19:51:57</td>
        <td>2</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:53:34</td>
        <td>2013-02-05 19:55:33</td>
        <td>1</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:55:46</td>
        <td>2013-02-05 19:57:42</td>
        <td>1</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 19:59:46</td>
        <td>2013-02-05 20:01:52</td>
        <td>2</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:02:06</td>
        <td>2013-02-05 20:05:54</td>
        <td>3</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:06:29</td>
        <td>2013-02-05 20:08:02</td>
        <td>1</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:24:16</td>
        <td>2013-10-02 20:34:03</td>
        <td>344169</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:24:22</td>
        <td>2013-10-02 20:34:03</td>
        <td>344169</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:25:09</td>
        <td>2013-10-02 20:34:03</td>
        <td>344168</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:25:15</td>
        <td>2013-10-02 20:34:03</td>
        <td>344168</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:25:38</td>
        <td>2013-10-02 20:34:03</td>
        <td>344168</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:25:44</td>
        <td>2013-10-02 20:34:03</td>
        <td>344168</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:25:54</td>
        <td>2013-10-02 20:34:03</td>
        <td>344168</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:25:59</td>
        <td>2013-10-02 20:34:03</td>
        <td>344168</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:26:04</td>
        <td>2013-10-02 20:34:03</td>
        <td>344167</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:26:08</td>
        <td>2013-10-02 20:34:03</td>
        <td>344167</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:26:15</td>
        <td>2013-10-02 20:34:03</td>
        <td>344167</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:26:24</td>
        <td>2013-10-02 20:34:03</td>
        <td>344167</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:26:30</td>
        <td>2013-10-02 20:34:03</td>
        <td>344167</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:26:36</td>
        <td>2013-10-02 20:34:03</td>
        <td>344167</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:26:43</td>
        <td>2013-10-02 20:34:03</td>
        <td>344167</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:26:56</td>
        <td>2013-10-02 20:34:03</td>
        <td>344167</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:27:05</td>
        <td>2013-10-02 20:34:03</td>
        <td>344166</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:27:25</td>
        <td>2013-10-02 20:34:03</td>
        <td>344166</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:27:36</td>
        <td>2013-10-02 20:34:03</td>
        <td>344166</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:27:41</td>
        <td>2013-10-02 20:34:03</td>
        <td>344166</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:27:55</td>
        <td>2013-10-02 20:34:03</td>
        <td>344166</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:27:58</td>
        <td>2013-10-02 20:34:03</td>
        <td>344166</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:28:02</td>
        <td>2013-10-02 20:34:03</td>
        <td>344166</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:28:08</td>
        <td>2013-10-02 20:34:03</td>
        <td>344165</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:28:23</td>
        <td>2013-10-02 20:34:03</td>
        <td>344165</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:28:31</td>
        <td>2013-10-02 20:34:03</td>
        <td>344165</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:28:38</td>
        <td>2013-10-02 20:34:03</td>
        <td>344165</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:28:49</td>
        <td>2013-10-02 20:34:03</td>
        <td>344165</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:28:56</td>
        <td>2013-10-02 20:34:03</td>
        <td>344165</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:29:03</td>
        <td>2013-10-02 20:34:03</td>
        <td>344165</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:29:07</td>
        <td>2013-10-02 20:34:03</td>
        <td>344164</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:29:13</td>
        <td>2013-10-02 20:34:03</td>
        <td>344164</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:29:40</td>
        <td>2013-10-02 20:34:03</td>
        <td>344164</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:29:43</td>
        <td>2013-10-02 20:34:03</td>
        <td>344164</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:29:48</td>
        <td>2013-10-02 20:34:03</td>
        <td>344164</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:29:53</td>
        <td>2013-10-02 20:34:03</td>
        <td>344164</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:30:05</td>
        <td>2013-10-02 20:34:03</td>
        <td>344163</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:30:11</td>
        <td>2013-10-02 20:34:03</td>
        <td>344163</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:30:16</td>
        <td>2013-10-02 20:34:03</td>
        <td>344163</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:30:20</td>
        <td>2013-10-02 20:34:03</td>
        <td>344163</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:30:27</td>
        <td>2013-10-02 20:34:03</td>
        <td>344163</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:30:41</td>
        <td>2013-10-02 20:34:03</td>
        <td>344163</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:30:46</td>
        <td>2013-10-02 20:34:03</td>
        <td>344163</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:30:51</td>
        <td>2013-10-02 20:34:03</td>
        <td>344163</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:31:00</td>
        <td>2013-10-02 20:34:03</td>
        <td>344163</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:31:10</td>
        <td>2013-10-02 20:34:03</td>
        <td>344162</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:31:16</td>
        <td>2013-10-02 20:34:03</td>
        <td>344162</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:31:26</td>
        <td>2013-10-02 20:34:03</td>
        <td>344162</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:31:32</td>
        <td>2013-10-02 20:34:03</td>
        <td>344162</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:31:43</td>
        <td>2013-10-02 20:34:03</td>
        <td>344162</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:31:50</td>
        <td>2013-10-02 20:34:03</td>
        <td>344162</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:00</td>
        <td>2013-10-02 20:34:03</td>
        <td>344162</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:06</td>
        <td>2013-10-02 20:34:03</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:11</td>
        <td>2013-10-02 20:34:04</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:15</td>
        <td>2013-10-02 20:34:04</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:16</td>
        <td>2013-10-02 20:34:04</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:16</td>
        <td>2013-10-02 20:34:04</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:16</td>
        <td>2013-10-02 20:34:04</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:17</td>
        <td>2013-10-02 20:34:04</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:18</td>
        <td>2013-10-02 20:34:04</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:19</td>
        <td>2013-10-02 20:34:04</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:19</td>
        <td>2013-10-02 20:34:04</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:20</td>
        <td>2013-10-02 20:34:04</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:21</td>
        <td>2013-10-02 20:34:04</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:21</td>
        <td>2013-10-02 20:34:04</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:22</td>
        <td>2013-10-02 20:34:04</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:22</td>
        <td>2013-10-02 20:34:04</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:23</td>
        <td>2013-10-02 20:34:04</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:23</td>
        <td>2013-10-02 20:34:04</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:23</td>
        <td>2013-10-02 20:34:04</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:24</td>
        <td>2013-10-02 20:34:04</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:24</td>
        <td>2013-10-02 20:34:04</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:24</td>
        <td>2013-10-02 20:34:04</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:25</td>
        <td>2013-10-02 20:34:04</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:25</td>
        <td>2013-10-02 20:34:04</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:26</td>
        <td>2013-10-02 20:34:04</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:27</td>
        <td>2013-10-02 20:34:04</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:27</td>
        <td>2013-10-02 20:34:04</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:27</td>
        <td>2013-10-02 20:34:04</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:27</td>
        <td>2013-10-02 20:34:04</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:28</td>
        <td>2013-10-02 20:34:04</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:29</td>
        <td>2013-10-02 20:34:04</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:29</td>
        <td>2013-10-02 20:34:04</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:30</td>
        <td>2013-10-02 20:34:04</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:30</td>
        <td>2013-10-02 20:34:04</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:30</td>
        <td>2013-10-02 20:34:04</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:30</td>
        <td>2013-10-02 20:34:04</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:31</td>
        <td>2013-10-02 20:34:04</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:31</td>
        <td>2013-10-02 20:34:04</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:31</td>
        <td>2013-10-02 20:34:04</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:31</td>
        <td>2013-10-02 20:34:04</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:32</td>
        <td>2013-10-02 20:34:04</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:32</td>
        <td>2013-10-02 20:34:04</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:33</td>
        <td>2013-10-02 20:34:04</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:33</td>
        <td>2013-10-02 20:34:04</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:33</td>
        <td>2013-10-02 20:34:04</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:33</td>
        <td>2013-10-02 20:34:04</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:33</td>
        <td>2013-10-02 20:34:04</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:33</td>
        <td>2013-10-02 20:34:04</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:33</td>
        <td>2013-10-02 20:34:04</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:35</td>
        <td>2013-10-02 20:34:04</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:35</td>
        <td>2013-10-02 20:34:04</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:35</td>
        <td>2013-10-02 20:34:04</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:36</td>
        <td>2013-10-02 20:34:04</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:36</td>
        <td>2013-10-02 20:34:04</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:36</td>
        <td>2013-10-02 20:34:04</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:36</td>
        <td>2013-10-02 20:34:04</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:36</td>
        <td>2013-10-02 20:34:04</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:37</td>
        <td>2013-10-02 20:34:04</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:37</td>
        <td>2013-10-02 20:34:04</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:37</td>
        <td>2013-10-02 20:34:04</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:37</td>
        <td>2013-10-02 20:34:04</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:38</td>
        <td>2013-10-02 20:34:04</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:38</td>
        <td>2013-10-02 20:34:04</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:39</td>
        <td>2013-10-02 20:34:04</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:39</td>
        <td>2013-10-02 20:34:04</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:40</td>
        <td>2013-10-02 20:34:04</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:40</td>
        <td>2013-10-02 20:34:04</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:40</td>
        <td>2013-10-02 20:34:04</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:40</td>
        <td>2013-10-02 20:34:05</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:40</td>
        <td>2013-10-02 20:34:05</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:41</td>
        <td>2013-10-02 20:34:05</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:41</td>
        <td>2013-10-02 20:34:05</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:41</td>
        <td>2013-10-02 20:34:05</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:41</td>
        <td>2013-10-02 20:34:05</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:42</td>
        <td>2013-10-02 20:34:05</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:43</td>
        <td>2013-10-02 20:34:05</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:47</td>
        <td>2013-10-02 20:34:05</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:52</td>
        <td>2013-10-02 20:34:05</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:32:56</td>
        <td>2013-10-02 20:34:05</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:33:02</td>
        <td>2013-10-02 20:34:05</td>
        <td>344161</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:33:07</td>
        <td>2013-10-02 20:34:05</td>
        <td>344160</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:33:10</td>
        <td>2013-10-02 20:34:05</td>
        <td>344160</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:33:31</td>
        <td>2013-10-02 20:34:05</td>
        <td>344160</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:33:44</td>
        <td>2013-10-02 20:34:05</td>
        <td>344160</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:33:49</td>
        <td>2013-10-02 20:34:05</td>
        <td>344160</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:33:51</td>
        <td>2013-10-02 20:34:05</td>
        <td>344160</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:33:57</td>
        <td>2013-10-02 20:34:05</td>
        <td>344160</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:34:02</td>
        <td>2013-10-02 20:34:05</td>
        <td>344160</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:34:09</td>
        <td>2013-10-02 20:34:05</td>
        <td>344159</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:34:17</td>
        <td>2013-10-02 20:34:05</td>
        <td>344159</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:34:20</td>
        <td>2013-10-02 20:34:05</td>
        <td>344159</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:34:23</td>
        <td>2013-10-02 20:34:05</td>
        <td>344159</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:34:28</td>
        <td>2013-10-02 20:34:05</td>
        <td>344159</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:34:33</td>
        <td>2013-10-02 20:34:05</td>
        <td>344159</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:34:38</td>
        <td>2013-10-02 20:34:05</td>
        <td>344159</td>
    </tr>
    <tr>
        <td>fd27c032-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:34:43</td>
        <td>2013-10-02 20:34:05</td>
        <td>344159</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:35:29</td>
        <td>2013-02-05 20:35:47</td>
        <td>0</td>
    </tr>
    <tr>
        <td>fd27bf60-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:36:03</td>
        <td>2013-10-02 20:34:05</td>
        <td>344158</td>
    </tr>
    <tr>
        <td>fd27bf60-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:36:09</td>
        <td>2013-10-02 20:34:05</td>
        <td>344157</td>
    </tr>
    <tr>
        <td>fd27bf60-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:36:27</td>
        <td>2013-10-02 20:34:05</td>
        <td>344157</td>
    </tr>
    <tr>
        <td>fd27bf60-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:36:35</td>
        <td>2013-10-02 20:34:05</td>
        <td>344157</td>
    </tr>
    <tr>
        <td>fd27bf60-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:36:39</td>
        <td>2013-10-02 20:34:05</td>
        <td>344157</td>
    </tr>
    <tr>
        <td>fd27bf60-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:36:42</td>
        <td>2013-10-02 20:34:05</td>
        <td>344157</td>
    </tr>
    <tr>
        <td>fd27bf60-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:36:45</td>
        <td>2013-10-02 20:34:05</td>
        <td>344157</td>
    </tr>
    <tr>
        <td>fd27bf60-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:36:56</td>
        <td>2013-10-02 20:34:05</td>
        <td>344157</td>
    </tr>
    <tr>
        <td>fd27bf60-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:36:59</td>
        <td>2013-10-02 20:34:05</td>
        <td>344157</td>
    </tr>
    <tr>
        <td>fd27bf60-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:37:02</td>
        <td>2013-10-02 20:34:05</td>
        <td>344157</td>
    </tr>
    <tr>
        <td>fd27bf60-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:37:07</td>
        <td>2013-10-02 20:34:05</td>
        <td>344156</td>
    </tr>
    <tr>
        <td>fd27bf60-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:37:14</td>
        <td>2013-10-02 20:34:05</td>
        <td>344156</td>
    </tr>
    <tr>
        <td>fd27bf60-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:37:19</td>
        <td>2013-10-02 20:34:05</td>
        <td>344156</td>
    </tr>
    <tr>
        <td>fd27bf60-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:37:27</td>
        <td>2013-10-02 20:34:05</td>
        <td>344156</td>
    </tr>
    <tr>
        <td>fd27bf60-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:37:35</td>
        <td>2013-10-02 20:34:05</td>
        <td>344156</td>
    </tr>
    <tr>
        <td>fd27bf60-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:38:15</td>
        <td>2013-10-02 20:34:05</td>
        <td>344155</td>
    </tr>
    <tr>
        <td>fd27bf60-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:38:24</td>
        <td>2013-10-02 20:34:05</td>
        <td>344155</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:38:26</td>
        <td>2013-02-05 20:47:00</td>
        <td>8</td>
    </tr>
    <tr>
        <td>fd27bf60-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:38:33</td>
        <td>2013-10-02 20:34:05</td>
        <td>344155</td>
    </tr>
    <tr>
        <td>fd27bf60-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:38:36</td>
        <td>2013-10-02 20:34:05</td>
        <td>344155</td>
    </tr>
    <tr>
        <td>fd27bf60-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:38:40</td>
        <td>2013-10-02 20:34:05</td>
        <td>344155</td>
    </tr>
    <tr>
        <td>fd27bf60-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:38:42</td>
        <td>2013-10-02 20:34:05</td>
        <td>344155</td>
    </tr>
    <tr>
        <td>fd27bf60-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:38:44</td>
        <td>2013-10-02 20:34:05</td>
        <td>344155</td>
    </tr>
    <tr>
        <td>fd27bf60-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:38:46</td>
        <td>2013-10-02 20:34:05</td>
        <td>344155</td>
    </tr>
    <tr>
        <td>fd27bf60-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:38:50</td>
        <td>2013-10-02 20:34:05</td>
        <td>344155</td>
    </tr>
    <tr>
        <td>fd27bf60-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:39:02</td>
        <td>2013-10-02 20:34:05</td>
        <td>344155</td>
    </tr>
    <tr>
        <td>fd27bf60-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:39:07</td>
        <td>2013-10-02 20:34:05</td>
        <td>344154</td>
    </tr>
    <tr>
        <td>fd27bf60-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:39:19</td>
        <td>2013-10-02 20:34:05</td>
        <td>344154</td>
    </tr>
    <tr>
        <td>fd27bf60-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:39:33</td>
        <td>2013-10-02 20:34:05</td>
        <td>344154</td>
    </tr>
    <tr>
        <td>fd27bf60-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:39:37</td>
        <td>2013-10-02 20:34:05</td>
        <td>344154</td>
    </tr>
    <tr>
        <td>fd27bf60-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:39:41</td>
        <td>2013-10-02 20:34:05</td>
        <td>344154</td>
    </tr>
    <tr>
        <td>fd27bf60-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:39:45</td>
        <td>2013-10-02 20:34:05</td>
        <td>344154</td>
    </tr>
    <tr>
        <td>fd27bf60-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:40:04</td>
        <td>2013-10-02 20:34:05</td>
        <td>344154</td>
    </tr>
    <tr>
        <td>fd27bf60-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:40:07</td>
        <td>2013-10-02 20:34:05</td>
        <td>344153</td>
    </tr>
    <tr>
        <td>fd27bf60-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:40:09</td>
        <td>2013-10-02 20:34:05</td>
        <td>344153</td>
    </tr>
    <tr>
        <td>fd27bf60-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:40:13</td>
        <td>2013-10-02 20:34:05</td>
        <td>344153</td>
    </tr>
    <tr>
        <td>fd27bf60-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:40:20</td>
        <td>2013-10-02 20:34:05</td>
        <td>344153</td>
    </tr>
    <tr>
        <td>fd27bf60-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:40:24</td>
        <td>2013-10-02 20:34:05</td>
        <td>344153</td>
    </tr>
    <tr>
        <td>fd27bf60-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:40:28</td>
        <td>2013-10-02 20:34:06</td>
        <td>344153</td>
    </tr>
    <tr>
        <td>fd27bf60-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:40:30</td>
        <td>2013-10-02 20:34:06</td>
        <td>344153</td>
    </tr>
    <tr>
        <td>fd27bf60-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:40:34</td>
        <td>2013-10-02 20:34:06</td>
        <td>344153</td>
    </tr>
    <tr>
        <td>fd27bf60-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-05 20:40:38</td>
        <td>2013-10-02 20:34:06</td>
        <td>344153</td>
    </tr>
</table>
<span style="font-style:italic;text-align:center;">2000 rows, truncated to displaylimit of 1000</span>



If you explore your output you will find that some of your calculated durations appear to be "0." In some cases, you will see many entries from the same Dog_ID with the same start time and end time.  That should be impossible.  These types of entries probably represent tests run by the Dognition team rather than real customer data.  In other cases, though, a "0" is entered in the Duration column even though the start_time and end_time are different.  This is because we instructed the function to output the time difference in minutes; unless you change your settings, it will output "0" for any time differences less than the integer 1.  If you change your function to output the time difference in seconds, the duration in most of these columns will have a non-zero number.  

**Question 8: What is the average amount of time it took customers to complete all of the tests in the exam_answers table, if you do not exclude any data (the answer will be approximately 587 minutes)?**


```python
%%sql
SELECT AVG(TIMESTAMPDIFF(minute,start_time, end_time)) AS Average_Time
FROM exam_answers;
```

    1 rows affected.





<table>
    <tr>
        <th>Average_Time</th>
    </tr>
    <tr>
        <td>586.9041</td>
    </tr>
</table>



**Question 9: What is the average amount of time it took customers to complete the "Treat Warm-Up" test, according to the exam_answers table (about 165 minutes, if no data is excluded)?**


```python
%%sql
SELECT AVG(TIMESTAMPDIFF(minute,start_time,end_time)) AS AvgDuration 
FROM exam_answers
WHERE test_name="Treat Warm-Up";
```

    1 rows affected.





<table>
    <tr>
        <th>AvgDuration</th>
    </tr>
    <tr>
        <td>164.9176</td>
    </tr>
</table>



**Question 10: How many possible test names are there in the exam_answers table?**


```python
%%sql
SELECT COUNT(DISTINCT test_name)
FROM exam_answers;
```

    1 rows affected.





<table>
    <tr>
        <th>COUNT(DISTINCT test_name)</th>
    </tr>
    <tr>
        <td>67</td>
    </tr>
</table>



You should have discovered that the exam_answers table has many more test names than the completed_tests table.  It turns out that this table has information about experimental tests that Dognition has not yet made available to its customers. 
   

**Question 11: What is the minimum and maximum value in the Duration column of your query that included the data from the entire table?**


```python
%%sql
SELECT MIN(TIMESTAMPDIFF(minute,start_time,end_time)) AS MinTIME, MAX(TIMESTAMPDIFF(minute,start_time,end_time)) AS MaxTIME
FROM exam_answers;
```

    1 rows affected.





<table>
    <tr>
        <th>MinTIME</th>
        <th>MaxTIME</th>
    </tr>
    <tr>
        <td>-187</td>
        <td>1036673</td>
    </tr>
</table>



The minimum Duration value is *negative*!  The end_times entered in rows with negative Duration values are earlier than the start_times.  Unless Dognition has created a time machine, that's impossible and these entries must be mistakes.  

**Question 12: How many of these negative Duration entries are there? (the answer should be 620)**


```python
%%sql
SELECT COUNT(TIMESTAMPDIFF(minute,start_time,end_time)) AS CountMin
FROM exam_answers
WHERE TIMESTAMPDIFF(minute,start_time,end_time) < 0;
```

    1 rows affected.





<table>
    <tr>
        <th>CountMin</th>
    </tr>
    <tr>
        <td>620</td>
    </tr>
</table>



**Question 13: How would you query all the columns of all the rows that have negative durations so that you could examine whether they share any features that might give you clues about what caused the entry mistake?**


```python
%%sql
SELECT Dog_Guid, start_time, end_time, TIMESTAMPDIFF(minute, start_time, end_time) AS Duration
FROM exam_answers
WHERE TIMESTAMPDIFF(minute,start_time,end_time) < 0;
```

    620 rows affected.





<table>
    <tr>
        <th>Dog_Guid</th>
        <th>start_time</th>
        <th>end_time</th>
        <th>Duration</th>
    </tr>
    <tr>
        <td>fd3fe18a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-17 20:35:43</td>
        <td>2013-02-17 20:34:43</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fe50e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-18 04:25:19</td>
        <td>2013-02-18 04:24:18</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fe5ea-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-18 07:44:09</td>
        <td>2013-02-18 07:43:09</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fe5ea-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-18 07:46:14</td>
        <td>2013-02-18 07:45:13</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fe5ea-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-18 07:47:07</td>
        <td>2013-02-18 07:46:06</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fe5ea-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-18 07:50:07</td>
        <td>2013-02-18 07:49:07</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fbd7c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-18 13:23:25</td>
        <td>2013-02-18 13:22:23</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fbd7c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-18 13:23:31</td>
        <td>2013-02-18 13:22:28</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fbd7c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-18 13:25:15</td>
        <td>2013-02-18 13:24:14</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fbd7c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-18 13:25:30</td>
        <td>2013-02-18 13:24:27</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fbd7c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-18 13:27:55</td>
        <td>2013-02-18 13:26:53</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3d4952-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-18 15:49:19</td>
        <td>2013-02-18 15:48:18</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3d4952-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-18 15:50:15</td>
        <td>2013-02-18 15:49:14</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3d4952-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-18 15:53:47</td>
        <td>2013-02-18 15:52:46</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3d4952-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-18 15:54:40</td>
        <td>2013-02-18 15:53:39</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fbd7c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-18 17:02:05</td>
        <td>2013-02-18 17:01:04</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fbd7c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-18 17:03:59</td>
        <td>2013-02-18 17:02:56</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fbd7c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-18 17:04:04</td>
        <td>2013-02-18 17:03:02</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fbd7c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-18 17:04:11</td>
        <td>2013-02-18 17:03:08</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fe7b6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-18 17:58:40</td>
        <td>2013-02-18 17:57:39</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fe7b6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-18 17:59:08</td>
        <td>2013-02-18 17:58:07</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fe7b6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-18 17:59:18</td>
        <td>2013-02-18 17:58:17</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fe7b6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-18 17:59:41</td>
        <td>2013-02-18 17:58:41</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fe7b6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-18 18:02:34</td>
        <td>2013-02-18 18:01:33</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fe842-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-18 18:10:22</td>
        <td>2013-02-18 18:09:19</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fe842-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-18 18:11:45</td>
        <td>2013-02-18 18:10:43</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fe842-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-18 18:11:51</td>
        <td>2013-02-18 18:10:50</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fe842-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-18 18:13:34</td>
        <td>2013-02-18 18:12:34</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3d06e0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-18 19:23:40</td>
        <td>2013-02-18 19:22:39</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd27cb72-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-18 19:53:27</td>
        <td>2013-02-18 19:52:26</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fea9a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-18 20:57:51</td>
        <td>2013-02-18 20:56:51</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fea9a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-18 20:57:52</td>
        <td>2013-02-18 20:56:52</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fea9a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-18 20:57:54</td>
        <td>2013-02-18 20:56:51</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fea9a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-18 20:59:37</td>
        <td>2013-02-18 20:58:33</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fea9a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-18 20:59:43</td>
        <td>2013-02-18 20:58:39</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fea9a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-18 20:59:47</td>
        <td>2013-02-18 20:58:43</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fea9a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-18 20:59:52</td>
        <td>2013-02-18 20:58:48</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fea9a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-18 20:59:56</td>
        <td>2013-02-18 20:58:52</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fea9a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-18 21:00:00</td>
        <td>2013-02-18 20:58:55</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fea9a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-18 21:00:02</td>
        <td>2013-02-18 20:58:58</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fea9a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-18 21:00:04</td>
        <td>2013-02-18 20:59:01</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fea9a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-18 21:00:09</td>
        <td>2013-02-18 20:59:07</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fea9a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-18 21:00:14</td>
        <td>2013-02-18 20:59:10</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fea9a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-18 21:00:20</td>
        <td>2013-02-18 20:59:15</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fea9a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-18 21:00:38</td>
        <td>2013-02-18 20:59:34</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fea9a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-18 21:00:42</td>
        <td>2013-02-18 20:59:41</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fea9a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-18 21:00:44</td>
        <td>2013-02-18 20:59:40</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fea9a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-18 21:00:46</td>
        <td>2013-02-18 20:59:45</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fea9a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-18 21:00:50</td>
        <td>2013-02-18 20:59:50</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fea9a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-18 21:00:54</td>
        <td>2013-02-18 20:59:53</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fea9a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-18 21:00:57</td>
        <td>2013-02-18 20:59:56</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fea9a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-18 21:38:03</td>
        <td>2013-02-18 21:37:00</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3feb30-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-18 22:04:23</td>
        <td>2013-02-18 22:03:23</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fdfc8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-18 22:27:05</td>
        <td>2013-02-18 22:26:04</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fe96e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-18 23:05:23</td>
        <td>2013-02-18 23:04:22</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fefd6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 00:17:09</td>
        <td>2013-02-19 00:16:08</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fefd6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 00:17:30</td>
        <td>2013-02-19 00:16:30</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fefd6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 00:18:09</td>
        <td>2013-02-19 00:17:08</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fefd6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 00:18:43</td>
        <td>2013-02-19 00:17:43</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fefd6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 00:21:49</td>
        <td>2013-02-19 00:20:48</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fefd6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 00:22:02</td>
        <td>2013-02-19 00:21:02</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fd5aa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 00:35:11</td>
        <td>2013-02-19 00:34:11</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ff18e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 01:30:53</td>
        <td>2013-02-19 01:29:52</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ff18e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 01:32:47</td>
        <td>2013-02-19 01:31:47</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ff18e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 01:34:22</td>
        <td>2013-02-19 01:33:22</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ff18e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 01:34:45</td>
        <td>2013-02-19 01:33:41</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ff18e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 01:35:09</td>
        <td>2013-02-19 01:34:07</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ff18e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 01:36:09</td>
        <td>2013-02-19 01:35:09</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ff22e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 01:39:42</td>
        <td>2013-02-19 01:38:42</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ff22e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 01:41:51</td>
        <td>2013-02-19 01:40:49</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ff22e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 01:42:05</td>
        <td>2013-02-19 01:41:03</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ff22e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 01:42:21</td>
        <td>2013-02-19 01:41:21</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ff22e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 01:43:16</td>
        <td>2013-02-19 01:42:14</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ff22e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 01:43:29</td>
        <td>2013-02-19 01:42:26</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ff22e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 01:44:26</td>
        <td>2013-02-19 01:43:26</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ff22e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 01:45:09</td>
        <td>2013-02-19 01:44:09</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ff22e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 01:45:16</td>
        <td>2013-02-19 01:44:16</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fed7e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 03:03:25</td>
        <td>2013-02-19 03:02:23</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fed7e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 03:07:34</td>
        <td>2013-02-19 03:06:32</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3febc6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 03:12:54</td>
        <td>2013-02-19 03:11:52</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ff882-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 03:21:01</td>
        <td>2013-02-19 03:19:59</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ff882-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 03:21:30</td>
        <td>2013-02-19 03:20:27</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ff882-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 03:21:52</td>
        <td>2013-02-19 03:20:50</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ff882-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 03:23:30</td>
        <td>2013-02-19 03:22:28</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ff882-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 03:23:51</td>
        <td>2013-02-19 03:22:51</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3feeb4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 03:23:55</td>
        <td>2013-02-19 03:22:55</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ff882-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 03:24:14</td>
        <td>2013-02-19 03:23:13</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ff882-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 03:25:01</td>
        <td>2013-02-19 03:24:01</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ff882-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 03:25:16</td>
        <td>2013-02-19 03:24:15</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ff5a8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 03:30:53</td>
        <td>2013-02-19 03:29:53</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ff5a8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 03:32:59</td>
        <td>2013-02-19 03:31:59</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ff5a8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 03:33:42</td>
        <td>2013-02-19 03:32:39</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ff5a8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 03:34:45</td>
        <td>2013-02-19 03:33:42</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ff5a8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 03:36:50</td>
        <td>2013-02-19 03:35:48</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ff5a8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 03:37:06</td>
        <td>2013-02-19 03:36:04</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ff5a8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 03:37:20</td>
        <td>2013-02-19 03:36:17</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ff5a8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 03:37:55</td>
        <td>2013-02-19 03:36:53</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd27c74e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 05:24:23</td>
        <td>2013-02-19 05:23:22</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3d1202-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 05:38:15</td>
        <td>2013-02-19 05:37:14</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3d1202-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 05:39:52</td>
        <td>2013-02-19 05:38:52</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3d343a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 11:04:27</td>
        <td>2013-02-19 11:03:25</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ffa30-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 13:27:06</td>
        <td>2013-02-19 13:26:06</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ffa30-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 13:29:15</td>
        <td>2013-02-19 13:28:14</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ffa30-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 13:29:38</td>
        <td>2013-02-19 13:28:38</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ffa30-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 13:29:54</td>
        <td>2013-02-19 13:28:54</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ffa30-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 13:30:13</td>
        <td>2013-02-19 13:29:11</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ffa30-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 13:30:38</td>
        <td>2013-02-19 13:29:36</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ffa30-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 13:31:10</td>
        <td>2013-02-19 13:30:09</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ffa30-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 13:31:20</td>
        <td>2013-02-19 13:30:20</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fdc3a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 14:11:45</td>
        <td>2013-02-19 14:10:45</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fdc3a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 14:13:50</td>
        <td>2013-02-19 14:12:50</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fdc3a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 14:24:23</td>
        <td>2013-02-19 14:23:23</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fbe44-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 17:27:28</td>
        <td>2013-02-19 17:26:25</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fbe44-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 17:32:48</td>
        <td>2013-02-19 17:31:43</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fbe44-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 17:33:23</td>
        <td>2013-02-19 17:32:22</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ffbf2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 18:02:44</td>
        <td>2013-02-19 18:01:44</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ffbf2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 18:03:26</td>
        <td>2013-02-19 18:02:25</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ffbf2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 18:04:01</td>
        <td>2013-02-19 18:03:00</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ffbf2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 18:04:31</td>
        <td>2013-02-19 18:03:30</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ffbf2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 18:06:14</td>
        <td>2013-02-19 18:05:12</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ffbf2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 18:08:46</td>
        <td>2013-02-19 18:07:45</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ffbf2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 18:10:30</td>
        <td>2013-02-19 18:09:28</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ffc7e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 18:42:31</td>
        <td>2013-02-19 18:41:31</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ffc7e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 18:45:02</td>
        <td>2013-02-19 18:43:58</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ffc7e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 18:45:24</td>
        <td>2013-02-19 18:44:20</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ffc7e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 18:45:39</td>
        <td>2013-02-19 18:44:37</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ffc7e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 18:46:12</td>
        <td>2013-02-19 18:45:09</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ffc7e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 18:46:30</td>
        <td>2013-02-19 18:45:29</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ffc7e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 18:47:14</td>
        <td>2013-02-19 18:46:13</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ffc7e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 18:47:19</td>
        <td>2013-02-19 18:46:15</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ffc7e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 18:47:38</td>
        <td>2013-02-19 18:46:37</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ffc7e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 18:47:43</td>
        <td>2013-02-19 18:46:39</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ffc7e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 18:47:57</td>
        <td>2013-02-19 18:46:54</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3d343a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 19:38:27</td>
        <td>2013-02-19 19:37:26</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3d343a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 19:51:10</td>
        <td>2013-02-19 19:50:10</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3d343a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 19:51:40</td>
        <td>2013-02-19 19:50:35</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3d343a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 19:54:21</td>
        <td>2013-02-19 19:53:21</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3d343a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 19:58:49</td>
        <td>2013-02-19 19:57:49</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3d343a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 19:59:03</td>
        <td>2013-02-19 19:58:03</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ffda0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 20:09:47</td>
        <td>2013-02-19 20:08:47</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ffda0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 20:12:04</td>
        <td>2013-02-19 20:11:04</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ffda0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 20:13:56</td>
        <td>2013-02-19 20:12:53</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ffda0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 20:18:59</td>
        <td>2013-02-19 20:17:58</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ffda0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 20:19:22</td>
        <td>2013-02-19 20:18:21</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ffda0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 20:20:07</td>
        <td>2013-02-19 20:19:07</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ffd14-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 20:23:27</td>
        <td>2013-02-19 20:22:26</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ffd14-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 20:24:05</td>
        <td>2013-02-19 20:23:05</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ffd14-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 20:24:52</td>
        <td>2013-02-19 20:23:50</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ffd14-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 20:25:26</td>
        <td>2013-02-19 20:24:25</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ffd14-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 20:26:15</td>
        <td>2013-02-19 20:25:15</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ffd14-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 20:27:01</td>
        <td>2013-02-19 20:26:00</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ffd14-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 20:27:35</td>
        <td>2013-02-19 20:26:34</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ffd14-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 20:27:46</td>
        <td>2013-02-19 20:26:43</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd4003b8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 23:11:00</td>
        <td>2013-02-19 23:09:56</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd4003b8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 23:13:37</td>
        <td>2013-02-19 23:12:37</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd4003b8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 23:15:19</td>
        <td>2013-02-19 23:14:19</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd4003b8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 23:16:59</td>
        <td>2013-02-19 23:15:59</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd4003b8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 23:17:23</td>
        <td>2013-02-19 23:16:23</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd4003b8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 23:20:14</td>
        <td>2013-02-19 23:19:11</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd400458-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 23:22:16</td>
        <td>2013-02-19 23:21:12</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd400458-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 23:22:39</td>
        <td>2013-02-19 23:21:35</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd400458-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 23:23:55</td>
        <td>2013-02-19 23:22:54</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd400458-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 23:24:24</td>
        <td>2013-02-19 23:23:21</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd400458-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 23:25:49</td>
        <td>2013-02-19 23:24:48</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd400458-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 23:26:59</td>
        <td>2013-02-19 23:25:59</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd400458-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 23:27:20</td>
        <td>2013-02-19 23:26:18</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd400458-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 23:28:39</td>
        <td>2013-02-19 23:27:38</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd400458-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 23:28:53</td>
        <td>2013-02-19 23:27:52</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3d34d0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-19 23:54:08</td>
        <td>2013-02-19 23:53:07</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3d06e0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 00:28:47</td>
        <td>2013-02-20 00:27:45</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3d06e0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 00:36:25</td>
        <td>2013-02-20 00:35:24</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3d06e0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 00:37:26</td>
        <td>2013-02-20 00:36:24</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3d440c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 01:02:38</td>
        <td>2013-02-20 01:01:34</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3d440c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 01:07:50</td>
        <td>2013-02-20 01:06:46</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ffb5c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 01:10:21</td>
        <td>2013-02-20 01:09:19</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ffb5c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 01:12:26</td>
        <td>2013-02-20 01:11:25</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ffb5c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 01:14:00</td>
        <td>2013-02-20 01:12:59</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ffb5c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 01:14:13</td>
        <td>2013-02-20 01:13:13</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ffb5c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 01:14:33</td>
        <td>2013-02-20 01:13:32</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ffb5c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 01:14:43</td>
        <td>2013-02-20 01:13:40</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ffb5c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 01:15:24</td>
        <td>2013-02-20 01:14:22</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ffb5c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 01:16:04</td>
        <td>2013-02-20 01:15:03</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ffb5c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 01:16:14</td>
        <td>2013-02-20 01:15:14</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ffb5c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 01:16:48</td>
        <td>2013-02-20 01:15:42</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ffb5c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 01:17:00</td>
        <td>2013-02-20 01:16:00</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ffb5c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 01:17:19</td>
        <td>2013-02-20 01:16:19</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ffb5c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 01:17:32</td>
        <td>2013-02-20 01:16:32</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd400584-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 01:55:00</td>
        <td>2013-02-20 01:53:58</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd400584-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 01:57:26</td>
        <td>2013-02-20 01:56:25</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd400584-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 02:06:44</td>
        <td>2013-02-20 02:05:42</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd400584-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 02:10:23</td>
        <td>2013-02-20 02:09:23</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd400584-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 02:11:12</td>
        <td>2013-02-20 02:10:10</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd400610-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 03:37:36</td>
        <td>2013-02-20 03:36:35</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd400610-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 03:40:29</td>
        <td>2013-02-20 03:39:29</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd400610-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 03:40:48</td>
        <td>2013-02-20 03:39:46</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd400610-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 03:41:11</td>
        <td>2013-02-20 03:40:10</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd400610-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 03:41:41</td>
        <td>2013-02-20 03:40:41</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd400610-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 03:42:03</td>
        <td>2013-02-20 03:41:00</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd400610-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 03:42:46</td>
        <td>2013-02-20 03:41:46</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd400610-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 03:43:49</td>
        <td>2013-02-20 03:42:47</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd400610-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 03:44:05</td>
        <td>2013-02-20 03:43:03</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd400610-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 03:44:48</td>
        <td>2013-02-20 03:43:48</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd4006a6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 04:14:07</td>
        <td>2013-02-20 04:13:07</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd4006a6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 04:14:16</td>
        <td>2013-02-20 04:13:13</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd4006a6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 04:16:53</td>
        <td>2013-02-20 04:15:52</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd4006a6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 04:21:02</td>
        <td>2013-02-20 04:19:57</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd4006a6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 04:24:43</td>
        <td>2013-02-20 04:23:43</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd4007c8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 14:38:20</td>
        <td>2013-02-20 14:37:16</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd4007c8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 14:39:30</td>
        <td>2013-02-20 14:38:28</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd4007c8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 14:40:17</td>
        <td>2013-02-20 14:39:10</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd4007c8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 14:40:37</td>
        <td>2013-02-20 14:39:30</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd4007c8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 14:42:27</td>
        <td>2013-02-20 14:41:24</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd4007c8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 14:42:42</td>
        <td>2013-02-20 14:41:39</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd4007c8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 14:43:06</td>
        <td>2013-02-20 14:42:04</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd4007c8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 14:47:09</td>
        <td>2013-02-20 14:46:06</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd4007c8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 14:47:25</td>
        <td>2013-02-20 14:46:21</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd40085e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 14:53:48</td>
        <td>2013-02-20 14:52:44</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd40085e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 14:54:09</td>
        <td>2013-02-20 14:53:04</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd40085e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 14:54:46</td>
        <td>2013-02-20 14:53:41</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd40085e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 14:55:36</td>
        <td>2013-02-20 14:54:31</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd40085e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 14:56:51</td>
        <td>2013-02-20 14:55:46</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3d1b44-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 15:17:25</td>
        <td>2013-02-20 15:16:16</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3d1b44-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 15:17:29</td>
        <td>2013-02-20 15:16:20</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3d1b44-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 15:17:33</td>
        <td>2013-02-20 15:16:23</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3d1b44-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 15:17:36</td>
        <td>2013-02-20 15:16:32</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3d1b44-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 15:17:44</td>
        <td>2013-02-20 15:16:35</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3d1b44-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 15:17:47</td>
        <td>2013-02-20 15:16:37</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3d1b44-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 15:17:50</td>
        <td>2013-02-20 15:16:40</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3d1b44-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 15:17:53</td>
        <td>2013-02-20 15:16:43</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3d1b44-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 15:17:56</td>
        <td>2013-02-20 15:16:47</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3d1b44-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 15:17:59</td>
        <td>2013-02-20 15:16:51</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3d1b44-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 15:18:04</td>
        <td>2013-02-20 15:16:56</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3d1b44-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 15:18:09</td>
        <td>2013-02-20 15:16:59</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3d1b44-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 15:18:13</td>
        <td>2013-02-20 15:17:03</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3d1b44-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 15:18:16</td>
        <td>2013-02-20 15:17:06</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3d1b44-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 15:18:22</td>
        <td>2013-02-20 15:17:13</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3d1b44-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 15:18:25</td>
        <td>2013-02-20 15:17:16</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3d1b44-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 15:18:28</td>
        <td>2013-02-20 15:17:18</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3d1b44-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 15:18:31</td>
        <td>2013-02-20 15:17:21</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3d10cc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 15:26:10</td>
        <td>2013-02-20 15:25:09</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3d10cc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 15:28:41</td>
        <td>2013-02-20 15:27:36</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3d10cc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 15:30:06</td>
        <td>2013-02-20 15:29:04</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3d10cc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 15:30:19</td>
        <td>2013-02-20 15:29:17</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3d10cc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 15:33:19</td>
        <td>2013-02-20 15:32:18</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd2808b2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 15:35:16</td>
        <td>2013-02-20 15:34:12</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3d10cc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 15:35:22</td>
        <td>2013-02-20 15:34:20</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3d10cc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 15:35:39</td>
        <td>2013-02-20 15:34:37</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd2808b2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 15:36:07</td>
        <td>2013-02-20 15:35:03</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd2808b2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 15:39:45</td>
        <td>2013-02-20 15:38:39</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd2808b2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 15:42:05</td>
        <td>2013-02-20 15:41:05</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3f9c2a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 15:48:34</td>
        <td>2013-02-20 15:47:32</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3f9c2a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 15:57:04</td>
        <td>2013-02-20 15:56:00</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3f9c2a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 15:58:25</td>
        <td>2013-02-20 15:57:17</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fa0b2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 16:04:02</td>
        <td>2013-02-20 16:02:55</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fa0b2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 16:08:22</td>
        <td>2013-02-20 16:07:18</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd400b4c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 16:47:16</td>
        <td>2013-02-20 16:46:12</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd400bec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 16:48:21</td>
        <td>2013-02-20 16:47:18</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd400bec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 16:48:39</td>
        <td>2013-02-20 16:47:38</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd400b4c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 16:48:47</td>
        <td>2013-02-20 16:47:44</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd400b4c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 16:50:52</td>
        <td>2013-02-20 16:49:51</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ffbf2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 16:52:51</td>
        <td>2013-02-20 16:51:49</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ffbf2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 16:53:25</td>
        <td>2013-02-20 16:52:22</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ffbf2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 16:57:43</td>
        <td>2013-02-20 16:56:36</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ffbf2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 17:06:25</td>
        <td>2013-02-20 17:05:17</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ffbf2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 17:07:15</td>
        <td>2013-02-20 17:06:11</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd400d04-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 19:29:51</td>
        <td>2013-02-20 19:28:44</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd400d04-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 19:30:48</td>
        <td>2013-02-20 19:29:41</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd400d04-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 19:31:52</td>
        <td>2013-02-20 19:30:51</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd400fca-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 20:09:34</td>
        <td>2013-02-20 20:08:34</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd400fca-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 20:09:58</td>
        <td>2013-02-20 20:08:56</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd400fca-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 20:10:22</td>
        <td>2013-02-20 20:09:22</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fd5aa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 20:13:53</td>
        <td>2013-02-20 20:12:52</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd400fca-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 20:17:40</td>
        <td>2013-02-20 20:16:37</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd400fca-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 20:18:11</td>
        <td>2013-02-20 20:17:06</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd4014a2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 20:25:45</td>
        <td>2013-02-20 20:24:43</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd4014a2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 20:26:32</td>
        <td>2013-02-20 20:25:30</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd4014a2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 20:32:07</td>
        <td>2013-02-20 20:31:00</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd4014a2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 20:32:36</td>
        <td>2013-02-20 20:31:35</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd4014a2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 20:33:14</td>
        <td>2013-02-20 20:32:09</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd4014a2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 20:33:24</td>
        <td>2013-02-20 20:32:23</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd4014a2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 20:33:47</td>
        <td>2013-02-20 20:32:46</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fd5aa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 20:37:38</td>
        <td>2013-02-20 20:36:35</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fd5aa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 20:40:44</td>
        <td>2013-02-20 20:39:41</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fe842-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 20:42:54</td>
        <td>2013-02-20 20:41:52</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3d4b8c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 20:59:11</td>
        <td>2013-02-20 20:58:10</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fd5aa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 21:04:12</td>
        <td>2013-02-20 21:03:10</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fd5aa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 21:04:23</td>
        <td>2013-02-20 21:03:19</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3d4b8c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 21:04:41</td>
        <td>2013-02-20 21:03:40</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fd5aa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 21:08:18</td>
        <td>2013-02-20 21:07:13</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fd5aa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 21:15:35</td>
        <td>2013-02-20 21:14:33</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd4017cc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 21:26:23</td>
        <td>2013-02-20 21:25:23</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd4017cc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 21:27:07</td>
        <td>2013-02-20 21:26:05</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fd5aa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 21:30:46</td>
        <td>2013-02-20 21:29:43</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd401858-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 21:43:06</td>
        <td>2013-02-20 21:42:01</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd401858-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 21:43:31</td>
        <td>2013-02-20 21:42:28</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd401858-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 21:43:56</td>
        <td>2013-02-20 21:42:51</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd401858-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 21:47:26</td>
        <td>2013-02-20 21:46:23</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd401858-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 21:47:58</td>
        <td>2013-02-20 21:46:56</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd4017cc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 22:06:54</td>
        <td>2013-02-20 22:05:54</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd4017cc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 22:07:12</td>
        <td>2013-02-20 22:06:11</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd4017cc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 22:08:11</td>
        <td>2013-02-20 22:07:11</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd4017cc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 22:09:01</td>
        <td>2013-02-20 22:07:59</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd4017cc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 22:10:05</td>
        <td>2013-02-20 22:09:03</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd4017cc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 22:11:01</td>
        <td>2013-02-20 22:09:57</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd4017cc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 22:11:17</td>
        <td>2013-02-20 22:10:12</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd4017cc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 22:11:35</td>
        <td>2013-02-20 22:10:34</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3d46fa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 22:14:18</td>
        <td>2013-02-20 22:13:16</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd401d6c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 23:31:19</td>
        <td>2013-02-20 23:30:16</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd401d6c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 23:34:01</td>
        <td>2013-02-20 23:32:57</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd401df8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 23:39:57</td>
        <td>2013-02-20 23:38:56</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd27d4dc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 23:40:31</td>
        <td>2013-02-20 23:39:29</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd27d4dc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 23:41:39</td>
        <td>2013-02-20 23:40:37</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd27d4dc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 23:46:07</td>
        <td>2013-02-20 23:45:05</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd401df8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 23:52:01</td>
        <td>2013-02-20 23:50:57</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd27d45a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 23:56:07</td>
        <td>2013-02-20 23:55:02</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd401df8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 23:56:28</td>
        <td>2013-02-20 23:55:27</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd401df8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-20 23:57:18</td>
        <td>2013-02-20 23:56:16</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd401df8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 00:03:29</td>
        <td>2013-02-21 00:02:29</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd401e7a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 00:23:47</td>
        <td>2013-02-21 00:22:45</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd401e7a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 00:24:08</td>
        <td>2013-02-21 00:23:08</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd401e7a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 00:24:39</td>
        <td>2013-02-21 00:23:39</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd401e7a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 00:25:02</td>
        <td>2013-02-21 00:23:58</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd401e7a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 00:25:16</td>
        <td>2013-02-21 00:24:08</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd401e7a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 00:25:26</td>
        <td>2013-02-21 00:24:25</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd401e7a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 00:25:44</td>
        <td>2013-02-21 00:24:41</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd401e7a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 00:26:01</td>
        <td>2013-02-21 00:24:58</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd401e7a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 00:26:14</td>
        <td>2013-02-21 00:25:09</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd401e7a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 00:26:27</td>
        <td>2013-02-21 00:25:22</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd401e7a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 00:26:41</td>
        <td>2013-02-21 00:25:36</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd401e7a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 00:27:00</td>
        <td>2013-02-21 00:25:56</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd401e7a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 00:27:13</td>
        <td>2013-02-21 00:26:10</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd401e7a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 00:27:35</td>
        <td>2013-02-21 00:26:34</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd401e7a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 00:27:55</td>
        <td>2013-02-21 00:26:53</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd401e7a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 00:28:10</td>
        <td>2013-02-21 00:27:07</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd401e7a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 00:28:23</td>
        <td>2013-02-21 00:27:20</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd401e7a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 00:28:59</td>
        <td>2013-02-21 00:27:55</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ff47c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 00:40:49</td>
        <td>2013-02-21 00:39:45</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ff47c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 00:43:06</td>
        <td>2013-02-21 00:42:00</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ff47c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 00:45:12</td>
        <td>2013-02-21 00:44:07</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3ff47c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 00:49:14</td>
        <td>2013-02-21 00:48:08</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd401efc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 01:53:41</td>
        <td>2013-02-21 01:52:38</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd401efc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 01:53:56</td>
        <td>2013-02-21 01:52:50</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd401efc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 01:54:48</td>
        <td>2013-02-21 01:53:48</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd401efc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 01:57:48</td>
        <td>2013-02-21 01:56:48</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd401efc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 01:59:07</td>
        <td>2013-02-21 01:58:00</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fc9fc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 01:59:09</td>
        <td>2013-02-21 01:58:01</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd401efc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 01:59:26</td>
        <td>2013-02-21 01:58:26</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd401efc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 02:00:06</td>
        <td>2013-02-21 01:59:03</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd401efc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 02:00:38</td>
        <td>2013-02-21 01:59:32</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd402334-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 02:02:08</td>
        <td>2013-02-21 02:01:01</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd401efc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 02:02:22</td>
        <td>2013-02-21 02:01:13</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd401efc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 02:03:12</td>
        <td>2013-02-21 02:02:07</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd4008f4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 02:07:56</td>
        <td>2013-02-21 02:06:51</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fc9fc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 02:12:09</td>
        <td>2013-02-21 02:10:59</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd40073c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 02:12:22</td>
        <td>2013-02-21 02:11:18</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd40073c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 02:12:44</td>
        <td>2013-02-21 02:11:41</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd40073c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 02:16:46</td>
        <td>2013-02-21 02:15:46</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd402334-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 02:19:03</td>
        <td>2013-02-21 02:18:01</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd27cfaa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 02:19:11</td>
        <td>2013-02-21 02:18:10</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd400584-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 02:19:27</td>
        <td>2013-02-21 02:18:24</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd40073c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 02:19:31</td>
        <td>2013-02-21 02:18:24</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd402334-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 02:19:35</td>
        <td>2013-02-21 02:18:35</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd400584-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 02:20:03</td>
        <td>2013-02-21 02:19:00</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd27cfaa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 02:20:12</td>
        <td>2013-02-21 02:19:04</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd402334-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 02:20:14</td>
        <td>2013-02-21 02:19:11</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd40073c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 02:20:37</td>
        <td>2013-02-21 02:19:37</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd4008f4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 02:22:05</td>
        <td>2013-02-21 02:21:00</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd40073c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 02:22:59</td>
        <td>2013-02-21 02:21:51</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd4008f4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 02:23:19</td>
        <td>2013-02-21 02:22:17</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd40073c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 02:25:37</td>
        <td>2013-02-21 02:24:35</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd27cfaa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 02:25:57</td>
        <td>2013-02-21 02:24:48</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd400584-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 02:27:05</td>
        <td>2013-02-21 02:25:56</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd40073c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 02:29:53</td>
        <td>2013-02-21 02:28:52</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd40073c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 02:30:12</td>
        <td>2013-02-21 02:29:09</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd40073c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 02:30:27</td>
        <td>2013-02-21 02:29:25</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd27cfaa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 02:32:04</td>
        <td>2013-02-21 02:30:58</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd27cfaa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 02:32:44</td>
        <td>2013-02-21 02:31:37</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd27cfaa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 02:33:15</td>
        <td>2013-02-21 02:32:10</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd27cfaa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 02:33:49</td>
        <td>2013-02-21 02:32:43</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd40241a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 04:01:22</td>
        <td>2013-02-21 04:00:21</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd40241a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 04:04:00</td>
        <td>2013-02-21 04:02:57</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd40241a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 04:06:05</td>
        <td>2013-02-21 04:04:59</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd40241a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 04:07:54</td>
        <td>2013-02-21 04:06:48</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd40241a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 04:08:56</td>
        <td>2013-02-21 04:07:51</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd40241a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 04:09:41</td>
        <td>2013-02-21 04:08:36</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd40241a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 04:09:56</td>
        <td>2013-02-21 04:08:54</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd40241a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 04:10:12</td>
        <td>2013-02-21 04:09:04</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd40241a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 04:12:21</td>
        <td>2013-02-21 04:11:17</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd40241a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 04:14:14</td>
        <td>2013-02-21 04:13:07</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd40241a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 04:14:36</td>
        <td>2013-02-21 04:13:35</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd40241a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 04:15:30</td>
        <td>2013-02-21 04:14:28</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd4024e2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 05:34:58</td>
        <td>2013-02-21 05:33:55</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd4024e2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 05:35:46</td>
        <td>2013-02-21 05:34:41</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd4024e2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 05:37:06</td>
        <td>2013-02-21 05:36:03</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd4024e2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 05:37:17</td>
        <td>2013-02-21 05:36:15</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd4024e2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 05:37:56</td>
        <td>2013-02-21 05:36:47</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd4024e2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 05:38:12</td>
        <td>2013-02-21 05:37:08</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd4024e2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 05:38:35</td>
        <td>2013-02-21 05:37:28</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd4024e2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 05:39:00</td>
        <td>2013-02-21 05:37:54</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd4024e2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 05:39:19</td>
        <td>2013-02-21 05:38:13</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd4024e2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 05:39:45</td>
        <td>2013-02-21 05:38:39</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd4024e2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 05:40:09</td>
        <td>2013-02-21 05:39:05</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd4024e2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 05:40:29</td>
        <td>2013-02-21 05:39:21</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd4024e2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 05:40:47</td>
        <td>2013-02-21 05:39:39</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd4024e2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 05:41:20</td>
        <td>2013-02-21 05:40:10</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd4024e2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 05:41:40</td>
        <td>2013-02-21 05:40:31</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd4024e2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 05:41:48</td>
        <td>2013-02-21 05:40:45</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd4024e2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 05:42:01</td>
        <td>2013-02-21 05:40:53</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd400084-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 05:55:31</td>
        <td>2013-02-21 05:54:22</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd400084-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 05:59:44</td>
        <td>2013-02-21 05:58:42</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd400084-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 06:00:18</td>
        <td>2013-02-21 05:59:13</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd400084-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 06:00:34</td>
        <td>2013-02-21 05:59:32</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd400084-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 06:02:58</td>
        <td>2013-02-21 06:01:55</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd400084-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 06:03:18</td>
        <td>2013-02-21 06:02:17</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd400084-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 06:04:00</td>
        <td>2013-02-21 06:02:58</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd400084-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 06:04:18</td>
        <td>2013-02-21 06:03:12</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd400084-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 06:04:42</td>
        <td>2013-02-21 06:03:39</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd400084-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 06:05:10</td>
        <td>2013-02-21 06:04:07</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd400084-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 06:05:45</td>
        <td>2013-02-21 06:04:41</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd400084-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 06:07:23</td>
        <td>2013-02-21 06:06:19</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd400084-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 06:07:46</td>
        <td>2013-02-21 06:06:44</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fda6e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 06:22:55</td>
        <td>2013-02-21 06:21:49</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fda6e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 06:29:35</td>
        <td>2013-02-21 06:28:29</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd3fda6e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 06:34:51</td>
        <td>2013-02-21 06:33:42</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd402988-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 07:39:35</td>
        <td>2013-02-21 07:38:32</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd402988-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 07:39:58</td>
        <td>2013-02-21 07:38:57</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd402988-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 07:40:29</td>
        <td>2013-02-21 07:39:23</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd402988-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 07:40:51</td>
        <td>2013-02-21 07:39:51</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd402988-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 07:41:15</td>
        <td>2013-02-21 07:40:15</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd402988-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 07:41:53</td>
        <td>2013-02-21 07:40:51</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd402988-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 07:43:07</td>
        <td>2013-02-21 07:42:00</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd402988-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 07:43:22</td>
        <td>2013-02-21 07:42:13</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd402988-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 07:43:44</td>
        <td>2013-02-21 07:42:35</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd402988-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 07:43:56</td>
        <td>2013-02-21 07:42:53</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd402988-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 07:44:38</td>
        <td>2013-02-21 07:43:30</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd402988-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 07:44:58</td>
        <td>2013-02-21 07:43:57</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd402988-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 07:45:23</td>
        <td>2013-02-21 07:44:15</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd402988-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 07:45:39</td>
        <td>2013-02-21 07:44:31</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd402988-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 07:45:53</td>
        <td>2013-02-21 07:44:46</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd402b90-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 13:42:15</td>
        <td>2013-02-21 13:41:11</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd402b90-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 13:42:44</td>
        <td>2013-02-21 13:41:43</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd402b90-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 13:43:04</td>
        <td>2013-02-21 13:41:58</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd402b90-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 13:43:32</td>
        <td>2013-02-21 13:42:28</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd402b90-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 13:43:45</td>
        <td>2013-02-21 13:42:42</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd402b90-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 13:44:27</td>
        <td>2013-02-21 13:43:25</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd402b90-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 13:44:57</td>
        <td>2013-02-21 13:43:52</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd402b90-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 13:45:09</td>
        <td>2013-02-21 13:44:05</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd402b90-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 13:45:44</td>
        <td>2013-02-21 13:44:37</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd402b90-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 13:47:31</td>
        <td>2013-02-21 13:46:30</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd402b90-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 13:48:10</td>
        <td>2013-02-21 13:46:59</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd403036-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 14:23:32</td>
        <td>2013-02-21 14:22:29</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd403036-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 14:26:13</td>
        <td>2013-02-21 14:25:07</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd403036-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 14:26:28</td>
        <td>2013-02-21 14:25:22</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd403036-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 14:27:54</td>
        <td>2013-02-21 14:26:49</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd403036-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 14:28:20</td>
        <td>2013-02-21 14:27:20</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd403036-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 14:28:51</td>
        <td>2013-02-21 14:27:45</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd403036-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 14:29:05</td>
        <td>2013-02-21 14:28:03</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd403036-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 14:30:08</td>
        <td>2013-02-21 14:29:00</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd403036-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 14:30:19</td>
        <td>2013-02-21 14:29:14</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd403036-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 14:31:52</td>
        <td>2013-02-21 14:30:42</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd403036-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 14:33:16</td>
        <td>2013-02-21 14:32:13</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd403036-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 14:33:28</td>
        <td>2013-02-21 14:32:21</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd403036-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2013-02-21 14:33:36</td>
        <td>2013-02-21 14:32:33</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd71adaa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2014-10-13 20:18:08</td>
        <td>2014-10-13 20:15:54</td>
        <td>-2</td>
    </tr>
    <tr>
        <td>None</td>
        <td>2014-10-20 22:32:30</td>
        <td>2014-10-20 22:31:20</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>None</td>
        <td>2014-10-20 22:32:30</td>
        <td>2014-10-20 22:31:20</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd70d24a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2014-10-20 23:02:48</td>
        <td>2014-10-20 22:59:07</td>
        <td>-3</td>
    </tr>
    <tr>
        <td>fd70d24a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2014-10-20 23:13:56</td>
        <td>2014-10-20 23:10:14</td>
        <td>-3</td>
    </tr>
    <tr>
        <td>fd70d24a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2014-10-20 23:19:16</td>
        <td>2014-10-20 23:15:32</td>
        <td>-3</td>
    </tr>
    <tr>
        <td>fd70d24a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2014-10-20 23:26:14</td>
        <td>2014-10-20 23:22:32</td>
        <td>-3</td>
    </tr>
    <tr>
        <td>fd73ea8e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2014-10-24 01:33:07</td>
        <td>2014-10-24 01:29:39</td>
        <td>-3</td>
    </tr>
    <tr>
        <td>fd7039a2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2014-10-25 12:29:00</td>
        <td>2014-10-25 12:27:37</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd74ffd2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2014-11-05 21:09:54</td>
        <td>2014-11-05 21:08:35</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd751e86-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2014-11-21 00:10:17</td>
        <td>2014-11-21 00:03:03</td>
        <td>-7</td>
    </tr>
    <tr>
        <td>fd6eed4a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2014-11-10 01:29:24</td>
        <td>2014-11-10 01:28:13</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd76473e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2014-11-17 16:23:24</td>
        <td>2014-11-17 16:20:16</td>
        <td>-3</td>
    </tr>
    <tr>
        <td>fd76473e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2014-11-17 16:24:03</td>
        <td>2014-11-17 16:20:54</td>
        <td>-3</td>
    </tr>
    <tr>
        <td>fd7485a2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2014-11-19 02:22:55</td>
        <td>2014-11-19 00:11:46</td>
        <td>-131</td>
    </tr>
    <tr>
        <td>fd751e86-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2014-11-21 00:10:53</td>
        <td>2014-11-21 00:03:57</td>
        <td>-6</td>
    </tr>
    <tr>
        <td>fd751e86-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2014-11-21 00:18:39</td>
        <td>2014-11-21 00:11:16</td>
        <td>-7</td>
    </tr>
    <tr>
        <td>fd768da2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2014-11-25 21:25:02</td>
        <td>2014-11-25 21:22:45</td>
        <td>-2</td>
    </tr>
    <tr>
        <td>fd76b822-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2014-11-26 05:42:25</td>
        <td>2014-11-26 05:40:43</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd76de92-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2014-11-29 09:03:40</td>
        <td>2014-11-29 07:05:37</td>
        <td>-118</td>
    </tr>
    <tr>
        <td>fd76de92-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2014-12-02 09:01:07</td>
        <td>2014-12-02 07:02:28</td>
        <td>-118</td>
    </tr>
    <tr>
        <td>fd76de92-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2014-12-02 09:02:15</td>
        <td>2014-12-02 07:03:38</td>
        <td>-118</td>
    </tr>
    <tr>
        <td>fd74e592-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2014-12-02 07:20:55</td>
        <td>2014-12-02 07:19:50</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd77670e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2014-12-04 03:24:42</td>
        <td>2014-12-04 03:23:25</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>None</td>
        <td>2014-12-10 04:06:53</td>
        <td>2014-12-10 04:05:53</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd6d1998-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2014-12-10 16:14:59</td>
        <td>2014-12-10 15:34:41</td>
        <td>-40</td>
    </tr>
    <tr>
        <td>fd6d1998-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2014-12-10 16:18:13</td>
        <td>2014-12-10 15:34:41</td>
        <td>-43</td>
    </tr>
    <tr>
        <td>fd75179c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2014-12-10 20:49:32</td>
        <td>2014-12-10 20:48:25</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>None</td>
        <td>2014-12-10 20:49:32</td>
        <td>2014-12-10 20:48:24</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>None</td>
        <td>2014-12-10 20:49:32</td>
        <td>2014-12-10 20:48:24</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd75e6f4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2014-12-10 21:48:10</td>
        <td>2014-12-10 20:48:25</td>
        <td>-59</td>
    </tr>
    <tr>
        <td>None</td>
        <td>2014-12-10 20:49:32</td>
        <td>2014-12-10 20:48:24</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>None</td>
        <td>2014-12-10 21:48:10</td>
        <td>2014-12-10 20:48:24</td>
        <td>-59</td>
    </tr>
    <tr>
        <td>None</td>
        <td>2014-12-10 21:48:10</td>
        <td>2014-12-10 20:48:24</td>
        <td>-59</td>
    </tr>
    <tr>
        <td>None</td>
        <td>2014-12-10 21:48:10</td>
        <td>2014-12-10 20:48:24</td>
        <td>-59</td>
    </tr>
    <tr>
        <td>None</td>
        <td>2014-12-10 21:48:10</td>
        <td>2014-12-10 20:48:24</td>
        <td>-59</td>
    </tr>
    <tr>
        <td>None</td>
        <td>2014-12-10 21:48:10</td>
        <td>2014-12-10 20:48:24</td>
        <td>-59</td>
    </tr>
    <tr>
        <td>fd7014cc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2014-12-11 02:39:08</td>
        <td>2014-12-11 02:25:25</td>
        <td>-13</td>
    </tr>
    <tr>
        <td>fd75e6f4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2014-12-16 20:26:29</td>
        <td>2014-12-16 19:26:35</td>
        <td>-59</td>
    </tr>
    <tr>
        <td>fd75e6f4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2014-12-16 20:30:59</td>
        <td>2014-12-16 19:31:00</td>
        <td>-59</td>
    </tr>
    <tr>
        <td>fd75e6f4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2014-12-16 20:36:36</td>
        <td>2014-12-16 19:36:37</td>
        <td>-59</td>
    </tr>
    <tr>
        <td>fd75e6f4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2014-12-16 20:42:54</td>
        <td>2014-12-16 19:42:57</td>
        <td>-59</td>
    </tr>
    <tr>
        <td>fd770cd2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2014-12-18 00:33:52</td>
        <td>2014-12-18 00:21:13</td>
        <td>-12</td>
    </tr>
    <tr>
        <td>None</td>
        <td>2014-12-18 00:33:52</td>
        <td>2014-12-18 00:21:13</td>
        <td>-12</td>
    </tr>
    <tr>
        <td>fd78358a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2014-12-18 01:06:28</td>
        <td>2014-12-18 01:03:00</td>
        <td>-3</td>
    </tr>
    <tr>
        <td>fd78358a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2014-12-18 01:11:24</td>
        <td>2014-12-18 01:07:46</td>
        <td>-3</td>
    </tr>
    <tr>
        <td>fd78358a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2014-12-18 01:15:57</td>
        <td>2014-12-18 01:12:08</td>
        <td>-3</td>
    </tr>
    <tr>
        <td>fd770cd2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2014-12-18 02:24:26</td>
        <td>2014-12-18 02:11:52</td>
        <td>-12</td>
    </tr>
    <tr>
        <td>fd770cd2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2014-12-18 02:28:48</td>
        <td>2014-12-18 02:16:04</td>
        <td>-12</td>
    </tr>
    <tr>
        <td>fd78358a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2014-12-19 00:30:49</td>
        <td>2014-12-19 00:26:51</td>
        <td>-3</td>
    </tr>
    <tr>
        <td>fd75e6f4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2014-12-19 20:13:51</td>
        <td>2014-12-19 19:13:52</td>
        <td>-59</td>
    </tr>
    <tr>
        <td>fd75e6f4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2014-12-19 20:26:54</td>
        <td>2014-12-19 19:26:53</td>
        <td>-60</td>
    </tr>
    <tr>
        <td>fd75e6f4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2014-12-19 20:47:19</td>
        <td>2014-12-19 19:47:39</td>
        <td>-59</td>
    </tr>
    <tr>
        <td>fd76c59c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2014-12-21 07:15:00</td>
        <td>2014-12-21 07:12:40</td>
        <td>-2</td>
    </tr>
    <tr>
        <td>fd6a6590-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2014-12-21 20:47:34</td>
        <td>2014-12-21 20:40:50</td>
        <td>-6</td>
    </tr>
    <tr>
        <td>fd6a6590-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2014-12-21 20:48:14</td>
        <td>2014-12-21 20:43:08</td>
        <td>-5</td>
    </tr>
    <tr>
        <td>fd6a6590-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2014-12-21 20:49:57</td>
        <td>2014-12-21 20:44:48</td>
        <td>-5</td>
    </tr>
    <tr>
        <td>fd747be8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2014-12-21 22:02:22</td>
        <td>2014-12-21 22:00:24</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd747be8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2014-12-21 22:15:38</td>
        <td>2014-12-21 22:13:33</td>
        <td>-2</td>
    </tr>
    <tr>
        <td>fd747be8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2014-12-21 22:22:00</td>
        <td>2014-12-21 22:19:57</td>
        <td>-2</td>
    </tr>
    <tr>
        <td>fd7701ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2014-12-24 19:15:01</td>
        <td>2014-12-24 19:02:02</td>
        <td>-12</td>
    </tr>
    <tr>
        <td>fd7701ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2014-12-24 19:18:36</td>
        <td>2014-12-24 19:05:30</td>
        <td>-13</td>
    </tr>
    <tr>
        <td>fd78fcfe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2014-12-26 18:26:52</td>
        <td>2014-12-26 18:25:52</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd789426-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2014-12-27 03:49:27</td>
        <td>2014-12-27 03:31:37</td>
        <td>-17</td>
    </tr>
    <tr>
        <td>fd75908c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2014-12-27 22:10:55</td>
        <td>2014-12-27 22:08:44</td>
        <td>-2</td>
    </tr>
    <tr>
        <td>fd784dfe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2014-12-27 22:30:42</td>
        <td>2014-12-27 22:28:46</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd78a5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2015-01-02 00:13:03</td>
        <td>2015-01-02 00:05:17</td>
        <td>-7</td>
    </tr>
    <tr>
        <td>fd725e80-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2015-01-02 01:59:23</td>
        <td>2015-01-02 01:55:13</td>
        <td>-4</td>
    </tr>
    <tr>
        <td>fd69f862-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2015-01-02 22:41:23</td>
        <td>2015-01-02 22:34:14</td>
        <td>-7</td>
    </tr>
    <tr>
        <td>fd69f862-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2015-01-02 22:41:52</td>
        <td>2015-01-02 22:34:44</td>
        <td>-7</td>
    </tr>
    <tr>
        <td>fd69f862-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2015-01-02 22:42:03</td>
        <td>2015-01-02 22:34:54</td>
        <td>-7</td>
    </tr>
    <tr>
        <td>fd69f862-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2015-01-02 22:58:37</td>
        <td>2015-01-02 22:51:28</td>
        <td>-7</td>
    </tr>
    <tr>
        <td>fd69f862-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2015-01-02 23:03:26</td>
        <td>2015-01-02 22:56:16</td>
        <td>-7</td>
    </tr>
    <tr>
        <td>fd69f862-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2015-01-02 23:07:43</td>
        <td>2015-01-02 23:00:32</td>
        <td>-7</td>
    </tr>
    <tr>
        <td>fd69f862-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2015-01-02 23:09:59</td>
        <td>2015-01-02 23:02:49</td>
        <td>-7</td>
    </tr>
    <tr>
        <td>fd69f862-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2015-01-02 23:12:44</td>
        <td>2015-01-02 23:05:35</td>
        <td>-7</td>
    </tr>
    <tr>
        <td>fd79d2b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2015-01-03 23:48:37</td>
        <td>2015-01-03 23:29:59</td>
        <td>-18</td>
    </tr>
    <tr>
        <td>None</td>
        <td>2015-01-04 21:28:21</td>
        <td>2015-01-04 21:27:16</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd793340-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2015-01-04 21:40:27</td>
        <td>2015-01-04 21:39:23</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd786460-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2015-01-05 20:13:45</td>
        <td>2015-01-05 20:00:13</td>
        <td>-13</td>
    </tr>
    <tr>
        <td>fd786460-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2015-01-05 20:23:27</td>
        <td>2015-01-05 20:09:55</td>
        <td>-13</td>
    </tr>
    <tr>
        <td>fd79d2b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2015-01-06 01:22:28</td>
        <td>2015-01-06 01:03:48</td>
        <td>-18</td>
    </tr>
    <tr>
        <td>fd7a1382-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2015-01-06 05:42:17</td>
        <td>2015-01-06 05:40:55</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd72d7a2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2015-01-07 00:59:24</td>
        <td>2015-01-06 23:27:07</td>
        <td>-92</td>
    </tr>
    <tr>
        <td>fd72d7a2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2015-01-07 01:02:04</td>
        <td>2015-01-06 23:29:51</td>
        <td>-92</td>
    </tr>
    <tr>
        <td>fd72d7a2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2015-01-07 01:05:41</td>
        <td>2015-01-06 23:33:25</td>
        <td>-92</td>
    </tr>
    <tr>
        <td>fd72d7a2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2015-01-07 01:09:50</td>
        <td>2015-01-06 23:40:01</td>
        <td>-89</td>
    </tr>
    <tr>
        <td>fd72d7a2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2015-01-07 01:49:25</td>
        <td>2015-01-07 00:17:10</td>
        <td>-92</td>
    </tr>
    <tr>
        <td>fd72d7a2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2015-01-07 01:55:37</td>
        <td>2015-01-07 00:23:22</td>
        <td>-92</td>
    </tr>
    <tr>
        <td>fd72d7a2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2015-01-07 02:05:32</td>
        <td>2015-01-07 00:33:22</td>
        <td>-92</td>
    </tr>
    <tr>
        <td>fd72d7a2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2015-01-07 02:11:31</td>
        <td>2015-01-07 00:39:15</td>
        <td>-92</td>
    </tr>
    <tr>
        <td>fd72d7a2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2015-01-07 02:32:13</td>
        <td>2015-01-07 01:00:00</td>
        <td>-92</td>
    </tr>
    <tr>
        <td>fd72d7a2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2015-01-08 20:22:56</td>
        <td>2015-01-08 18:50:32</td>
        <td>-92</td>
    </tr>
    <tr>
        <td>fd72d7a2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2015-01-08 20:29:56</td>
        <td>2015-01-08 18:57:32</td>
        <td>-92</td>
    </tr>
    <tr>
        <td>fd72d7a2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2015-01-08 20:40:41</td>
        <td>2015-01-08 19:08:17</td>
        <td>-92</td>
    </tr>
    <tr>
        <td>fd72d7a2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2015-01-08 20:50:42</td>
        <td>2015-01-08 19:18:18</td>
        <td>-92</td>
    </tr>
    <tr>
        <td>None</td>
        <td>2015-01-10 14:47:56</td>
        <td>2015-01-10 14:39:34</td>
        <td>-8</td>
    </tr>
    <tr>
        <td>fd7a39a2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2015-01-11 16:30:07</td>
        <td>2015-01-11 16:25:07</td>
        <td>-5</td>
    </tr>
    <tr>
        <td>fd7a39a2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2015-01-11 16:55:22</td>
        <td>2015-01-11 16:50:22</td>
        <td>-5</td>
    </tr>
    <tr>
        <td>fd7a1382-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2015-01-13 06:04:51</td>
        <td>2015-01-13 06:03:25</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd78928c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2015-01-30 22:37:19</td>
        <td>2015-01-30 22:33:30</td>
        <td>-3</td>
    </tr>
    <tr>
        <td>fd78928c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2015-01-30 22:38:59</td>
        <td>2015-01-30 22:35:10</td>
        <td>-3</td>
    </tr>
    <tr>
        <td>None</td>
        <td>2015-02-02 17:02:46</td>
        <td>2015-02-02 16:03:10</td>
        <td>-59</td>
    </tr>
    <tr>
        <td>None</td>
        <td>2015-02-04 22:18:37</td>
        <td>2015-02-04 21:18:56</td>
        <td>-59</td>
    </tr>
    <tr>
        <td>None</td>
        <td>2015-02-04 22:29:12</td>
        <td>2015-02-04 21:29:30</td>
        <td>-59</td>
    </tr>
    <tr>
        <td>None</td>
        <td>2015-02-04 22:29:12</td>
        <td>2015-02-04 21:29:30</td>
        <td>-59</td>
    </tr>
    <tr>
        <td>None</td>
        <td>2015-02-15 23:09:53</td>
        <td>2015-02-15 22:10:32</td>
        <td>-59</td>
    </tr>
    <tr>
        <td>None</td>
        <td>2015-02-15 22:51:07</td>
        <td>2015-02-15 22:45:31</td>
        <td>-5</td>
    </tr>
    <tr>
        <td>None</td>
        <td>2015-03-01 18:00:55</td>
        <td>2015-03-01 17:56:55</td>
        <td>-4</td>
    </tr>
    <tr>
        <td>None</td>
        <td>2015-03-18 14:51:44</td>
        <td>2015-03-18 13:52:31</td>
        <td>-59</td>
    </tr>
    <tr>
        <td>None</td>
        <td>2015-03-18 14:51:44</td>
        <td>2015-03-18 13:52:31</td>
        <td>-59</td>
    </tr>
    <tr>
        <td>None</td>
        <td>2015-04-12 22:02:58</td>
        <td>2015-04-12 21:57:52</td>
        <td>-5</td>
    </tr>
    <tr>
        <td>None</td>
        <td>2015-05-13 02:38:41</td>
        <td>2015-05-13 02:28:05</td>
        <td>-10</td>
    </tr>
    <tr>
        <td>None</td>
        <td>2015-05-13 02:49:36</td>
        <td>2015-05-13 02:38:59</td>
        <td>-10</td>
    </tr>
    <tr>
        <td>None</td>
        <td>2015-05-18 21:01:25</td>
        <td>2015-05-18 17:59:47</td>
        <td>-181</td>
    </tr>
    <tr>
        <td>None</td>
        <td>2015-05-18 21:01:25</td>
        <td>2015-05-18 17:59:47</td>
        <td>-181</td>
    </tr>
    <tr>
        <td>None</td>
        <td>2015-05-18 21:01:25</td>
        <td>2015-05-18 17:59:47</td>
        <td>-181</td>
    </tr>
    <tr>
        <td>None</td>
        <td>2015-05-18 21:01:25</td>
        <td>2015-05-18 17:59:47</td>
        <td>-181</td>
    </tr>
    <tr>
        <td>None</td>
        <td>2015-05-18 21:01:25</td>
        <td>2015-05-18 17:59:47</td>
        <td>-181</td>
    </tr>
    <tr>
        <td>None</td>
        <td>2015-05-18 21:38:13</td>
        <td>2015-05-18 18:34:37</td>
        <td>-183</td>
    </tr>
    <tr>
        <td>None</td>
        <td>2015-05-18 22:05:58</td>
        <td>2015-05-18 19:01:41</td>
        <td>-184</td>
    </tr>
    <tr>
        <td>None</td>
        <td>2015-05-18 22:20:10</td>
        <td>2015-05-18 19:16:29</td>
        <td>-183</td>
    </tr>
    <tr>
        <td>None</td>
        <td>2015-05-18 22:20:20</td>
        <td>2015-05-18 19:16:29</td>
        <td>-183</td>
    </tr>
    <tr>
        <td>fd7f57e8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2015-05-18 21:21:00</td>
        <td>2015-05-18 21:17:59</td>
        <td>-3</td>
    </tr>
    <tr>
        <td>None</td>
        <td>2015-05-20 00:02:03</td>
        <td>2015-05-20 00:00:43</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>None</td>
        <td>2015-05-20 00:09:44</td>
        <td>2015-05-20 00:08:21</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>None</td>
        <td>2015-05-20 02:51:38</td>
        <td>2015-05-20 02:50:14</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>None</td>
        <td>2015-05-20 22:32:54</td>
        <td>2015-05-20 22:31:28</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>None</td>
        <td>2015-05-20 23:29:27</td>
        <td>2015-05-20 23:27:59</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>None</td>
        <td>2015-05-20 23:36:44</td>
        <td>2015-05-20 23:35:19</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd82a0d8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2015-05-22 01:54:09</td>
        <td>2015-05-22 01:53:02</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fd892246-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2015-05-29 01:08:22</td>
        <td>2015-05-28 22:03:59</td>
        <td>-184</td>
    </tr>
    <tr>
        <td>None</td>
        <td>2015-05-29 01:24:41</td>
        <td>2015-05-28 22:20:20</td>
        <td>-184</td>
    </tr>
    <tr>
        <td>fd892246-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2015-05-29 01:28:07</td>
        <td>2015-05-28 22:23:45</td>
        <td>-184</td>
    </tr>
    <tr>
        <td>None</td>
        <td>2015-05-31 19:17:34</td>
        <td>2015-05-31 19:15:30</td>
        <td>-2</td>
    </tr>
    <tr>
        <td>None</td>
        <td>2015-05-31 19:17:34</td>
        <td>2015-05-31 19:15:30</td>
        <td>-2</td>
    </tr>
    <tr>
        <td>None</td>
        <td>2015-06-03 02:54:52</td>
        <td>2015-06-02 23:50:35</td>
        <td>-184</td>
    </tr>
    <tr>
        <td>None</td>
        <td>2015-06-03 02:54:52</td>
        <td>2015-06-02 23:50:35</td>
        <td>-184</td>
    </tr>
    <tr>
        <td>fd8a2d58-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2015-06-03 03:14:41</td>
        <td>2015-06-03 00:10:14</td>
        <td>-184</td>
    </tr>
    <tr>
        <td>fdaa22b6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2015-06-08 15:26:08</td>
        <td>2015-06-08 15:24:49</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>None</td>
        <td>2015-06-16 02:47:35</td>
        <td>2015-06-15 23:40:15</td>
        <td>-187</td>
    </tr>
    <tr>
        <td>fdab05b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2015-06-18 02:53:16</td>
        <td>2015-06-18 02:49:49</td>
        <td>-3</td>
    </tr>
    <tr>
        <td>None</td>
        <td>2015-06-28 20:36:34</td>
        <td>2015-06-28 20:35:25</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fdad67c8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2015-07-06 02:58:30</td>
        <td>2015-07-06 02:56:33</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>fdb23e60-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2015-07-10 02:18:15</td>
        <td>2015-07-10 02:13:30</td>
        <td>-4</td>
    </tr>
    <tr>
        <td>None</td>
        <td>2015-07-19 03:17:59</td>
        <td>2015-07-19 03:11:40</td>
        <td>-6</td>
    </tr>
    <tr>
        <td>None</td>
        <td>2015-08-03 22:49:35</td>
        <td>2015-08-03 22:47:53</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>None</td>
        <td>2015-08-03 23:06:07</td>
        <td>2015-08-03 23:04:20</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>None</td>
        <td>2015-09-16 15:10:34</td>
        <td>2015-09-16 15:08:40</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>None</td>
        <td>2015-09-21 23:07:14</td>
        <td>2015-09-21 23:05:38</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>None</td>
        <td>2015-09-22 00:38:34</td>
        <td>2015-09-22 00:36:58</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>None</td>
        <td>2015-09-22 00:55:48</td>
        <td>2015-09-22 00:54:12</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>None</td>
        <td>2015-09-22 01:07:03</td>
        <td>2015-09-22 01:05:28</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>None</td>
        <td>2015-09-22 01:22:17</td>
        <td>2015-09-22 01:20:42</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>None</td>
        <td>2015-09-27 21:23:22</td>
        <td>2015-09-27 21:16:23</td>
        <td>-6</td>
    </tr>
    <tr>
        <td>None</td>
        <td>2015-09-27 21:23:22</td>
        <td>2015-09-27 21:16:23</td>
        <td>-6</td>
    </tr>
    <tr>
        <td>fdbc021a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2015-09-30 00:56:04</td>
        <td>2015-09-30 00:53:05</td>
        <td>-2</td>
    </tr>
</table>



**Question 14: What is the average amount of time it took customers to complete all of the tests in the exam_answers table when the negative durations are excluded from your calculation (you should get 11233 minutes)?**


```python
%%sql
SELECT AVG(TIMESTAMPDIFF(minute,start_time,end_time)) AS AvgDuration 
FROM exam_answers
WHERE TIMESTAMPDIFF(minute,start_time,end_time) > 0;
```

    1 rows affected.





<table>
    <tr>
        <th>AvgDuration</th>
    </tr>
    <tr>
        <td>11233.0951</td>
    </tr>
</table>



You have just seen another first-hand example of how messy real-world data can be, and how easy it can be to miss the "mess" when your data sets are too large to examine thoroughly by eye.  Before continuing on to the next SQL lesson, make sure to watch the video about how you as a data analyst can practice building habits that will prevent you from being fooled by messy data.

**And, as always, feel free to practice more queries here!**


```python

```
