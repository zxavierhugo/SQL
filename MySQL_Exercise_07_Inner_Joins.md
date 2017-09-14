
Copyright Jana Schaich Borg/Attribution-NonCommercial 4.0 International (CC BY-NC 4.0)

# MySQL Exercise 7: Joining Tables with Inner Joins

Before completing these exercises, I strongly recommend that you watch the video called "What are Joins?" that describe what joins are, and how different types of joins work.

As one of the last building blocks we need to address our Dognition analysis questions, in this lesson we will learn how to combine tables using inner joins.

   
## 1. Inner Joins between 2 tables

To begin, load the sql library, connect to the Dognition database, and set the Dognition database as the default.


```python
%load_ext sql
%sql mysql://studentuser:studentpw@mysqlserver/dognitiondb
%sql USE dognitiondb;
```

    0 rows affected.





    []



Recall that tables in relational databases are linked through primary keys and sometimes other fields that are common to multiple tables (as is the case with our Dognition data set).  Our goal when we execute a JOIN or make a joined table is to use those common columns to let the database figure out which rows in one table match up to which rows in another table.  Once that mapping is established using at least one common field or column, the database can pull any columns you want out of the mapped, or joined, tables and output the matched data to one common table.

An inner join is a join that outputs only rows that have an exact match in both tables being joined:

<img src="https://duke.box.com/shared/static/xazeqtyq6bjo12ojvgxup4bx0e9qcn5d.jpg" width=400 alt="INNER_JOIN" />

To illustrate how this works, let's find out whether dog owners that are particularly surprised by their dog's performance on Dognition tests tend to own similar breeds (or breed types, or breed groups) of dogs.  There are many ways to address this question, but let's start by focusing on the dog owners who provided at least 10 ratings for one or more of their dogs in the ratings table. Of these owners, which 200 owners reported the highest average amount of surprise at their dog's performance, and what was the breed, breed_type, and breed_group of each of these owner's dog?

The surprise ratings are stored in the reviews table.  The dog breed information is provided in the dogs table.  There are two columns that are common to both tables: user_guid and dog_guid. How do we use the common columns to combine information from the two tables? 

To join the tables, you can use a WHERE clause and add a couple of details to the FROM clause so that the database knows from what table each field in your SELECT clause comes.  

First, start by adding all the columns we want to examine to the SELECT statement:

```mySQL
SELECT dog_guid AS DogID, user_guid AS UserID, AVG(rating) AS AvgRating, 
       COUNT(rating) AS NumRatings, breed, breed_group, breed_type
```
then list all the tables from which the fields we are interested in come, separated by commas (with no comma at the end of the list):

```mySQL
FROM dogs, reviews
```
then add the other restrictions:

```mySQL
GROUP BY user_guid, dog_guid, breed, breed_group, breed_type
HAVING NumRatings >= 10
ORDER BY AvgRating DESC
LIMIT 200
```

**Try running this query and see what happens:**




```python
%%sql
SELECT dog_guid AS DogID, user_guid AS UserID, AVG(rating) AS AvgRating, COUNT(rating) AS NumRatings, 
breed, breed_group, breed_type
FROM dogs, reviews
GROUP BY user_guid, dog_guid, breed, breed_group, breed_type
HAVING NumRatings >= 10
ORDER BY AvgRating DESC
LIMIT 200;
```

    (_mysql_exceptions.OperationalError) (1052, "Column 'dog_guid' in field list is ambiguous") [SQL: 'SELECT dog_guid AS DogID, user_guid AS UserID, AVG(rating) AS AvgRating, COUNT(rating) AS NumRatings, \nbreed, breed_group, breed_type\nFROM dogs, reviews\nGROUP BY user_guid, dog_guid, breed, breed_group, breed_type\nHAVING NumRatings >= 10\nORDER BY AvgRating DESC\nLIMIT 200;']


You should receive an error message stating that the identity of dog_guid and user_guid in the field list is ambiguous. The reason is that the column title exists in both tables, and MySQL doesn't know which one we want. We have to specify the table name before stating the field name, and separate the two names by a period <mark>(NOTE: read this entire section before deciding whether you want to execute this query)<mark\>:

```mySQL
SELECT dogs.dog_guid AS DogID, dogs.user_guid AS UserID, AVG(reviews.rating) AS AvgRating,     
       COUNT(reviews.rating) AS NumRatings, dogs.breed, dogs.breed_group, dogs.breed_type
FROM dogs, reviews
GROUP BY dogs.user_guid, dogs.dog_guid, dogs.breed, dogs.breed_group, dogs.breed_type
HAVING NumRatings >= 10
ORDER BY AvgRating DESC
LIMIT 200
```


You can also take advantage of aliases so that you don't have to write out the name of the tables each time.  Here I will introduce another syntax for aliases that omits the AS completely.  In this syntax, the alias is whatever word (or phrase, if you use quotation marks) follows immediately after the field or table name, separated by a space. Some people find it easier to read queries if you use the syntax without an AS to create table aliases, but retain the syntax with an AS to create field aliases (but both syntaxes work for field aliases and table aliases).  So we could write:

```mySQL
SELECT d.dog_guid AS DogID, d.user_guid AS UserID, AVG(r.rating) AS AvgRating, 
       COUNT(r.rating) AS NumRatings, d.breed, d.breed_group, d.breed_type
FROM dogs d, reviews r
GROUP BY DogID, UserID, d.breed, d.breed_group, d.breed_type
HAVING NumRatings >= 10
ORDER BY AvgRating DESC
LIMIT 200
```

I am tempted to tell you to run this query so that you will see what happens, but instead, I will explain what will happen and let you decide if you want to see what the output looks...and feels...like.  
      
There is nothing built into the database table definitions that can instruct the server how to combine the tables on its own (remember, this is how relational databases save space and remain flexible).  Further, the query as written does not tell the database how the two tables are related.  As a consequence, rather than match up the two tables according to the values in the user_id and/or dog_id column, the database will do the only thing it knows how to do which is output every single combination of the records in the dogs table with the records in the reviews table.  In other words, every single row of the dogs table will get paired with every single row of the reviews table.  This is known as a Cartesian product.  Not only will it be a heavy burden on the database to output a table that has the full length of one table multiplied times the full length of another (and frustrating to you, because the query would take a very long time to run), the output would be close to useless.

To prevent this from happening, tell the database how to relate the tables in the WHERE clause:

```mySQL
SELECT d.dog_guid AS DogID, d.user_guid AS UserID, AVG(r.rating) AS AvgRating, 
       COUNT(r.rating) AS NumRatings, d.breed, d.breed_group, d.breed_type
FROM dogs d, reviews r
WHERE d.dog_guid=r.dog_guid
GROUP BY UserID, DogID, d.breed, d.breed_group, d.breed_type
HAVING NumRatings >= 10
ORDER BY AvgRating DESC
LIMIT 200
```

To be very careful and exclude any incorrect dog_guid or user_guid entries, you can include both shared columns in the WHERE clause:

```mySQL
SELECT d.dog_guid AS DogID, d.user_guid AS UserID, AVG(r.rating) AS AvgRating, 
       COUNT(r.rating) AS NumRatings, d.breed, d.breed_group, d.breed_type
FROM dogs d, reviews r
WHERE d.dog_guid=r.dog_guid AND d.user_guid=r.user_guid
GROUP BY UserID, DogID, d.breed, d.breed_group, d.breed_type
HAVING NumRatings >= 10
ORDER BY AvgRating DESC
LIMIT 200
```

**Try running this query now:**


```python
%%sql
SELECT d.dog_guid AS DogID, d.user_guid AS UserID, AVG(r.rating) AS AvgRating, COUNT(r.rating) AS NumRatings, d.breed, d.breed_group, d.breed_type
FROM dogs d, reviews r
WHERE d.dog_guid = r.dog_guid AND d.user_guid = r.user_guid
GROUP BY UserID, DogID, d.breed, d.breed_group, d.breed_type
HAVING NumRatings >=10
ORDER BY AvgRating DESC
LIMIT 200
```

    38 rows affected.





<table>
    <tr>
        <th>DogID</th>
        <th>UserID</th>
        <th>AvgRating</th>
        <th>NumRatings</th>
        <th>breed</th>
        <th>breed_group</th>
        <th>breed_type</th>
    </tr>
    <tr>
        <td>fdbf39f8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce987914-7144-11e5-ba71-058fbc01cf0b</td>
        <td>8.0000</td>
        <td>12</td>
        <td>Canaan Dog</td>
        <td>Herding</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc09a82-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce99bb12-7144-11e5-ba71-058fbc01cf0b</td>
        <td>5.0000</td>
        <td>10</td>
        <td>Golden Doodle</td>
        <td>None</td>
        <td>Popular Hybrid</td>
    </tr>
    <tr>
        <td>fdbef330-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce984d2c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4.5385</td>
        <td>13</td>
        <td>Havanese</td>
        <td>Toy</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc0518a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce99661c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3.7333</td>
        <td>15</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fd684cf6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce473856-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3.7222</td>
        <td>18</td>
        <td>Labrador Retriever</td>
        <td>Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbf66e4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98b9b0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3.5000</td>
        <td>14</td>
        <td>Golden Retriever</td>
        <td>Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc09b0e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce99bb76-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3.2308</td>
        <td>13</td>
        <td>Golden Doodle</td>
        <td>None</td>
        <td>Popular Hybrid</td>
    </tr>
    <tr>
        <td>fdbeedc2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9847dc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3.1538</td>
        <td>13</td>
        <td>Eurasier</td>
        <td>None</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc180be-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9a7110-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3.0625</td>
        <td>16</td>
        <td>Chihuahua-Dachshund Mix</td>
        <td>None</td>
        <td>Cross Breed</td>
    </tr>
    <tr>
        <td>fd6aabd6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce6cd804-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3.0000</td>
        <td>13</td>
        <td>American Pit Bull Terrier</td>
        <td>None</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbfc0b2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98f3bc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.6923</td>
        <td>13</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdbedef4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce982e46-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.4615</td>
        <td>13</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdbec8ce-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce980e3e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.4545</td>
        <td>11</td>
        <td>Basset Hound</td>
        <td>Hound</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbeff06-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9856f0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.4000</td>
        <td>10</td>
        <td>Bearded Collie</td>
        <td>Herding</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbfe22c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce99049c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.4000</td>
        <td>10</td>
        <td>Irish Water Spaniel</td>
        <td>Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbed7ce-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce981dc0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.3333</td>
        <td>18</td>
        <td>Havanese</td>
        <td>Toy</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc1c31c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9ac674-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.3077</td>
        <td>13</td>
        <td>Labrador Retriever</td>
        <td>Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc0c49e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce99d674-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.2778</td>
        <td>18</td>
        <td>Other</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdc19b30-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9aab30-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.2222</td>
        <td>18</td>
        <td>Brittany</td>
        <td>Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbedf94-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce982ed2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.0000</td>
        <td>11</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdc0c502-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce99d732-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.0000</td>
        <td>15</td>
        <td>Golden Retriever</td>
        <td>Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbfddfe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce99049c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.9375</td>
        <td>16</td>
        <td>Irish Water Spaniel</td>
        <td>Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fd7104ae-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce7212a6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.9000</td>
        <td>10</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdc198d8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9a9500-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.8000</td>
        <td>10</td>
        <td>German Shepherd Dog</td>
        <td>Herding</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbe923c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce97f926-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.7647</td>
        <td>17</td>
        <td>Beagle</td>
        <td>Hound</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fd6a7490-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce6cf3fc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.7500</td>
        <td>20</td>
        <td>Golden Retriever</td>
        <td>Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbef5e2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce984f70-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.7000</td>
        <td>10</td>
        <td>Havanese</td>
        <td>Toy</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbea952-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9805ce-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.6667</td>
        <td>15</td>
        <td>Golden Doodle</td>
        <td>None</td>
        <td>Popular Hybrid</td>
    </tr>
    <tr>
        <td>fdbf9f10-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98d6de-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.6667</td>
        <td>18</td>
        <td>Australian Cattle Dog-Border Collie Mix</td>
        <td>None</td>
        <td>Cross Breed</td>
    </tr>
    <tr>
        <td>fdbf686a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98bca8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.6000</td>
        <td>10</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdbf6144-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98b776-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.5455</td>
        <td>11</td>
        <td>American Pit Bull Terrier-Labrador Retriever Mix</td>
        <td>None</td>
        <td>Cross Breed</td>
    </tr>
    <tr>
        <td>fdc1edec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9ae4b0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.5333</td>
        <td>15</td>
        <td>Poodle</td>
        <td>Non-Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbe7cac-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce97e10c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.3333</td>
        <td>12</td>
        <td>Labrador Retriever</td>
        <td>Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbe8a9e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce97f1f6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.2500</td>
        <td>16</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdbfe56a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce99049c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.2143</td>
        <td>14</td>
        <td>Irish Water Spaniel</td>
        <td>Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbf0456-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce985d9e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.7500</td>
        <td>16</td>
        <td>English Springer Spaniel</td>
        <td>Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc23ba8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9b306e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.4211</td>
        <td>19</td>
        <td>Cockapoo</td>
        <td>None</td>
        <td>Popular Hybrid</td>
    </tr>
    <tr>
        <td>fdbec040-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98098e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.4000</td>
        <td>10</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
</table>



The query should execute quickly.  This would NOT have been the case if you did not include the WHERE clause to combine the two tables. If you accidentally request a Cartesian product from datasets with billions of rows, you could be waiting for your query output for days (and will probably get in trouble with your database administrator).  <mark>So always remember to tell the database how to join your tables!</mark>

Let's examine our joined table a bit further.  The joined table outputted by the query above should have 38 rows, despite the fact that we set our LIMIT at 200.  The reason for this is that it turns out that a relatively small number of customers provided 10 or more reviews.  If you remove the HAVING and LIMIT BY clause from the query, you should end up with 389 rows.  **Go ahead and try it:**



```python
%%sql
SELECT d.dog_guid AS DogID, d.user_guid AS UserID, AVG(r.rating) AS AvgRating, COUNT(r.rating) AS NumRatings, d.breed, d.breed_group, d.breed_type
FROM dogs d, reviews r
WHERE d.dog_guid = r.dog_guid AND d.user_guid = r.user_guid
GROUP BY UserID, DogID, d.breed, d.breed_group, d.breed_type
ORDER BY AvgRating DESC;
```

    395 rows affected.





<table>
    <tr>
        <th>DogID</th>
        <th>UserID</th>
        <th>AvgRating</th>
        <th>NumRatings</th>
        <th>breed</th>
        <th>breed_group</th>
        <th>breed_type</th>
    </tr>
    <tr>
        <td>fdbedd14-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce975994-7144-11e5-ba71-058fbc01cf0b</td>
        <td>9.0000</td>
        <td>1</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdbf94de-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98cc34-7144-11e5-ba71-058fbc01cf0b</td>
        <td>9.0000</td>
        <td>2</td>
        <td>Pembroke Welsh Corgi</td>
        <td>Herding</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbf178e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce986546-7144-11e5-ba71-058fbc01cf0b</td>
        <td>9.0000</td>
        <td>1</td>
        <td>Airedale Terrier</td>
        <td>Terrier</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc057ac-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce996b9e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>9.0000</td>
        <td>1</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdc121dc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9a310a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>9.0000</td>
        <td>1</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdc19554-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9a8a24-7144-11e5-ba71-058fbc01cf0b</td>
        <td>9.0000</td>
        <td>1</td>
        <td>Miniature Pinscher</td>
        <td>Toy</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc147ac-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9a545a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>9.0000</td>
        <td>1</td>
        <td>Tibetan Terrier</td>
        <td>Non-Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc1c880-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce96c768-7144-11e5-ba71-058fbc01cf0b</td>
        <td>9.0000</td>
        <td>1</td>
        <td>Maltese</td>
        <td>Toy</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc059fa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce996dc4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>9.0000</td>
        <td>1</td>
        <td>Irish Water Spaniel</td>
        <td>Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc019c2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce99489e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>9.0000</td>
        <td>1</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fd41bca8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce7b055a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>8.0000</td>
        <td>1</td>
        <td>Labrador Retriever</td>
        <td>Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbf39f8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce987914-7144-11e5-ba71-058fbc01cf0b</td>
        <td>8.0000</td>
        <td>12</td>
        <td>Canaan Dog</td>
        <td>Herding</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc1e1a8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9ad934-7144-11e5-ba71-058fbc01cf0b</td>
        <td>7.5000</td>
        <td>2</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdbe7a40-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce97df9a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>7.0000</td>
        <td>6</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdbe8774-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce97ebfc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>7.0000</td>
        <td>4</td>
        <td>Labrador Retriever</td>
        <td>Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbf940c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98cb80-7144-11e5-ba71-058fbc01cf0b</td>
        <td>7.0000</td>
        <td>1</td>
        <td>Soft Coated Wheaten Terrier</td>
        <td>Terrier</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc1baca-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9ac2fa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>7.0000</td>
        <td>1</td>
        <td>Poodle-Miniature Schnauzer Mix</td>
        <td>None</td>
        <td>Cross Breed</td>
    </tr>
    <tr>
        <td>fdc2417a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9b35e6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>7.0000</td>
        <td>1</td>
        <td>Bichon Frise</td>
        <td>Non-Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc1706a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9a6846-7144-11e5-ba71-058fbc01cf0b</td>
        <td>6.8000</td>
        <td>5</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdc0c2aa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce99d322-7144-11e5-ba71-058fbc01cf0b</td>
        <td>6.5000</td>
        <td>2</td>
        <td>Border Collie</td>
        <td>Herding</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbf8598-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98c46e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>6.5000</td>
        <td>2</td>
        <td>Labrador Retriever</td>
        <td>Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fd442826-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce275612-7144-11e5-ba71-058fbc01cf0b</td>
        <td>6.0000</td>
        <td>1</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdbee278-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9782c0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>6.0000</td>
        <td>1</td>
        <td>Miniature Pinscher</td>
        <td>Toy</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc00f86-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce993b6a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>6.0000</td>
        <td>1</td>
        <td>Yorkshire Terrier</td>
        <td>Toy</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbef2cc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9838c8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>6.0000</td>
        <td>1</td>
        <td>Labradoodle</td>
        <td>None</td>
        <td>Popular Hybrid</td>
    </tr>
    <tr>
        <td>fdc01f76-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce994f1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>6.0000</td>
        <td>1</td>
        <td>Golden Doodle</td>
        <td>None</td>
        <td>Popular Hybrid</td>
    </tr>
    <tr>
        <td>fdc0345c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce995f78-7144-11e5-ba71-058fbc01cf0b</td>
        <td>6.0000</td>
        <td>1</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdc0cebc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce99ddd6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>6.0000</td>
        <td>2</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdc0d75e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce99e466-7144-11e5-ba71-058fbc01cf0b</td>
        <td>6.0000</td>
        <td>1</td>
        <td>Dachshund</td>
        <td>Hound</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc07f66-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce99ac9e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>6.0000</td>
        <td>2</td>
        <td>Pug</td>
        <td>Toy</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc0827c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce99af32-7144-11e5-ba71-058fbc01cf0b</td>
        <td>6.0000</td>
        <td>1</td>
        <td>Irish Setter</td>
        <td>Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc0ef00-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce99fe4c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>6.0000</td>
        <td>1</td>
        <td>I Don&#x27;t Know</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdc12d62-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9a381c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>6.0000</td>
        <td>1</td>
        <td>Chihuahua</td>
        <td>Toy</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc1ab52-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9ab8f0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>6.0000</td>
        <td>1</td>
        <td>Golden Retriever</td>
        <td>Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc191f8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9a8308-7144-11e5-ba71-058fbc01cf0b</td>
        <td>5.6667</td>
        <td>3</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdc1ea18-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9ae190-7144-11e5-ba71-058fbc01cf0b</td>
        <td>5.6667</td>
        <td>3</td>
        <td>Poodle</td>
        <td>Non-Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbf9740-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98cdb0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>5.5000</td>
        <td>4</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdbfd5ca-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce990258-7144-11e5-ba71-058fbc01cf0b</td>
        <td>5.2000</td>
        <td>5</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fd6913ac-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce66aaec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>5.0000</td>
        <td>3</td>
        <td>Golden Retriever</td>
        <td>Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbe9e08-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce980150-7144-11e5-ba71-058fbc01cf0b</td>
        <td>5.0000</td>
        <td>3</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdbfb1b2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98e5b6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>5.0000</td>
        <td>1</td>
        <td>Labrador Retriever</td>
        <td>Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc09a82-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce99bb12-7144-11e5-ba71-058fbc01cf0b</td>
        <td>5.0000</td>
        <td>10</td>
        <td>Golden Doodle</td>
        <td>None</td>
        <td>Popular Hybrid</td>
    </tr>
    <tr>
        <td>fdc072e6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9984a8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>5.0000</td>
        <td>1</td>
        <td>Labrador Retriever</td>
        <td>Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc0de66-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce99eaba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>5.0000</td>
        <td>9</td>
        <td>Poodle</td>
        <td>Non-Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbfbbb2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98eeee-7144-11e5-ba71-058fbc01cf0b</td>
        <td>5.0000</td>
        <td>1</td>
        <td>Golden Retriever</td>
        <td>Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc17ce0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9a6ca6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>5.0000</td>
        <td>2</td>
        <td>German Shepherd Dog</td>
        <td>Herding</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc16dae-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9a68aa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>5.0000</td>
        <td>1</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdc03862-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce995bea-7144-11e5-ba71-058fbc01cf0b</td>
        <td>5.0000</td>
        <td>3</td>
        <td>Shih Tzu</td>
        <td>Toy</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc0177e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce994434-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4.7500</td>
        <td>4</td>
        <td>Golden Doodle</td>
        <td>None</td>
        <td>Popular Hybrid</td>
    </tr>
    <tr>
        <td>fdbe8c6a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce97f426-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4.6667</td>
        <td>3</td>
        <td>Akita</td>
        <td>Working</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbf3d86-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce987b44-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4.6667</td>
        <td>6</td>
        <td>Australian Cattle Dog</td>
        <td>Herding</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc1db36-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9aceda-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4.6667</td>
        <td>3</td>
        <td>Bulldog</td>
        <td>Non-Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbef330-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce984d2c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4.5385</td>
        <td>13</td>
        <td>Havanese</td>
        <td>Toy</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbeef52-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce984908-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4.5000</td>
        <td>2</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdbf2fa8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce986eba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4.5000</td>
        <td>2</td>
        <td>Yorkshire Terrier-Poodle Mix</td>
        <td>None</td>
        <td>Cross Breed</td>
    </tr>
    <tr>
        <td>fdbff730-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce990fd2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4.5000</td>
        <td>2</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdc159b8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9a6260-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4.5000</td>
        <td>2</td>
        <td>Bernese Mountain Dog-Chow Chow Mix</td>
        <td>None</td>
        <td>Cross Breed</td>
    </tr>
    <tr>
        <td>fdc1a3b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9ab31e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4.5000</td>
        <td>4</td>
        <td>Chihuahua-Pomeranian Mix</td>
        <td>None</td>
        <td>Cross Breed</td>
    </tr>
    <tr>
        <td>fdbf008c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce985812-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4.3333</td>
        <td>6</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdc0140e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce994146-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4.3333</td>
        <td>6</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdbf2d3c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce986c76-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4.3333</td>
        <td>3</td>
        <td>Cocker Spaniel</td>
        <td>Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbf7b20-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98c2f2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4.2857</td>
        <td>7</td>
        <td>Poodle</td>
        <td>Non-Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc0cd90-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce99dd18-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4.2500</td>
        <td>4</td>
        <td>I Don&#x27;t Know</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdc1e45a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9adba0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4.2500</td>
        <td>8</td>
        <td>Poodle</td>
        <td>Non-Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fd684d5a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce4738c4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4.0000</td>
        <td>6</td>
        <td>Belgian Tervuren</td>
        <td>Herding</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fd675a08-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce416692-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4.0000</td>
        <td>2</td>
        <td>German Shepherd Dog</td>
        <td>Herding</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbe8210-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce97e4cc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4.0000</td>
        <td>1</td>
        <td>Soft Coated Wheaten Terrier</td>
        <td>Terrier</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbea3c6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce980330-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4.0000</td>
        <td>1</td>
        <td>Golden Retriever</td>
        <td>Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbf4196-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce987e50-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4.0000</td>
        <td>1</td>
        <td>German Shorthaired Pointer</td>
        <td>Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbfb0ea-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98e55c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4.0000</td>
        <td>4</td>
        <td>Border Collie</td>
        <td>Herding</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbf735a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98c11c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4.0000</td>
        <td>1</td>
        <td>American Pit Bull Terrier</td>
        <td>None</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbff3c0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce990910-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4.0000</td>
        <td>1</td>
        <td>Poodle-Yorkshire Terrier Mix</td>
        <td>None</td>
        <td>Cross Breed</td>
    </tr>
    <tr>
        <td>fdbffdac-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce992ddc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4.0000</td>
        <td>1</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdc010b2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce993d9a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4.0000</td>
        <td>1</td>
        <td>Boston Terrier-Chihuahua Mix</td>
        <td>None</td>
        <td>Cross Breed</td>
    </tr>
    <tr>
        <td>fdc05504-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce99690a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4.0000</td>
        <td>1</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdc0b878-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98191a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4.0000</td>
        <td>2</td>
        <td>German Shepherd Dog</td>
        <td>Herding</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc14248-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9a4d20-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4.0000</td>
        <td>2</td>
        <td>Border Collie</td>
        <td>Herding</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbea542-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9803ee-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4.0000</td>
        <td>1</td>
        <td>Golden Retriever</td>
        <td>Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fd6a73b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce6ccc1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3.8000</td>
        <td>5</td>
        <td>Shih Tzu</td>
        <td>Toy</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc188ca-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9a7b88-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3.7500</td>
        <td>4</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdc0518a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce99661c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3.7333</td>
        <td>15</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fd684cf6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce473856-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3.7222</td>
        <td>18</td>
        <td>Labrador Retriever</td>
        <td>Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc1a224-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9ab1f2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3.7143</td>
        <td>7</td>
        <td>Other</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdc1d0a0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9acbb0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3.7143</td>
        <td>7</td>
        <td>Beagle</td>
        <td>Hound</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbf3732-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce987608-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3.6667</td>
        <td>3</td>
        <td>Cavalier King Charles Spaniel-Poodle Mix</td>
        <td>None</td>
        <td>Cross Breed</td>
    </tr>
    <tr>
        <td>fdbff302-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce99085c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3.6667</td>
        <td>3</td>
        <td>Small Munsterlander Pointer</td>
        <td>None</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc06bca-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce997cf6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3.6667</td>
        <td>6</td>
        <td>Poodle-Havanese Mix</td>
        <td>None</td>
        <td>Cross Breed</td>
    </tr>
    <tr>
        <td>fdbfb77a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98eb42-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3.5556</td>
        <td>9</td>
        <td>I Don&#x27;t Know</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fd6f1360-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce7054fc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3.5000</td>
        <td>4</td>
        <td>Samoyed</td>
        <td>Working</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbed6f2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce981d0c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3.5000</td>
        <td>4</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdbf66e4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98b9b0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3.5000</td>
        <td>14</td>
        <td>Golden Retriever</td>
        <td>Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbe81a2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce97e3aa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3.5000</td>
        <td>2</td>
        <td>Other</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdc00234-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce980452-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3.5000</td>
        <td>4</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdbf62de-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98a56a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3.5000</td>
        <td>2</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdc021e2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce994b8c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3.5000</td>
        <td>4</td>
        <td>Rhodesian Ridgeback-Labrador Retriever Mix</td>
        <td>None</td>
        <td>Cross Breed</td>
    </tr>
    <tr>
        <td>fdc067ce-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce997a58-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3.5000</td>
        <td>2</td>
        <td>Labrador Retriever-Beagle Mix</td>
        <td>None</td>
        <td>Cross Breed</td>
    </tr>
    <tr>
        <td>fdc06d82-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce997e22-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3.5000</td>
        <td>2</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdbff852-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce99146e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3.5000</td>
        <td>2</td>
        <td>Chihuahua</td>
        <td>Toy</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc139e2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9a419a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3.5000</td>
        <td>4</td>
        <td>Australian Labradoodle</td>
        <td>None</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc1b160-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9abc6a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3.5000</td>
        <td>4</td>
        <td>German Shorthaired Pointer</td>
        <td>Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbfd11a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98ce0a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3.4000</td>
        <td>5</td>
        <td>Boxer</td>
        <td>Working</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc0fdb0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9a0810-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3.3750</td>
        <td>8</td>
        <td>Poodle</td>
        <td>Non-Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbed968-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce982338-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3.3333</td>
        <td>6</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdbf0e2e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce96a01c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3.3333</td>
        <td>3</td>
        <td>Boston Terrier</td>
        <td>Non-Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc1b228-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9abd32-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3.3333</td>
        <td>3</td>
        <td>Labrador Retriever</td>
        <td>Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc0bec2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce99cdb4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3.2857</td>
        <td>7</td>
        <td>Bearded Collie</td>
        <td>Herding</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc09b0e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce99bb76-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3.2308</td>
        <td>13</td>
        <td>Golden Doodle</td>
        <td>None</td>
        <td>Popular Hybrid</td>
    </tr>
    <tr>
        <td>fdbeedc2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9847dc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3.1538</td>
        <td>13</td>
        <td>Eurasier</td>
        <td>None</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc180be-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9a7110-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3.0625</td>
        <td>16</td>
        <td>Chihuahua-Dachshund Mix</td>
        <td>None</td>
        <td>Cross Breed</td>
    </tr>
    <tr>
        <td>fd6aabd6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce6cd804-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3.0000</td>
        <td>13</td>
        <td>American Pit Bull Terrier</td>
        <td>None</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fd6a5654-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce675b68-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3.0000</td>
        <td>1</td>
        <td>Bichon Frise</td>
        <td>Non-Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fd6abc8e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce6d4e9c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3.0000</td>
        <td>2</td>
        <td>Australian Cattle Dog-Labrador Retriever Mix</td>
        <td>None</td>
        <td>Cross Breed</td>
    </tr>
    <tr>
        <td>fdbeaf38-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98074a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3.0000</td>
        <td>1</td>
        <td>Alaskan Malamute-German Shepherd Dog Mix</td>
        <td>None</td>
        <td>Cross Breed</td>
    </tr>
    <tr>
        <td>fdbf1bda-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce986834-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3.0000</td>
        <td>3</td>
        <td>Cockapoo</td>
        <td>None</td>
        <td>Popular Hybrid</td>
    </tr>
    <tr>
        <td>fdbf5eb0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98afc4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3.0000</td>
        <td>1</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdbedb70-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98287e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3.0000</td>
        <td>1</td>
        <td>Labradoodle</td>
        <td>None</td>
        <td>Popular Hybrid</td>
    </tr>
    <tr>
        <td>fdbf9d76-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98d558-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3.0000</td>
        <td>3</td>
        <td>I Don&#x27;t Know</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdbfc044-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98da08-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3.0000</td>
        <td>2</td>
        <td>Golden Doodle</td>
        <td>None</td>
        <td>Popular Hybrid</td>
    </tr>
    <tr>
        <td>fdbf698c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98bdca-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3.0000</td>
        <td>1</td>
        <td>Collie</td>
        <td>Herding</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc01c06-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce994ace-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3.0000</td>
        <td>3</td>
        <td>Brittany</td>
        <td>Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc05126-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9965b8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3.0000</td>
        <td>1</td>
        <td>Australian Shepherd-Weimaraner Mix</td>
        <td>None</td>
        <td>Cross Breed</td>
    </tr>
    <tr>
        <td>fdc070fc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9980a2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3.0000</td>
        <td>4</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdc0776e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce99915a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3.0000</td>
        <td>4</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdbfa974-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98e070-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3.0000</td>
        <td>5</td>
        <td>Beagle</td>
        <td>Hound</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc01472-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce993a48-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3.0000</td>
        <td>4</td>
        <td>Labrador Retriever-Golden Retriever Mix</td>
        <td>None</td>
        <td>Cross Breed</td>
    </tr>
    <tr>
        <td>fdc13104-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9a3bb4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3.0000</td>
        <td>1</td>
        <td>Golden Retriever</td>
        <td>Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc1517a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9a5d38-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3.0000</td>
        <td>5</td>
        <td>Australian Shepherd</td>
        <td>Herding</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc1021a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9a11ac-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3.0000</td>
        <td>3</td>
        <td>Staffordshire Bull Terrier</td>
        <td>Terrier</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc1204c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9a2f70-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3.0000</td>
        <td>5</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdc20764-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9afd7e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3.0000</td>
        <td>5</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdbf2ee0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce986e60-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.8333</td>
        <td>6</td>
        <td>Labradoodle</td>
        <td>None</td>
        <td>Popular Hybrid</td>
    </tr>
    <tr>
        <td>fdbe98e0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce97fda4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.8000</td>
        <td>5</td>
        <td>Boxer</td>
        <td>Working</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc013aa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9940e2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.7500</td>
        <td>4</td>
        <td>Cairn Terrier</td>
        <td>Terrier</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc1892e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9a7bf6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.7500</td>
        <td>4</td>
        <td>German Shepherd Dog-Siberian Husky Mix</td>
        <td>None</td>
        <td>Cross Breed</td>
    </tr>
    <tr>
        <td>fdc1a288-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9ab256-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.7500</td>
        <td>4</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdbfc0b2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98f3bc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.6923</td>
        <td>13</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdbe7d56-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce97e0b2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.6667</td>
        <td>3</td>
        <td>Soft Coated Wheaten Terrier</td>
        <td>Terrier</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbe9d86-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce980038-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.6667</td>
        <td>9</td>
        <td>Australian Cattle Dog-German Shepherd Dog Mix</td>
        <td>None</td>
        <td>Cross Breed</td>
    </tr>
    <tr>
        <td>fdc21060-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9b03d2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.6250</td>
        <td>8</td>
        <td>German Shepherd Dog</td>
        <td>Herding</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fd702994-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce715af0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.6000</td>
        <td>5</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdc18f78-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9a80ce-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.6000</td>
        <td>5</td>
        <td>I Don&#x27;t Know</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fd6a6590-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce6ce394-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.5000</td>
        <td>2</td>
        <td>Golden Retriever</td>
        <td>Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbed148-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce981578-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.5000</td>
        <td>4</td>
        <td>Miniature Schnauzer</td>
        <td>Terrier</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbecd6a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9812d0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.5000</td>
        <td>4</td>
        <td>Maltese-Yorkshire Terrier Mix</td>
        <td>None</td>
        <td>Cross Breed</td>
    </tr>
    <tr>
        <td>fdc005ae-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce99358e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.5000</td>
        <td>6</td>
        <td>Poodle</td>
        <td>Non-Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc0758e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9987b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.5000</td>
        <td>6</td>
        <td>Chihuahua-Dachshund Mix</td>
        <td>None</td>
        <td>Cross Breed</td>
    </tr>
    <tr>
        <td>fdc0e91a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce99f906-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.5000</td>
        <td>2</td>
        <td>German Shepherd Dog</td>
        <td>Herding</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc1b5f2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9ac110-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.5000</td>
        <td>2</td>
        <td>Golden Doodle</td>
        <td>None</td>
        <td>Popular Hybrid</td>
    </tr>
    <tr>
        <td>fdc1df64-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9ad4d4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.5000</td>
        <td>4</td>
        <td>Labrador Retriever</td>
        <td>Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc1fb8e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9aeee2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.5000</td>
        <td>2</td>
        <td>Australian Shepherd</td>
        <td>Herding</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc07ad4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce99a654-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.5000</td>
        <td>4</td>
        <td>Cocker Spaniel</td>
        <td>Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc235e0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9b2aba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.5000</td>
        <td>6</td>
        <td>Chihuahua-Jack Russell Terrier Mix</td>
        <td>None</td>
        <td>Cross Breed</td>
    </tr>
    <tr>
        <td>fdbedef4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce982e46-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.4615</td>
        <td>13</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdbec8ce-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce980e3e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.4545</td>
        <td>11</td>
        <td>Basset Hound</td>
        <td>Hound</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc1486a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9a559a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.4286</td>
        <td>7</td>
        <td>Golden Doodle</td>
        <td>None</td>
        <td>Popular Hybrid</td>
    </tr>
    <tr>
        <td>fdbeff06-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9856f0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.4000</td>
        <td>10</td>
        <td>Bearded Collie</td>
        <td>Herding</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbfe22c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce99049c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.4000</td>
        <td>10</td>
        <td>Irish Water Spaniel</td>
        <td>Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbed7ce-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce981dc0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.3333</td>
        <td>18</td>
        <td>Havanese</td>
        <td>Toy</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbfcc60-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98ce0a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.3333</td>
        <td>3</td>
        <td>American Pit Bull Terrier</td>
        <td>None</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc008ce-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce993764-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.3333</td>
        <td>6</td>
        <td>Cavalier King Charles Spaniel-Poodle Mix</td>
        <td>None</td>
        <td>Cross Breed</td>
    </tr>
    <tr>
        <td>fdc15058-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9a5c70-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.3333</td>
        <td>3</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdc1c31c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9ac674-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.3077</td>
        <td>13</td>
        <td>Labrador Retriever</td>
        <td>Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc1e63a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9a4744-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.2857</td>
        <td>7</td>
        <td>Cocker Spaniel</td>
        <td>Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc0c49e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce99d674-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.2778</td>
        <td>18</td>
        <td>Other</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdbf44f2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce987fd6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.2500</td>
        <td>8</td>
        <td>Shetland Sheepdog</td>
        <td>Herding</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc19b30-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9aab30-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.2222</td>
        <td>18</td>
        <td>Brittany</td>
        <td>Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc06a08-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce997c38-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.2000</td>
        <td>5</td>
        <td>Tibetan Spaniel-Papillon Mix</td>
        <td>None</td>
        <td>Cross Breed</td>
    </tr>
    <tr>
        <td>fd681c5e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce471e48-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.0000</td>
        <td>6</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fd427a76-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce25f772-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.0000</td>
        <td>1</td>
        <td>Belgian Malinois</td>
        <td>Herding</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fd6e49da-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce6fadfe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.0000</td>
        <td>1</td>
        <td>West Highland White Terrier</td>
        <td>Terrier</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fd6ea538-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce6fede6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.0000</td>
        <td>3</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdbee1e2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce983fda-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.0000</td>
        <td>1</td>
        <td>Labrador Retriever</td>
        <td>Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbedf94-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce982ed2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.0000</td>
        <td>11</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdbf3930-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9876c6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.0000</td>
        <td>8</td>
        <td>German Shepherd Dog</td>
        <td>Herding</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbf38cc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9877e8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.0000</td>
        <td>1</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdbf6928-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98bd66-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.0000</td>
        <td>2</td>
        <td>Rottweiler</td>
        <td>Working</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbf4e70-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9888f0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.0000</td>
        <td>2</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdbfbd42-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98f0c4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.0000</td>
        <td>4</td>
        <td>Collie</td>
        <td>Herding</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbfbaf4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98ee94-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.0000</td>
        <td>1</td>
        <td>Labrador Retriever</td>
        <td>Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbf96dc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98ce0a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.0000</td>
        <td>6</td>
        <td>American Pit Bull Terrier-American Staffordshire Terrier Mix</td>
        <td>None</td>
        <td>Cross Breed</td>
    </tr>
    <tr>
        <td>fdbed904-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce981eec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.0000</td>
        <td>1</td>
        <td>Australian Shepherd</td>
        <td>Herding</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbfff32-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce992fbc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.0000</td>
        <td>2</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdc018fa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce99454c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.0000</td>
        <td>6</td>
        <td>American Eskimo Dog</td>
        <td>Non-Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc01b48-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce994a74-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.0000</td>
        <td>2</td>
        <td>Pembroke Welsh Corgi</td>
        <td>Herding</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc04a14-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce996446-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.0000</td>
        <td>2</td>
        <td>Cavalier King Charles Spaniel</td>
        <td>Toy</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc06878-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce997b70-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.0000</td>
        <td>1</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdc0c502-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce99d732-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.0000</td>
        <td>15</td>
        <td>Golden Retriever</td>
        <td>Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc0bd28-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce99c9ea-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.0000</td>
        <td>1</td>
        <td>Other</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdc0bf8a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce99cee0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.0000</td>
        <td>2</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdc1236c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9a329a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.0000</td>
        <td>1</td>
        <td>Standard Poodle-English Cocker Spaniel Mix</td>
        <td>None</td>
        <td>Cross Breed</td>
    </tr>
    <tr>
        <td>fdc125ce-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9a3498-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.0000</td>
        <td>1</td>
        <td>Maltese-Silky Terrier Mix</td>
        <td>None</td>
        <td>Cross Breed</td>
    </tr>
    <tr>
        <td>fdc14810-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9a54fa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.0000</td>
        <td>1</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdc19b94-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9aac70-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.0000</td>
        <td>1</td>
        <td>Other</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdc17d9e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9a6d78-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.0000</td>
        <td>1</td>
        <td>Rottweiler-American Pit Bull Terrier Mix</td>
        <td>None</td>
        <td>Cross Breed</td>
    </tr>
    <tr>
        <td>fdc0c930-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce99da20-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.0000</td>
        <td>1</td>
        <td>Basset Hound-Boston Terrier Mix</td>
        <td>None</td>
        <td>Cross Breed</td>
    </tr>
    <tr>
        <td>fdc1fe40-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9af144-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.0000</td>
        <td>1</td>
        <td>Golden Retriever</td>
        <td>Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc20e12-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9b01f2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2.0000</td>
        <td>3</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdbfddfe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce99049c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.9375</td>
        <td>16</td>
        <td>Irish Water Spaniel</td>
        <td>Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fd7104ae-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce7212a6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.9000</td>
        <td>10</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdc0ccf0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce99dcb4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.8571</td>
        <td>7</td>
        <td>Labrador Retriever</td>
        <td>Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc1ed88-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9ae44c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.8571</td>
        <td>7</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdbfb342-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98e732-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.8000</td>
        <td>5</td>
        <td>Cocker Spaniel-Poodle Mix</td>
        <td>None</td>
        <td>Cross Breed</td>
    </tr>
    <tr>
        <td>fdc14126-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9a4be0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.8000</td>
        <td>5</td>
        <td>Irish Setter</td>
        <td>Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc198d8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9a9500-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.8000</td>
        <td>10</td>
        <td>German Shepherd Dog</td>
        <td>Herding</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbe923c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce97f926-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.7647</td>
        <td>17</td>
        <td>Beagle</td>
        <td>Hound</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fd6a7490-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce6cf3fc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.7500</td>
        <td>20</td>
        <td>Golden Retriever</td>
        <td>Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbf5be0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce989ff2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.7500</td>
        <td>4</td>
        <td>Shiba Inu</td>
        <td>Non-Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbfab36-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9810e6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.7500</td>
        <td>4</td>
        <td>Smooth Fox Terrier-Bluetick Coonhound Mix</td>
        <td>None</td>
        <td>Cross Breed</td>
    </tr>
    <tr>
        <td>fdbef5e2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce984f70-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.7000</td>
        <td>10</td>
        <td>Havanese</td>
        <td>Toy</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fd3fd514-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce2261a2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.6667</td>
        <td>3</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdbea952-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9805ce-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.6667</td>
        <td>15</td>
        <td>Golden Doodle</td>
        <td>None</td>
        <td>Popular Hybrid</td>
    </tr>
    <tr>
        <td>fdbf9f10-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98d6de-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.6667</td>
        <td>18</td>
        <td>Australian Cattle Dog-Border Collie Mix</td>
        <td>None</td>
        <td>Cross Breed</td>
    </tr>
    <tr>
        <td>fdbf9542-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98cc98-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.6667</td>
        <td>3</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdc0e186-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce99f244-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.6667</td>
        <td>3</td>
        <td>Boxer-Labrador Retriever Mix</td>
        <td>None</td>
        <td>Cross Breed</td>
    </tr>
    <tr>
        <td>fdc01d96-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce994ca4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.6667</td>
        <td>3</td>
        <td>Doberman Pinscher-Labrador Retriever Mix</td>
        <td>None</td>
        <td>Cross Breed</td>
    </tr>
    <tr>
        <td>fdc0276e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9957ee-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.6250</td>
        <td>8</td>
        <td>Collie</td>
        <td>Herding</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbf686a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98bca8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.6000</td>
        <td>10</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdbf6144-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98b776-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.5455</td>
        <td>11</td>
        <td>American Pit Bull Terrier-Labrador Retriever Mix</td>
        <td>None</td>
        <td>Cross Breed</td>
    </tr>
    <tr>
        <td>fdc1edec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9ae4b0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.5333</td>
        <td>15</td>
        <td>Poodle</td>
        <td>Non-Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fd27dc52-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce138268-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.5000</td>
        <td>2</td>
        <td>Mudi</td>
        <td>None</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fd732310-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce73cb6e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.5000</td>
        <td>6</td>
        <td>West Highland White Terrier</td>
        <td>Terrier</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbef70e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce985092-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.5000</td>
        <td>4</td>
        <td>Border Collie</td>
        <td>Herding</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbf6e28-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce989480-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.5000</td>
        <td>2</td>
        <td>Other</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdbeee8a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9848a4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.5000</td>
        <td>2</td>
        <td>Jack Russell Terrier</td>
        <td>Terrier</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc02ad4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce995ac8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.5000</td>
        <td>4</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdc0b5b2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce99c03a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.5000</td>
        <td>6</td>
        <td>Other</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdbf9bdc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98d3dc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.5000</td>
        <td>4</td>
        <td>Australian Cattle Dog</td>
        <td>Herding</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbf993e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98cfcc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.5000</td>
        <td>2</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdbfa046-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98d896-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.5000</td>
        <td>6</td>
        <td>Maltese-Pomeranian Mix</td>
        <td>None</td>
        <td>Cross Breed</td>
    </tr>
    <tr>
        <td>fdc1e766-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9adeca-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.4286</td>
        <td>7</td>
        <td>Weimaraner</td>
        <td>Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbe7cac-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce97e10c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.3333</td>
        <td>12</td>
        <td>Labrador Retriever</td>
        <td>Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbe7b9e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce97e04e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.3333</td>
        <td>6</td>
        <td>Cairn Terrier</td>
        <td>Terrier</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbf0212-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98598e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.3333</td>
        <td>3</td>
        <td>Australian Cattle Dog</td>
        <td>Herding</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbf5460-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce988f9e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.3333</td>
        <td>3</td>
        <td>Standard Poodle</td>
        <td>Non-Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc017d8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce99448e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.3333</td>
        <td>3</td>
        <td>Catahoula Leopard Dog-Labrador Retriever Mix</td>
        <td>None</td>
        <td>Cross Breed</td>
    </tr>
    <tr>
        <td>fdbfbc16-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98efac-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.3333</td>
        <td>3</td>
        <td>Golden Retriever-Standard Poodle Mix</td>
        <td>None</td>
        <td>Cross Breed</td>
    </tr>
    <tr>
        <td>fdc077d2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce999272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.2857</td>
        <td>7</td>
        <td>Siberian Husky-German Shepherd Dog Mix</td>
        <td>None</td>
        <td>Cross Breed</td>
    </tr>
    <tr>
        <td>fdbed0e4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce981514-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.2500</td>
        <td>4</td>
        <td>Cairn Terrier-Australian Terrier Mix</td>
        <td>None</td>
        <td>Cross Breed</td>
    </tr>
    <tr>
        <td>fdbe8a9e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce97f1f6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.2500</td>
        <td>16</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdc05874-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce996b3a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.2500</td>
        <td>4</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdc1397e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9a4136-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.2500</td>
        <td>4</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdbfe56a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce99049c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.2143</td>
        <td>14</td>
        <td>Irish Water Spaniel</td>
        <td>Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc08150-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce99ae74-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.1667</td>
        <td>6</td>
        <td>Basenji</td>
        <td>Hound</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbe8530-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce97e97c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.0000</td>
        <td>2</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdbf3b24-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98637a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.0000</td>
        <td>2</td>
        <td>Jindo-Shiba Inu Mix</td>
        <td>None</td>
        <td>Cross Breed</td>
    </tr>
    <tr>
        <td>fdbf4d9e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce988832-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.0000</td>
        <td>1</td>
        <td>Golden Retriever</td>
        <td>Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbe9412-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce97faa2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.0000</td>
        <td>2</td>
        <td>Leonberger</td>
        <td>Working</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbf3f20-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce987cc0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.0000</td>
        <td>2</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdbf9132-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98c7ca-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.0000</td>
        <td>1</td>
        <td>Labrador Retriever</td>
        <td>Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbf97a4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98ce6e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.0000</td>
        <td>6</td>
        <td>Labrador Retriever-Airedale Terrier Mix</td>
        <td>None</td>
        <td>Cross Breed</td>
    </tr>
    <tr>
        <td>fdbfb590-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98e908-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.0000</td>
        <td>3</td>
        <td>Golden Doodle</td>
        <td>None</td>
        <td>Popular Hybrid</td>
    </tr>
    <tr>
        <td>fdc07228-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9980fc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.0000</td>
        <td>1</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdc086f0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce99b112-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.0000</td>
        <td>1</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdbfabd6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98e228-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.0000</td>
        <td>2</td>
        <td>Dalmatian</td>
        <td>Non-Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc0a3ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce99bce8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.0000</td>
        <td>2</td>
        <td>German Shorthaired Pointer-Pointer Mix</td>
        <td>None</td>
        <td>Cross Breed</td>
    </tr>
    <tr>
        <td>fdc11d90-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9a2caa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.0000</td>
        <td>1</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdbeaa1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce980628-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.0000</td>
        <td>2</td>
        <td>Yorkshire Terrier</td>
        <td>Toy</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc12f74-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9a3a88-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.0000</td>
        <td>1</td>
        <td>Siberian Husky-German Shepherd Dog Mix</td>
        <td>None</td>
        <td>Cross Breed</td>
    </tr>
    <tr>
        <td>fdc14748-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9a53ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.0000</td>
        <td>2</td>
        <td>Border Collie</td>
        <td>Herding</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc12ed4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9a3a1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.0000</td>
        <td>7</td>
        <td>Wire Fox Terrier</td>
        <td>Terrier</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc15256-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9a5e6e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.0000</td>
        <td>1</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdc18244-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9a74bc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.0000</td>
        <td>3</td>
        <td>Poodle</td>
        <td>Non-Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc13c30-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9a460e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.0000</td>
        <td>2</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdc11a5c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9a2a48-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.0000</td>
        <td>1</td>
        <td>Mastiff</td>
        <td>Working</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc1f24c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9ae7da-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.0000</td>
        <td>4</td>
        <td>Other</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdc12a24-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9a38ee-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.0000</td>
        <td>1</td>
        <td>Australian Shepherd</td>
        <td>Herding</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc20886-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9afea0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1.0000</td>
        <td>1</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdbe8044-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce97e2ec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.8571</td>
        <td>7</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdc1a2ec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9ab1f2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.8571</td>
        <td>7</td>
        <td>Other</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdbf93a8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98cac2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.8333</td>
        <td>6</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdbf0456-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce985d9e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.7500</td>
        <td>16</td>
        <td>English Springer Spaniel</td>
        <td>Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbfb90a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98ecb4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.6667</td>
        <td>3</td>
        <td>I Don&#x27;t Know</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdc016b6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce994376-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.6667</td>
        <td>3</td>
        <td>Doberman Pinscher</td>
        <td>Working</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc015f8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9942c2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.6667</td>
        <td>3</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fd6fd44e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce710154-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.5000</td>
        <td>2</td>
        <td>Beagle-Russell Terrier Mix</td>
        <td>None</td>
        <td>Cross Breed</td>
    </tr>
    <tr>
        <td>fdc07d22-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce99a9a6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.5000</td>
        <td>4</td>
        <td>Brittany</td>
        <td>Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc055cc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9969be-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.5000</td>
        <td>2</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdc1cc54-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9ac7fa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.5000</td>
        <td>2</td>
        <td>I Don&#x27;t Know</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdc23ba8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9b306e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.4211</td>
        <td>19</td>
        <td>Cockapoo</td>
        <td>None</td>
        <td>Popular Hybrid</td>
    </tr>
    <tr>
        <td>fdbec040-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98098e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.4000</td>
        <td>10</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdbf506e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce988b2a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.3333</td>
        <td>3</td>
        <td>I Don&#x27;t Know</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdc1351e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9a3e0c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.3333</td>
        <td>3</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdc07d7c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce99aa64-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.3333</td>
        <td>3</td>
        <td>Border Collie</td>
        <td>Herding</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fd439f3c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce26de3a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>1</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fd450d2c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce28392e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>4</td>
        <td>Wirehaired Pointing Griffon</td>
        <td>Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fd6eb0aa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce701492-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>1</td>
        <td>Nova Scotia Duck Tolling Retriever-Chihuahua Mix</td>
        <td>None</td>
        <td>Cross Breed</td>
    </tr>
    <tr>
        <td>fd754000-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce758990-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>1</td>
        <td>Soft Coated Wheaten Terrier</td>
        <td>Terrier</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbe78ec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce97df36-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>1</td>
        <td>Golden Retriever</td>
        <td>Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbe86fc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce97eb66-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>1</td>
        <td>Collie</td>
        <td>Herding</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbe9d04-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce97febc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>1</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdbed396-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9816fe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>1</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdbecc48-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce981208-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>2</td>
        <td>English Cocker Spaniel</td>
        <td>Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbed760-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce981d66-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>1</td>
        <td>Chihuahua</td>
        <td>Toy</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbe844a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce97e6a2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>1</td>
        <td>German Shepherd Dog</td>
        <td>Herding</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbf259e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce986af0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>1</td>
        <td>Pomeranian-Chihuahua Mix</td>
        <td>None</td>
        <td>Cross Breed</td>
    </tr>
    <tr>
        <td>fdbecf5e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9813f2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>1</td>
        <td>Miniature Schnauzer</td>
        <td>Terrier</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbf0578-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce985eca-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>2</td>
        <td>Dachshund</td>
        <td>Hound</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbf35a2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce987540-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>4</td>
        <td>Redbone Coonhound</td>
        <td>Hound</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbf3ac0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98796e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>1</td>
        <td>Great Dane</td>
        <td>Working</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbe74d2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce97dd60-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>1</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdbf61a8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98b71c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>3</td>
        <td>Border Collie-American Pit Bull Terrier Mix</td>
        <td>None</td>
        <td>Cross Breed</td>
    </tr>
    <tr>
        <td>fdbf6ca2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98be88-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>2</td>
        <td>Rottweiler</td>
        <td>Working</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbf8ec6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98c590-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>1</td>
        <td>Great Dane</td>
        <td>Working</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbf92d6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98c9a0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>2</td>
        <td>German Shepherd Dog</td>
        <td>Herding</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbfb96e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98ed18-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>1</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdbfcf58-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98ff9c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>1</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdbfb27a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98e674-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>2</td>
        <td>Wirehaired Pointing Griffon</td>
        <td>Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbf98da-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98cec8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>1</td>
        <td>Labrador Retriever</td>
        <td>Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc00b1c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce993930-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>1</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdbf080c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce985f2e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>1</td>
        <td>Nova Scotia Duck Tolling Retriever</td>
        <td>Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc0217e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9952da-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>1</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdc01fda-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce994fec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>1</td>
        <td>I Don&#x27;t Know</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdc026a6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9956cc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>3</td>
        <td>Pug</td>
        <td>Toy</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbe9ed0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9800f6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>1</td>
        <td>Boston Terrier</td>
        <td>Non-Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbfca44-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98fd4e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>1</td>
        <td>German Shepherd Dog</td>
        <td>Herding</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbfa488-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98dd8c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>1</td>
        <td>Standard Poodle</td>
        <td>Non-Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc05fb8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce997292-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>2</td>
        <td>Golden Doodle</td>
        <td>None</td>
        <td>Popular Hybrid</td>
    </tr>
    <tr>
        <td>fdc06e18-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce997e86-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>1</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdc03060-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce995eba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>1</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdc07b38-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce99a834-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>4</td>
        <td>Dutch Shepherd</td>
        <td>None</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbf9fe2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98d80a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>2</td>
        <td>Shih Tzu</td>
        <td>Toy</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc05f54-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce99722e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>1</td>
        <td>Irish Wolfhound-Mastiff Mix</td>
        <td>None</td>
        <td>Cross Breed</td>
    </tr>
    <tr>
        <td>fdc0b6e8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce99c152-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>1</td>
        <td>Chihuahua-Dachshund Mix</td>
        <td>None</td>
        <td>Cross Breed</td>
    </tr>
    <tr>
        <td>fdc0ba08-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce99c6de-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>1</td>
        <td>Bulldog</td>
        <td>Non-Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc0bbfc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce99c85a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>1</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdbf65a4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98bad2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>1</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdbf4894-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce988468-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>1</td>
        <td>Boxer</td>
        <td>Working</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbece96-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce981334-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>1</td>
        <td>Rat Terrier-Beagle Mix</td>
        <td>None</td>
        <td>Cross Breed</td>
    </tr>
    <tr>
        <td>fdc07a0c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce99a71c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>1</td>
        <td>Alaskan Malamute</td>
        <td>Working</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc0bb34-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce97a6b0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>1</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdc0ed02-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce99fc44-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>2</td>
        <td>Cavalier King Charles Spaniel</td>
        <td>Toy</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc075e8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce998c64-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>4</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdc01eb8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce994df8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>1</td>
        <td>Airedale Terrier</td>
        <td>Terrier</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc115fc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9a26b0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>1</td>
        <td>Labrador Retriever</td>
        <td>Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc12696-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9a34fc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>4</td>
        <td>St. Bernard</td>
        <td>None</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc1306e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9a3aec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>1</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdc13640-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce958a74-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>1</td>
        <td>Shetland Sheepdog</td>
        <td>Herding</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc12e34-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9a39b6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>1</td>
        <td>Standard Poodle</td>
        <td>Non-Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc0d2d6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce99e11e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>2</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdbf9f74-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98d774-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>1</td>
        <td>Bedlington Terrier</td>
        <td>Terrier</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc0d56a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce99e2f4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>1</td>
        <td>I Don&#x27;t Know</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdc11c64-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9a2be2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>1</td>
        <td>Giant Schnauzer</td>
        <td>Working</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc1993c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9a9596-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>1</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdc1a6c0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9ab576-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>1</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdc1dbfe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9acfa2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>1</td>
        <td>Pomeranian</td>
        <td>Toy</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbfb5f4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98e962-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>1</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdc1e090-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9ad6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>2</td>
        <td>Standard Poodle</td>
        <td>Non-Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc07890-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9997c2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>2</td>
        <td>Border Collie-Cocker Spaniel Mix</td>
        <td>None</td>
        <td>Cross Breed</td>
    </tr>
    <tr>
        <td>fdc1e702-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9ade02-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>1</td>
        <td>Maltese</td>
        <td>Toy</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc16750-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9a677e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>1</td>
        <td>Boston Terrier</td>
        <td>Non-Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc1b58e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9ac0ac-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>1</td>
        <td>Labradoodle</td>
        <td>None</td>
        <td>Popular Hybrid</td>
    </tr>
    <tr>
        <td>fdbfd624-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9902b2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>1</td>
        <td>Poodle-Soft Coated Wheaten Terrier Mix</td>
        <td>None</td>
        <td>Cross Breed</td>
    </tr>
    <tr>
        <td>fdc1e392-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9adb3c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>4</td>
        <td>Poodle-Yorkshire Terrier Mix</td>
        <td>None</td>
        <td>Cross Breed</td>
    </tr>
    <tr>
        <td>fdc1f40e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9ae906-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>3</td>
        <td>Australian Shepherd</td>
        <td>Herding</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc0b486-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce99bf22-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>4</td>
        <td>Other</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdc1dc62-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9ad006-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>2</td>
        <td>Boston Terrier</td>
        <td>Non-Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc01e54-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce994d62-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0.0000</td>
        <td>1</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fd45992c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce287fba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>0</td>
        <td>Black Russian Terrier</td>
        <td>Working</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fd6a72e2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce6cf370-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>0</td>
        <td>German Shepherd Dog</td>
        <td>Herding</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbe99d0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce97fdfe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>0</td>
        <td>Flat-Coated Retriever</td>
        <td>Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbef07e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce984a84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>0</td>
        <td>Chesapeake Bay Retriever-Labrador Retriever Mix</td>
        <td>None</td>
        <td>Cross Breed</td>
    </tr>
    <tr>
        <td>fdbe7630-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce97de1e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>0</td>
        <td>Rhodesian Ridgeback</td>
        <td>Hound</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbf4588-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce97e166-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>0</td>
        <td>Labrador Retriever</td>
        <td>Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbf56c2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce988eea-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>0</td>
        <td>Tibetan Terrier</td>
        <td>Non-Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbef9c0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce985330-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>0</td>
        <td>Miniature Pinscher</td>
        <td>Toy</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbef45c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce984656-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>0</td>
        <td>Australian Shepherd-Border Collie Mix</td>
        <td>None</td>
        <td>Cross Breed</td>
    </tr>
    <tr>
        <td>fdbf762a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98c176-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>0</td>
        <td>Border Collie</td>
        <td>Herding</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbeec32-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9846ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>0</td>
        <td>Labrador Retriever</td>
        <td>Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbf3070-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce986fdc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>0</td>
        <td>Labrador Retriever</td>
        <td>Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbf0aaa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce986154-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>0</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdbef6aa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98502e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>0</td>
        <td>Miniature Schnauzer</td>
        <td>Terrier</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbf0e88-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9862da-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>0</td>
        <td>Golden Retriever</td>
        <td>Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbff424-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce99096a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>0</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdbfc972-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98fa38-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>0</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdc01594-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce99425e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>0</td>
        <td>Border Collie</td>
        <td>Herding</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc001d0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9932a0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>0</td>
        <td>Jack Russell Terrier</td>
        <td>Terrier</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc01df0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce994cfe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>0</td>
        <td>Boston Terrier</td>
        <td>Non-Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc01530-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce994204-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>0</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdc02034-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce995118-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>0</td>
        <td>Australian Shepherd</td>
        <td>Herding</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdbfcc06-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce98ff38-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>0</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdc025de-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9955aa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>0</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdbf55fa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce986708-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>0</td>
        <td>Poodle</td>
        <td>Non-Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc06454-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9978dc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>0</td>
        <td>Portuguese Water Dog</td>
        <td>Working</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc08506-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9978dc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>0</td>
        <td>Portuguese Water Dog</td>
        <td>Working</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc0cf52-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce99de30-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>0</td>
        <td>Great Dane-Bullmastiff Mix</td>
        <td>None</td>
        <td>Cross Breed</td>
    </tr>
    <tr>
        <td>fdc0be5e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce99cbe8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>0</td>
        <td>Boston Terrier</td>
        <td>Non-Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc0edd4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce99fd0c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>0</td>
        <td>Border Collie-Labrador Retriever Mix</td>
        <td>None</td>
        <td>Cross Breed</td>
    </tr>
    <tr>
        <td>fdbf320a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9871c6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>0</td>
        <td>Cocker Spaniel-Bichon Frise Mix</td>
        <td>None</td>
        <td>Cross Breed</td>
    </tr>
    <tr>
        <td>fdc178ee-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9a6972-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>0</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdbf578a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9894da-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>0</td>
        <td>Poodle-Golden Retriever Mix</td>
        <td>None</td>
        <td>Cross Breed</td>
    </tr>
    <tr>
        <td>fdc18be0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9a7ebc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>0</td>
        <td>Labrador Retriever-Border Collie Mix</td>
        <td>None</td>
        <td>Cross Breed</td>
    </tr>
    <tr>
        <td>fdc1a472-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9a07a2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>0</td>
        <td>Poodle</td>
        <td>Non-Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc1fddc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce88940e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>0</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>fdc20a0c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9b001c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>0</td>
        <td>French Bulldog</td>
        <td>Non-Sporting</td>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>fdc1f940-7144-11e5-ba71-058fbc01cf0b</td>
        <td>ce9aec80-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>0</td>
        <td>Mixed</td>
        <td>None</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
</table>



It's clear from looking at this output that (A) not many customers provided ratings, and (B) when they did, they usually were not very surprised by their dog's performance.  Therefore, these ratings are probably not going to provide a lot of instructive insight into how to improve Dognition's completion rate.  However, the ratings table still provides a great opportunity to illustrate the results of different types of joins.  

To help prepare us for this:

**Questions 1-4:  How many unique dog_guids and user_guids are there in the reviews and dogs table independently?**  


```python
%%sql
SELECT COUNT(DISTINCT dog_guid)
FROM dogs;
```

    1 rows affected.





<table>
    <tr>
        <th>COUNT(DISTINCT dog_guid)</th>
    </tr>
    <tr>
        <td>35050</td>
    </tr>
</table>




```python
%%sql
SELECT COUNT(DISTINCT user_guid)
FROM dogs;
```

    1 rows affected.





<table>
    <tr>
        <th>COUNT(DISTINCT user_guid)</th>
    </tr>
    <tr>
        <td>30967</td>
    </tr>
</table>




```python
%%sql
SELECT COUNT(DISTINCT dog_guid)
FROM reviews;
```

    1 rows affected.





<table>
    <tr>
        <th>COUNT(DISTINCT dog_guid)</th>
    </tr>
    <tr>
        <td>5991</td>
    </tr>
</table>




```python
%%sql
SELECT COUNT(DISTINCT user_guid)
FROM reviews;
```

    1 rows affected.





<table>
    <tr>
        <th>COUNT(DISTINCT user_guid)</th>
    </tr>
    <tr>
        <td>5586</td>
    </tr>
</table>



These counts indicate some important things:

+ Many customers in both the reviews and the dogs table have multiple dogs
+ There are many more unique dog_guids and user_guids in the dogs table than the reviews table
+ There are many more unique dog_guids and user_guids in the reviews table than in the output of our inner join

Let's test one more thing.  
   
**Try the inner join query once with just the dog_guid or once with just the user_guid clause in the WHERE statement:**




```python
%%sql
SELECT DISTINCT d.dog_guid
FROM dogs d, reviews r
WHERE d.dog_guid = r.dog_guid;
```

    395 rows affected.





<table>
    <tr>
        <th>dog_guid</th>
    </tr>
    <tr>
        <td>fd27dc52-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fd3fd514-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fd41bca8-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fd427a76-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fd439f3c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fd442826-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fd450d2c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fd45992c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fd675a08-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fd681c5e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fd684cf6-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fd684d5a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fd6913ac-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fd6a5654-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fd6a6590-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fd6a72e2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fd6a73b4-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fd6a7490-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fd6aabd6-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fd6abc8e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fd6e49da-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fd6ea538-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fd6eb0aa-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fd6f1360-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fd6fd44e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fd702994-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fd7104ae-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fd732310-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fd754000-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbe74d2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbe7630-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbe78ec-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbe7a40-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbe7b9e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbe7cac-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbe7d56-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbe8044-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbe81a2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbe8210-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbe844a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbe8530-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbe86fc-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbe8774-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbe8a9e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbe8c6a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbe923c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbe9412-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbe98e0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbe99d0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbe9d04-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbe9d86-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbe9e08-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbe9ed0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbea3c6-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbea542-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbea952-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbeaa1a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbeaf38-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbec040-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbec8ce-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbecc48-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbecd6a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbece96-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbecf5e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbed0e4-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbed148-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbed396-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbed6f2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbed760-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbed7ce-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbed904-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbed968-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbedb70-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbedd14-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbedef4-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbedf94-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbee1e2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbee278-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbeec32-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbeedc2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbeee8a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbeef52-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbef07e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbef2cc-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbef330-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbef45c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbef5e2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbef6aa-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbef70e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbef9c0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbeff06-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf008c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf0212-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf0456-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf0578-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf080c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf0aaa-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf0e2e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf0e88-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf178e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf1bda-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf259e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf2d3c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf2ee0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf2fa8-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf3070-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf320a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf35a2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf3732-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf38cc-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf3930-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf39f8-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf3ac0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf3b24-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf3d86-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf3f20-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf4196-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf44f2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf4588-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf4894-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf4d9e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf4e70-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf506e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf5460-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf55fa-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf56c2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf578a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf5be0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf5eb0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf6144-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf61a8-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf62de-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf65a4-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf66e4-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf686a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf6928-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf698c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf6ca2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf6e28-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf735a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf762a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf7b20-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf8598-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf8ec6-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf9132-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf92d6-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf93a8-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf940c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf94de-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf9542-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf96dc-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf9740-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf97a4-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf98da-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf993e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf9bdc-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf9d76-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf9f10-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf9f74-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbf9fe2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbfa046-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbfa488-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbfa974-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbfab36-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbfabd6-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbfb0ea-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbfb1b2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbfb27a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbfb342-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbfb590-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbfb5f4-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbfb77a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbfb90a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbfb96e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbfbaf4-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbfbbb2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbfbc16-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbfbd42-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbfc044-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbfc0b2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbfc972-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbfca44-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbfcc06-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbfcc60-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbfcf58-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbfd11a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbfd5ca-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbfd624-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbfddfe-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbfe22c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbfe56a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbff302-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbff3c0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbff424-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbff730-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbff852-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbffdac-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdbfff32-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc001d0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc00234-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc005ae-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc008ce-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc00b1c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc00f86-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc010b2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc013aa-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc0140e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc01472-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc01530-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc01594-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc015f8-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc016b6-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc0177e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc017d8-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc018fa-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc019c2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc01b48-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc01c06-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc01d96-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc01df0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc01e54-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc01eb8-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc01f76-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc01fda-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc02034-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc0217e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc021e2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc025de-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc026a6-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc0276e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc02ad4-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc03060-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc0345c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc03862-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc04a14-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc05126-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc0518a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc05504-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc055cc-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc057ac-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc05874-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc059fa-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc05f54-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc05fb8-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc06454-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc067ce-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc06878-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc06a08-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc06bca-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc06d82-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc06e18-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc070fc-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc07228-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc072e6-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc0758e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc075e8-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc0776e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc077d2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc07890-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc07a0c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc07ad4-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc07b38-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc07d22-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc07d7c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc07f66-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc08150-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc0827c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc08506-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc086f0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc09a82-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc09b0e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc0a3ba-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc0b486-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc0b5b2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc0b6e8-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc0b878-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc0ba08-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc0bb34-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc0bbfc-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc0bd28-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc0be5e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc0bec2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc0bf8a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc0c2aa-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc0c49e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc0c502-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc0c930-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc0ccf0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc0cd90-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc0cebc-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc0cf52-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc0d2d6-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc0d56a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc0d75e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc0de66-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc0e186-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc0e91a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc0ed02-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc0edd4-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc0ef00-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc0fdb0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc1021a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc115fc-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc11a5c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc11c64-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc11d90-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc1204c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc121dc-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc1236c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc125ce-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc12696-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc12a24-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc12d62-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc12e34-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc12ed4-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc12f74-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc1306e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc13104-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc1351e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc13640-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc1397e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc139e2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc13c30-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc14126-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc14248-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc14748-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc147ac-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc14810-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc1486a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc15058-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc1517a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc15256-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc159b8-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc16750-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc16dae-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc1706a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc178ee-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc17ce0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc17d9e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc180be-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc18244-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc188ca-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc1892e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc18be0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc18f78-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc191f8-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc19554-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc198d8-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc1993c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc19b30-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc19b94-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc1a224-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc1a288-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc1a2ec-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc1a3b4-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc1a472-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc1a6c0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc1ab52-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc1b160-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc1b228-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc1b58e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc1b5f2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc1baca-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc1c31c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc1c880-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc1cc54-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc1d0a0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc1db36-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc1dbfe-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc1dc62-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc1df64-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc1e090-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc1e1a8-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc1e392-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc1e45a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc1e63a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc1e702-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc1e766-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc1ea18-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc1ed88-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc1edec-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc1f24c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc1f40e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc1f940-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc1fb8e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc1fddc-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc1fe40-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc20764-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc20886-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc20a0c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc20e12-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc21060-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc235e0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc23ba8-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fdc2417a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
</table>




```python
%%sql
SELECT DISTINCT d.user_guid
FROM dogs d, reviews r
WHERE d.user_guid = r.user_guid;
```

    5586 rows affected.





<table>
    <tr>
        <th>user_guid</th>
    </tr>
    <tr>
        <td>ce134492-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce137868-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce137f20-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce137fca-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce138268-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce1385c4-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce1387cc-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce2232cc-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce224208-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce224cee-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce2261a2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce226576-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce23ec8e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce242e74-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce2434e6-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce24476a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce244fd0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce2461dc-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce24a4ee-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce24c50a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce24c8a2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce24d130-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce250e48-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce2514c4-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce251690-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce251af0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce252c98-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce253cf6-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce254534-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce25482c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce2554ca-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce257d2e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce2581a2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce258896-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce25a466-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce25a6f0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce25b9d8-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce25c2ac-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce25f0c4-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce25f772-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce25f9a2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce25fac4-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce260168-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce261aa4-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce261bd0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce261edc-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce262364-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce264330-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce2673f0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce267e18-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce2684b2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce268c1e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce26ad84-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce26b266-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce26de3a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce26edbc-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce27023e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce272124-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce275612-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce275676-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce275f22-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce277962-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce277e30-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce278236-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce27928a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce279956-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce27c0ac-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce27c250-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce27cf02-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce27e262-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce27e910-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce27eb2c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce27ef6e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce28392e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce284536-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce284892-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce284b1c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce284e5a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce28623c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce286868-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce287fba-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce28806e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce28c150-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce28c2cc-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce28e3ce-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce28e61c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce28eaa4-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce28ec8e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce28f2b0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce28fd1e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce28ff08-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce291880-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce292b54-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce2934dc-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce296f88-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce297870-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce29818a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce298a72-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce298d42-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce29a43a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce29c21c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce29d4d2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce29d590-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce29e4b8-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce2a0db2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce2a45c0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce2a5b28-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce2a67b2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce2a787e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce2a7946-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce2ab050-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce2acd7e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce2ad544-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce2ade04-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce2ae0f2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce2aedcc-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce2af330-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce2b0082-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce2b9a42-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce2cc566-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce2d667e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce2d7c90-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce2d9108-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce2dad5a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce2dc6f0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce2de248-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce2e1434-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce2e3ed2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce2e6eac-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce311486-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce312d22-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce31f162-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce324fc2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce325b66-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce32991e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce32b6a6-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce32bf5c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce32ccf4-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce32db4a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce32e0ea-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce32fd00-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce3315ba-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce332d20-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce333270-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce334a08-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce335868-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce3371b8-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce337f1e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce338b58-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce33b2e0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce359c72-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce370ef4-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce39a9ac-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce39c02c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce3a07c6-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce3a69f0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce3a80de-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce3a8ca0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce3c42ca-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce3c4c52-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce3c856e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce3ca01c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce3ca79c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce3cb732-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce3d28b6-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce3d291a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce3d42d8-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce3d4846-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce3d4bd4-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce3d4fb2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce3da412-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce3db222-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce3e2b80-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce3e38be-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce3e54f2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce3e59a2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce3ecbe4-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce3ed8fa-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce3edb98-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce3edf9e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce3ee26e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce3ee5ca-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce3ee692-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce3f01e0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce3f044c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce3f0be0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce3f21ca-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce3f22d8-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce3f2846-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce3f41d2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce3f4286-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce3f4344-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce3f5334-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce3f5500-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce3f58a2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce3f5e6a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce3f65c2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce3f69d2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce3f6d06-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce3f6efa-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce3f8a7a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce3f8e1c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce3f963c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce3f9768-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce3fa776-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce3fc85a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce3fd2aa-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce3fe2d6-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce3feede-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce3ff7a8-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce3fff28-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce3fff8c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce400518-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce400ca2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce400d6a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce4016de-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce401e2c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce4027a0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce4031be-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce403b50-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce405ae0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce405f90-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce4060b2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce406850-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce4073b8-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce407b4c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce4082ae-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce409316-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce409956-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce40a5f4-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce40b71a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce40c610-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce40cd4a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce40d2b8-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce40d31c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce40e3f2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce40e79e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce40fda6-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce40ff86-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce410756-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce4107ba-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce410936-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce411200-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce411980-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce411dcc-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce411fd4-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce4125b0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce41272c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce412c90-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce415a26-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce415c56-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce416692-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce4166f6-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce4177a4-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce417934-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce417c2c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce418258-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce4182b2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce4185b4-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce418adc-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce418d2a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce41907c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce419310-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce419522-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce45833a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce458498-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce45974e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce45996a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce45ac98-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce46c308-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce46d7e4-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce46e43c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce46e6da-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce46e7e8-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce46e86a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce46eeb4-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce46f60c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce46fa44-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce46fe7c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce470098-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce4702be-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce470340-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce4705e8-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce47099e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce470bb0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce470d5e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce470e30-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce470f02-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce47131c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce4714c0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce47165a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce47172c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce471a74-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce471b46-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce471e48-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce471eca-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce472064-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce47228a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce472398-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce472424-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce4725be-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce4726cc-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce4727e4-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce472866-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce472e42-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce47327a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce473388-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce473496-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce4737f2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce473856-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce4738c4-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce4739a0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce473a0e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce473bc6-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce473c34-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce473d1a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce474026-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce47410c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce474260-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce4742ce-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce474418-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce4744f4-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce4745d0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce4746b6-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce474724-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce47480a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce47486e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce474954-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce474b20-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce474cce-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce474e04-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce474e72-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce475232-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce47530e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce4755ac-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce476042-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce476178-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce4766c8-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce476eb6-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce477afa-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce477bf4-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce477c62-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce477cda-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce477e24-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce477e92-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce477f00-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce478054-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce4780c2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce47819e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce47827a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce4782e8-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce478360-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce4783ce-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce47843c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce4784aa-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce478518-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce478586-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce4785f4-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce478748-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce4787b6-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce478900-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce47896e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce478a54-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce478ac2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce5c117c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce5c12d0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce5c1410-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce5c15c8-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce5c16ea-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce5c1802-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce5c188e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce5c1b54-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce5c1d84-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce5c1e10-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce5c1e9c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce5c2036-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce5c214e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce5c21da-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce5c2374-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce5c2482-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce5c25e0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce662090-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6621e4-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66228e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6624f0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66259a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6629fa-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce662aea-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce662cb6-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce662e78-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce662f54-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66303a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66324c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce663364-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6633fa-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce663486-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66351c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6635b2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66363e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66399a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce663a26-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce663ab2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce663bca-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce663c60-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce663e0e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce663e9a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce663fbc-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce664160-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66430e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6644bc-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce664548-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6645d4-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce664778-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce664804-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66491c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce664a34-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce664aca-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce664be2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce664c6e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce664d86-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce664e12-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce664fb6-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6650ce-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6651e6-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66539e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66554c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6659a2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce665a4c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce665ad8-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce665c90-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce665d30-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce666122-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6661b8-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6662da-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce666492-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66651e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce666762-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6667ee-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66687a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce666a28-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce666abe-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce666b86-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce666d2a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce666e38-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce666eba-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce667040-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6670c2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66796e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce667a86-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce668058-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6688c8-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66899a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce668f44-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66900c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce669e08-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66a9ca-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66aa6a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66aaec-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66ab6e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66b096-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66b122-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66b244-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66b2c6-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66b460-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66b578-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66b5fa-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66b794-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66b8a2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66b92e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66b9b0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66ba3c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66bc62-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66bcee-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66bd70-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66bf0a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66c09a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66c450-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66c5b8-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66c7de-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66c86a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66c982-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66ce3c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66cec8-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66cf4a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66d36e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66d59e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66d710-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66d7e2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66df26-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66e282-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66e35e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66e610-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66ea52-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66ebe2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66ed7c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66ee08-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66ef20-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66efac-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66f1c8-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66f466-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66f4e8-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66f56a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66f5f6-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66f844-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66fb82-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66fc0e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66fd1c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66fda8-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66feb6-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66ff42-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce66ffce-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6700e6-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce670280-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce67041a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6704a6-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6705b4-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce670d3e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce670e56-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce671220-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6713ba-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce671446-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce67166c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce67178e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6719a0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce671aa4-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce671b9e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce67210c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce67218e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce672206-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6729ea-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce672ac6-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce673174-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6733ae-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce67393a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6739f8-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce673e8a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce675050-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6750dc-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce67515e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce675276-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6752f8-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce675410-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce675492-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6755a0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce67562c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6758ca-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce675b68-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce675bea-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce675cf8-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce675e88-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce675f96-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce676126-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6761b2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce676234-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce676342-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce676450-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce676662-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce676cfc-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce676f22-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6771d4-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6779fe-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce677b5c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce677c06-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce677e7c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce677ef4-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6780ac-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce678188-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce678494-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6c2756-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6c2b70-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6c2dc8-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6c2f3a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6c2ff8-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6c3250-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6c330e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6c33cc-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6c376e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6c39a8-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6c3c00-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6c3cbe-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6c3d7c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6c3fc0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6c4664-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6c4722-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6c4e2a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6c4ee8-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6c505a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6c5122-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6c51d6-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6c5294-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6c56f4-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6c57d0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6c5b7c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6c5e88-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6c62b6-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6c6536-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6c663a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6c6720-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6c6806-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6c6aa4-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6c6c5c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6c6dc4-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6c6e46-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6c72ce-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6c735a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6c762a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6c7c92-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6c7d8c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6c82e6-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6c8458-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6c852a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6c8aa2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6c9006-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6ca690-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6ca898-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6ca924-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6ca9ba-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6cac80-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6cb054-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6cb1f8-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6cb4be-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6cb54a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6cb89c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6cbdf6-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6cc5d0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6cc760-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6cc7e2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6cc8f0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6cca8a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6ccc1a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6ccd3c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6cce4a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6cd480-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6cd624-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6cd804-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6cd912-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6cdaac-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6cdeda-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6ce394-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6ce524-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6ce5a6-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6ce632-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6ce736-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6ced80-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6cee8e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6cf01e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6cf0a0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6cf2ee-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6cf370-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6cf3fc-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6cf47e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6cf730-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6cfa96-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6cfb2c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6cfc3a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6cfe56-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d0324-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d03a6-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d0428-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d0536-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d064e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d06d0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d0996-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d0a18-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d0b94-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d0c16-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d0d10-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d13a0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d1abc-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d28ae-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d2cf0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d2dae-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d37ae-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d4078-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d4488-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d4514-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d4622-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d4956-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d49e2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d4b72-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d4c80-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d4e9c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d4faa-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d502c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d50b8-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d51c6-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d53e2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d55fe-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d570c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d581a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d589c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d59aa-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d5a2c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d5aae-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d5b3a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d5cd4-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d5d88-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d5ea0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d62e2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d6364-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d66b6-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d6756-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d6832-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d6f94-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d7142-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d7214-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d72f0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d73c2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d7642-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d77aa-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d7836-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d7c6e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d7cfa-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d7e08-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d7e8a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d8024-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d8132-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d83c6-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d84d4-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d870e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d879a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d881c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d892a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d89b6-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d8ccc-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d8d62-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d8ef2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d8f7e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d9118-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d91a4-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d94e2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d9960-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d9a82-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d9b04-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6d9ca8-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6da068-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6da180-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6da31a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6da3a6-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6da64e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6da752-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6da7de-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6da856-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6db0bc-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6dbc88-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6dc6c4-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6dc75a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6dd952-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6dda2e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6ddbf0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6de046-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6de122-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6de190-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6de348-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6de3b6-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6de42e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6de87a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6de956-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6dea3c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6decd0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6dee24-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6dee92-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6def00-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6df054-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6df216-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6df2f2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6df66c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6df78e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6df860-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6dfb8a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6dfcf2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6dfe5a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6dff0e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e0116-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e01ca-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e027e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e0332-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e0418-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e05d0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e0648-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e06b6-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e0792-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e0b8e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e0c6a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e0e9a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e0fe4-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e1052-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e119c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e1214-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e143a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e1516-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e1656-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e17a0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e1a34-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e1b1a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e1c00-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e1c78-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e1fb6-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e201a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e2100-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e2254-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e2966-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e29d4-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e2c4a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e3276-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e57f6-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e58a0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e5936-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e5aa8-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e5b5c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e5c74-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e5e40-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e5f58-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e600c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e6296-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e65ca-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e6624-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e678c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e67e6-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e689a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e694e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e6ac0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e6c32-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e6ce6-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e6d4a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e6dfe-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e6e58-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e7100-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e73da-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e7470-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e7506-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e7628-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e77e0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e786c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e798e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e7aa6-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e7c86-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e7eb6-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e7f10-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e7f74-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e819a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e83c0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e8532-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e858c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e869a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e86fe-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e8758-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e880c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e89d8-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e8a3c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e8af0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e8ba4-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e8c62-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e8cbc-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e8e24-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e8e88-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e8f46-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e8fa0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e936a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e94e6-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e959a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e95fe-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e9658-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e9770-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e982e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e9888-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e9946-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e99a0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e9a04-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e9ac2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e9bc6-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e9c20-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6e9c7a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6ea008-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6ea1b6-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6ea648-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6eaad0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6eaf4e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6eafd0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6ebc14-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6ec380-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6ec3da-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6ec4c0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6ec696-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6ec736-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6ec8b2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6ec916-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6eca92-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6ecaec-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6ecb46-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6ecc04-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6eccb8-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6ecd1c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6ecd80-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6ece34-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6ece98-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6ecef2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6ed06e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6ed122-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6ed186-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6ed23a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6ed2f8-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6ed352-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6ed46a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6ed578-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6ed5dc-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6ed636-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6ed6f4-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6ed74e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6ed7a8-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6ed866-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6ed97e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6eda96-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6edaf0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6edf28-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6edfb4-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6ee126-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6ee2e8-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6ee374-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6ee52c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6ee64e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6ee9e6-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6eeaa4-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6eeb58-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6eee78-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6eef36-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6eef9a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6eeffe-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6ef058-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6ef0bc-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6ef22e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6ef292-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6ef634-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6ef74c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6ef7a6-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6ef80a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6efb48-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6efbac-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6f008e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6f00f2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6f014c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6f02c8-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6f037c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6f03e0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6f049e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6f04f8-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6f055c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6f06ce-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6f0728-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6f0908-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6f0962-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6f0a70-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6f0aca-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6f0ea8-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6f1006-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6f10c4-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6f1128-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6f1498-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6f1902-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6f19fc-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6f21a4-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6f249c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6f2f00-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6f318a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6f3310-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6f33ce-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6f343c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6f34a0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6f34fa-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6f3676-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6f3734-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6f3798-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6f38b0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6f39c8-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6f3a22-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6f3b9e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6f3cb6-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6f3d10-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6f3d74-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6f3dce-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6f3e32-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6f3ee6-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6f3f4a-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6f3ffe-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6f4062-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6f40bc-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6f4116-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6f4346-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6f43aa-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6f4404-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6f4468-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6f45da-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6f463e-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6f4698-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6f4756-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6f4a6c-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6f4ad0-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6f4c88-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6f4fb2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>ce6f5296-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
</table>
<span style="font-style:italic;text-align:center;">5586 rows, truncated to displaylimit of 1000</span>



When you run the query by joining on the dog_guid only, you still get 389 rows in your output.  When you run the query by joining on the user_guid only, you get 5586 rows in your output.  This means that:

+ All of the user_guids in the reviews table are in the dogs table
+ Only 389 of the over 5000 dog_guids in the reviews table are in the dogs table

Perhaps most importantly for our current purposes, these COUNT queries show you that <mark>*inner joins only output the data from rows that have equivalent values in both tables being joined*.</mark>  If you wanted to include all the dog_guids or user_guids in one or both of the tables, you would have to use an outer join, which we will practice in the next lesson.

Try an inner join on your own.  
   

**Question 5: How would you extract the user_guid, dog_guid, breed, breed_type, and breed_group for all animals who completed the "Yawn Warm-up" game (you should get 20,845 rows if you join on dog_guid only)?**


```python
%%sql
SELECT d.user_guid, d.dog_guid, d.breed, d.breed_type, d.breed_group
FROM dogs d, complete_tests c
WHERE c.test_name = "Yawn Warm-up" AND d.dog_guid = c.dog_guid;
```

    20845 rows affected.





<table>
    <tr>
        <th>user_guid</th>
        <th>dog_guid</th>
        <th>breed</th>
        <th>breed_type</th>
        <th>breed_group</th>
    </tr>
    <tr>
        <td>ce134e42-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce1353d8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Shetland Sheepdog</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce135ab8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce13507c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce135e14-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Shih Tzu</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce13615c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Siberian Husky</td>
        <td>Pure Breed</td>
        <td>Working</td>
    </tr>
    <tr>
        <td>ce135e14-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Shih Tzu</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce135f2c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce136a1c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd27c1c2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce136ac6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd27c5be-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Shih Tzu-Poodle Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce136c24-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd27c74e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>German Shepherd Dog-Pembroke Welsh Corgi Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce136e36-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd27c7d0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Vizsla</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce136ee0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd27c852-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Pug</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce136f94-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd27c8d4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Boxer</td>
        <td>Pure Breed</td>
        <td>Working</td>
    </tr>
    <tr>
        <td>ce134be0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd27c956-7144-11e5-ba71-058fbc01cf0b</td>
        <td>German Shepherd Dog-Nova Scotia Duck Tolling Retriever Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce1371a6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd27cb72-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Beagle</td>
        <td>Pure Breed</td>
        <td>Hound</td>
    </tr>
    <tr>
        <td>ce136f94-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd27cd98-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Beagle</td>
        <td>Pure Breed</td>
        <td>Hound</td>
    </tr>
    <tr>
        <td>ce136f94-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd27ce1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Beagle</td>
        <td>Pure Breed</td>
        <td>Hound</td>
    </tr>
    <tr>
        <td>ce1373ae-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd27cea6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce13750c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd27cf28-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Chesapeake Bay Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce1375b6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd27cfaa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Border Collie</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce1377b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd27d02c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Belgian Malinois</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce137700-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd27d0b8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Australian Shepherd-German Shepherd Dog Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce137868-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd27d144-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Poodle</td>
        <td>Pure Breed</td>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>ce137868-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd27d1c6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Poodle</td>
        <td>Pure Breed</td>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>ce137912-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd27d248-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Doodle</td>
        <td>Popular Hybrid</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce1379c6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd27d2ca-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce137a7a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd27d34c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>German Shepherd Dog</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce137a7a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd27d3d8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>German Shepherd Dog</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce137034-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd27d45a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Weimaraner</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce137034-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd27d4dc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce137c78-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd27d770-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Bouvier des Flandres</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce135bd0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd27d9fa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce13807e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd27db08-7144-11e5-ba71-058fbc01cf0b</td>
        <td>German Shepherd Dog</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce1381c8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd27db8a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Beagle</td>
        <td>Pure Breed</td>
        <td>Hound</td>
    </tr>
    <tr>
        <td>ce138268-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd27dc52-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mudi</td>
        <td>Pure Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce138312-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd27dd38-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Parson Russell Terrier-Beagle Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce135194-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd27e026-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Dalmatian</td>
        <td>Pure Breed</td>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>ce135194-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd27e0d0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>I Don&#x27;t Know</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce13851a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd27e1e8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Border Collie-Labrador Retriever Mix</td>
        <td>Cross Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce13851a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd27e31e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Belgian Tervuren</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce1385c4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd27e454-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce1385c4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd27e580-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce135e14-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd27e9a4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Shih Tzu</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce138722-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd27eae4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Australian Terrier</td>
        <td>Pure Breed</td>
        <td>Terrier</td>
    </tr>
    <tr>
        <td>ce135e14-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd27ed46-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Shih Tzu</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce1389d4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd27efb2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce1387cc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd27f110-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Bernese Mountain Dog</td>
        <td>Pure Breed</td>
        <td>Working</td>
    </tr>
    <tr>
        <td>ce138a88-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd27f25a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Chihuahua- Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce138f92-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd27f4c6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Shetland Sheepdog</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce138f92-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd27f732-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Shetland Sheepdog</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce1390f0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd27f868-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce13919a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd27f9a8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Chihuahua-Dachshund Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce137458-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd28010a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Finnish Spitz</td>
        <td>Pure Breed</td>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>ce1394f6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd280236-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Siberian Husky</td>
        <td>Pure Breed</td>
        <td>Working</td>
    </tr>
    <tr>
        <td>ce13964a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd280344-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Rottweiler</td>
        <td>Pure Breed</td>
        <td>Working</td>
    </tr>
    <tr>
        <td>ce13985c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd280826-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Pembroke Welsh Corgi</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce137656-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd2808b2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Brussels Griffon</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce139a6e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd28093e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce139bea-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd2809c0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>French Bulldog</td>
        <td>Pure Breed</td>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>ce139bea-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3ccd24-7144-11e5-ba71-058fbc01cf0b</td>
        <td>French Bulldog</td>
        <td>Pure Breed</td>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>ce135766-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3ccf2c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Shih Tzu</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce139cb2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3cd40e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Doberman Pinscher</td>
        <td>Pure Breed</td>
        <td>Working</td>
    </tr>
    <tr>
        <td>ce139e1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3cd4d6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>German Shepherd Dog</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce13a108-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3cd8d2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Pembroke Welsh Corgi</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce13a5cc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3cd99a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>English Cocker Spaniel-Cocker Spaniel Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce13a734-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3cec50-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Rottweiler</td>
        <td>Pure Breed</td>
        <td>Working</td>
    </tr>
    <tr>
        <td>ce13a734-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3cf5c4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>German Shepherd Dog</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce13a7e8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3cf678-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Cavalier King Charles Spaniel-Bichon Frise Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce135e14-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3cf718-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Shih Tzu</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce13b152-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3cf8ee-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Bedlington Terrier</td>
        <td>Pure Breed</td>
        <td>Terrier</td>
    </tr>
    <tr>
        <td>ce21d7d2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3cf984-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce21d7d2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3cfa1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Russell Terrier</td>
        <td>Pure Breed</td>
        <td>Terrier</td>
    </tr>
    <tr>
        <td>ce137fca-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3cfab0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Poodle</td>
        <td>Pure Breed</td>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>ce21df2a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3cfcfe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Irish Setter</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce21e11e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3cfd94-7144-11e5-ba71-058fbc01cf0b</td>
        <td>German Shepherd Dog</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce21df2a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3cfe2a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Irish Setter</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce21e11e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3cfeb6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Poodle</td>
        <td>Pure Breed</td>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>ce21df2a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3cff4c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Irish Red and White Setter</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce21e736-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d0078-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Poodle-Cocker Spaniel Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce21e826-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d01ae-7144-11e5-ba71-058fbc01cf0b</td>
        <td>American Pit Bull Terrier</td>
        <td>Pure Breed</td>
        <td>Terrier</td>
    </tr>
    <tr>
        <td>ce21f122-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d03fc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce21f528-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d0492-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Beagle-Schipperke Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce22007c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d05be-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Greyhound</td>
        <td>Pure Breed</td>
        <td>Hound</td>
    </tr>
    <tr>
        <td>ce22011c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d064a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever-Golden Retriever Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce22011c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d06e0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever-Golden Retriever Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce2202e8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d0776-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce2202e8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d080c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Boston Terrier-Chihuahua Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce2203f6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d0898-7144-11e5-ba71-058fbc01cf0b</td>
        <td>American Pit Bull Terrier</td>
        <td>Pure Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce2204a0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d0938-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce2204a0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d09ce-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Beagle-Cavalier King Charles Spaniel Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce220734-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d0b7c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Boxer</td>
        <td>Pure Breed</td>
        <td>Working</td>
    </tr>
    <tr>
        <td>ce2205ea-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d0c12-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Pug</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce2205ea-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d0cb2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>French Bulldog</td>
        <td>Pure Breed</td>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>ce2207de-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d0d48-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce220892-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d0dde-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce220a72-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d0f00-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labradoodle</td>
        <td>Popular Hybrid</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce220b12-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d0f96-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce220bb2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d102c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Pembroke Welsh Corgi</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce220bb2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d10cc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce220cf2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d1162-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Cocker Spaniel</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce2209d2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d1202-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce2209d2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d12fc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Rottweiler</td>
        <td>Pure Breed</td>
        <td>Working</td>
    </tr>
    <tr>
        <td>ce220e3c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d150e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever-Border Collie Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce220ee6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d15f4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Lhasa Apso-Poodle Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce137e80-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d17c0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labradoodle</td>
        <td>Popular Hybrid</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce221210-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d18a6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>English Springer Spaniel</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce221210-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d1982-7144-11e5-ba71-058fbc01cf0b</td>
        <td>English Springer Spaniel</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce22135a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d1a5e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce134a78-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d1b44-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Shih Tzu</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce2213fa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d1d06-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Neapolitan Mastiff</td>
        <td>Pure Breed</td>
        <td>Working</td>
    </tr>
    <tr>
        <td>ce221774-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d2080-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Rat Terrier</td>
        <td>Pure Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce221a76-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d2116-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Border Terrier</td>
        <td>Pure Breed</td>
        <td>Terrier</td>
    </tr>
    <tr>
        <td>ce2218e6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d224c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Collie-Shetland Sheepdog Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce221b3e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d22d8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>German Shepherd Dog</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce221dbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d2530-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Dachshund</td>
        <td>Pure Breed</td>
        <td>Hound</td>
    </tr>
    <tr>
        <td>ce221dbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d25c6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Dachshund</td>
        <td>Pure Breed</td>
        <td>Hound</td>
    </tr>
    <tr>
        <td>ce221e5e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d265c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever-Collie Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce221efe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d26f2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Beagle</td>
        <td>Pure Breed</td>
        <td>Hound</td>
    </tr>
    <tr>
        <td>ce221f9e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d2788-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce22203e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d281e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>American Eskimo Dog-Papillon Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce22203e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d28aa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Papillon</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce2220de-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d2940-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Pomeranian</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce222214-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d29d6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>German Shepherd Dog-Belgian Tervuren Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce2222aa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d2a6c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>German Shepherd Dog</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce22234a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d2af8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Maltese-Yorkshire Terrier Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce22248a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d2c24-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Australian Shepherd</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce22252a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d2d46-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Shiba Inu</td>
        <td>Pure Breed</td>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>ce2225d4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d2ddc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Rat Terrier</td>
        <td>Pure Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce222674-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d2e72-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Poodle</td>
        <td>Pure Breed</td>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>ce2227be-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d2f08-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Border Collie-Greyhound Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce2228fe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d2f9e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Poodle</td>
        <td>Pure Breed</td>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>ce222a02-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d30c0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labradoodle</td>
        <td>Popular Hybrid</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce137f20-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d3156-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce221c88-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d3278-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Maltese-Poodle Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce222a70-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d330e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Siberian Husky-German Shepherd Dog Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce222ade-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d33a4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Chihuahua</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce221d1e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d343a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever-German Shepherd Dog Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce222fa2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d34d0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Parson Russell Terrier</td>
        <td>Pure Breed</td>
        <td>Terrier</td>
    </tr>
    <tr>
        <td>ce2232cc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d35f2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Rhodesian Ridgeback</td>
        <td>Pure Breed</td>
        <td>Hound</td>
    </tr>
    <tr>
        <td>ce223452-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d3688-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce2234f2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d371e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce2234f2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d37b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Chihuahua</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce223628-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d3840-7144-11e5-ba71-058fbc01cf0b</td>
        <td>West Highland White Terrier</td>
        <td>Pure Breed</td>
        <td>Terrier</td>
    </tr>
    <tr>
        <td>ce2236c8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d38d6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Poodle-Miniature Schnauzer Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce2236c8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d396c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Maltese-Poodle Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce2236c8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d39f8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Yorkshire Terrier-Poodle Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce22375e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d3a8e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Vizsla</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce2237f4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d3b24-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce2239c0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d3bb0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labradoodle</td>
        <td>Popular Hybrid</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce223af6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d3c46-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Silky Terrier-Poodle Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce223c22-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d3ffc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce222214-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d4092-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Russell Terrier-Miniature Pinscher Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce223cb8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d4128-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce223de4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d424a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Bernese Mountain Dog</td>
        <td>Pure Breed</td>
        <td>Working</td>
    </tr>
    <tr>
        <td>ce223de4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d42ea-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Bernese Mountain Dog</td>
        <td>Pure Breed</td>
        <td>Working</td>
    </tr>
    <tr>
        <td>ce223de4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d4376-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Bernese Mountain Dog</td>
        <td>Pure Breed</td>
        <td>Working</td>
    </tr>
    <tr>
        <td>ce223f06-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d440c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Yorkshire Terrier</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce22408c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d44a2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Australian Cattle Dog- Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce22392a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d452e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce2240f0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d45ce-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Flat-Coated Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce224028-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d4664-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Staffordshire Bull Terrier-Bulldog Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce22414a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d46fa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce2241ae-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d479a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce224208-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d4830-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Kooikerhondje</td>
        <td>Pure Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce224208-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d48c6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>German Shepherd Dog</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce22426c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d4952-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever-Golden Retriever Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce22432a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d4a6a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>German Shepherd Dog-Border Collie Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce22438e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d4b00-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce2243e8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d4b8c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce22444c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d4f74-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce22456e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d515e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>German Shepherd Dog</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce22456e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d51f4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce22492e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d5320-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labradoodle</td>
        <td>Popular Hybrid</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce135e14-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d587a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Shih Tzu</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce222e58-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d5a1e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>American Staffordshire Terrier</td>
        <td>Pure Breed</td>
        <td>Terrier</td>
    </tr>
    <tr>
        <td>ce222e58-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3d5aa0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Bulldog</td>
        <td>Pure Breed</td>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>ce224b0e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3f81ea-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce138920-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3f83d4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce224b68-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3f8514-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Beagle</td>
        <td>Pure Breed</td>
        <td>Hound</td>
    </tr>
    <tr>
        <td>ce224b68-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3f8aa0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Beagle</td>
        <td>Pure Breed</td>
        <td>Hound</td>
    </tr>
    <tr>
        <td>ce224bcc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3f8bcc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Cardigan Welsh Corgi-German Shepherd Dog Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce224a46-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3f90fe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>German Shepherd Dog</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce224d52-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3f9c2a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Doberman Pinscher</td>
        <td>Pure Breed</td>
        <td>Working</td>
    </tr>
    <tr>
        <td>ce224d52-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3fa0b2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Doberman Pinscher-German Shepherd Dog Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce224e10-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3fb00c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce1353d8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3fb0f2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Shetland Sheepdog</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce224dac-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3fb5e8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Canaan Dog- Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce22511c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3fb76e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Beagle</td>
        <td>Pure Breed</td>
        <td>Hound</td>
    </tr>
    <tr>
        <td>ce2251da-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3fb8fe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Rhodesian Ridgeback</td>
        <td>Pure Breed</td>
        <td>Hound</td>
    </tr>
    <tr>
        <td>ce22523e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3fb9c6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Miniature Schnauzer</td>
        <td>Pure Breed</td>
        <td>Terrier</td>
    </tr>
    <tr>
        <td>ce225360-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3fbc00-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Russell Terrier-Pug Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce138876-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3fbd7c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce2255f4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3fbe44-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Australian Cattle Dog-Border Collie Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce225658-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3fbf0c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce135e14-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3fbfe8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Shih Tzu</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce224866-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3fc0ce-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Shih Tzu</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce22590a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3fc1be-7144-11e5-ba71-058fbc01cf0b</td>
        <td>English Cocker Spaniel</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce225c16-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3fc628-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Australian Shepherd</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce225c16-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3fc704-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Parson Russell Terrier</td>
        <td>Pure Breed</td>
        <td>Terrier</td>
    </tr>
    <tr>
        <td>ce224cee-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3fc7ea-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Boxer</td>
        <td>Pure Breed</td>
        <td>Working</td>
    </tr>
    <tr>
        <td>ce225d92-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3fc9fc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>German Shepherd Dog</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce2249ec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3fcc40-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Chihuahua-Dachshund Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce225df6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3fcd58-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Australian Cattle Dog</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce225f18-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3fd014-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Border Collie-Australian Shepherd Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce135766-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3fd140-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Shih Tzu</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce225f72-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3fd1fe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Miniature Schnauzer</td>
        <td>Pure Breed</td>
        <td>Terrier</td>
    </tr>
    <tr>
        <td>ce225fd6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3fd2b2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Australian Shepherd-Border Collie Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce22608a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3fd348-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce2260e4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3fd3de-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Poodle</td>
        <td>Pure Breed</td>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>ce22613e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3fd474-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce2261a2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3fd514-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce2266de-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3fd5aa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>French Spaniel</td>
        <td>Pure Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce226c1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3fd74e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce226bb6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3fd988-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Italian Greyhound</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce23e626-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3fda6e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Weimaraner</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce23e4a0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3fdb54-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Leonberger</td>
        <td>Pure Breed</td>
        <td>Working</td>
    </tr>
    <tr>
        <td>ce23e6ee-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3fdc3a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Poodle</td>
        <td>Pure Breed</td>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>ce2267a6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3fdd20-7144-11e5-ba71-058fbc01cf0b</td>
        <td>German Shepherd Dog</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce136210-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3fde06-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce23eb9e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3fdeec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Portuguese Water Dog</td>
        <td>Pure Breed</td>
        <td>Working</td>
    </tr>
    <tr>
        <td>ce23ec8e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3fdfc8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Boykin Spaniel</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce23f1de-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3fe0a4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce23f2a6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3fe18a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce23f634-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3fe270-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Soft Coated Wheaten Terrier</td>
        <td>Pure Breed</td>
        <td>Terrier</td>
    </tr>
    <tr>
        <td>ce23f9cc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3fe432-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Poodle</td>
        <td>Pure Breed</td>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>ce24071e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3fe50e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce2409ee-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3fe7b6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce240a8e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3fe842-7144-11e5-ba71-058fbc01cf0b</td>
        <td>German Shepherd Dog</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce240bba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3fe8e2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Bulldog</td>
        <td>Pure Breed</td>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>ce240b24-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3fe96e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce240d72-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3fea04-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Boston Terrier</td>
        <td>Pure Breed</td>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>ce663ce2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3fea9a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Shih Tzu</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce240f20-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3febc6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Brittany</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce240fac-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3fec5c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Bernese Mountain Dog</td>
        <td>Pure Breed</td>
        <td>Working</td>
    </tr>
    <tr>
        <td>ce241042-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3fed7e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever-Labrador Retriever Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce2410d8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3fee14-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce241042-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3feeb4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever-Labrador Retriever Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce241178-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3fef40-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce241218-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3fefd6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Australian Cattle Dog</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce2412ae-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3ff06c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever-Chinese Shar-Pei Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce2412ae-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3ff102-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce241344-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3ff18e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Nova Scotia Duck Tolling Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce241344-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3ff22e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Nova Scotia Duck Tolling Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce2413e4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3ff47c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce241524-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3ff512-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce241524-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3ff5a8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce2415ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3ff63e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Australian Shepherd</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce2413e4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3ff882-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce24165a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3ff90e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Siberian Husky</td>
        <td>Pure Breed</td>
        <td>Working</td>
    </tr>
    <tr>
        <td>ce24181c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3ffb5c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce2419de-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3ffbf2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce241a74-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3ffc7e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Lhasa Apso</td>
        <td>Pure Breed</td>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>ce241b00-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3ffda0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Australian Shepherd</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce241ba0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3ffe36-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce135cf2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd3ffecc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce241ccc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd400084-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce241d62-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4003b8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce241de4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd400458-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce2422e4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd400584-7144-11e5-ba71-058fbc01cf0b</td>
        <td>American Eskimo Dog</td>
        <td>Pure Breed</td>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>ce2423fc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd400610-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Boxer</td>
        <td>Pure Breed</td>
        <td>Working</td>
    </tr>
    <tr>
        <td>ce242370-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4007c8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Keeshond</td>
        <td>Pure Breed</td>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>ce2427f8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4008f4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Bull Terrier</td>
        <td>Pure Breed</td>
        <td>Terrier</td>
    </tr>
    <tr>
        <td>ce242762-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd40098a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Soft Coated Wheaten Terrier</td>
        <td>Pure Breed</td>
        <td>Terrier</td>
    </tr>
    <tr>
        <td>ce24291a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd400a16-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Boston Terrier</td>
        <td>Pure Breed</td>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>ce242884-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd400aac-7144-11e5-ba71-058fbc01cf0b</td>
        <td>German Shepherd Dog</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce242aa0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd400b4c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Poodle</td>
        <td>Pure Breed</td>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>ce242b2c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd400bec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>German Shepherd Dog</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce242cc6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd400c78-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Shih Tzu-Maltese Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce241786-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd400d04-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce243202-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd400fca-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Chesapeake Bay Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce2432a2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4014a2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td></td>
    </tr>
    <tr>
        <td>ce47264a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd401614-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Shih Tzu</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce2434e6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4016aa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Cocker Spaniel</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce2421cc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd401858-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce2437de-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd401d6c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Maltese</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce242640-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd401efc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce243d1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd40241a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Cockapoo</td>
        <td>Popular Hybrid</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce243fa4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd402a5a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Chihuahua-Miniature Pinscher Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce24422e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd402b90-7144-11e5-ba71-058fbc01cf0b</td>
        <td>German Shepherd Dog</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce24430a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd403036-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce24465c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4038c4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Shih Tzu</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce224866-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd404634-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Shih Tzu</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce244800-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd404760-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Australian Cattle Dog</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce244918-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd404832-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Maltese-Poodle Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce244918-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4048c8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Maltese-Shih Tzu Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce2449a4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd404954-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce244c7e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd404b02-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Boxer</td>
        <td>Pure Breed</td>
        <td>Working</td>
    </tr>
    <tr>
        <td>ce244be8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd404b8e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Dachshund-Miniature Pinscher Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce223894-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd404d3c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce244da0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd404dc8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Miniature American Shepherd</td>
        <td>Pure Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce244f44-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd404e54-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce24505c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd404eea-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce24476a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd404ff8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce2420aa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd40511a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Parson Russell Terrier</td>
        <td>Pure Breed</td>
        <td>Terrier</td>
    </tr>
    <tr>
        <td>ce2453a4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4051a6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Border Collie</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce2454c6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd405232-7144-11e5-ba71-058fbc01cf0b</td>
        <td>German Shepherd Dog</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce2455e8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4052b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever-Australian Cattle Dog Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce2444ae-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd405462-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Poodle</td>
        <td>Pure Breed</td>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>ce135766-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4054ee-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Shih Tzu</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce134d16-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd40557a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Shih Tzu</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce244b52-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd405610-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce245926-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd405732-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce2459b2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4057be-7144-11e5-ba71-058fbc01cf0b</td>
        <td>I Don&#x27;t Know</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce245bc4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4058d6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Border Collie</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce245cdc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4059e4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce245e08-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd405a70-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce245e6c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd405b88-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce242136-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd405c14-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Chihuahua</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce244eb8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd405e30-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Australian Shepherd</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce245ffc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd405f52-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Poodle</td>
        <td>Pure Breed</td>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>ce138876-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd405fde-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Affenpinscher</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce2460ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd40606a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Keeshond</td>
        <td>Pure Breed</td>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>ce246182-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4060f6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Chihuahua</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce2461dc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd406196-7144-11e5-ba71-058fbc01cf0b</td>
        <td>English Setter</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce246236-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd406466-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Miniature Schnauzer-Poodle Mix</td>
        <td>Cross Breed</td>
        <td></td>
    </tr>
    <tr>
        <td>ce2462fe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd406628-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce23ed60-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd406704-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Havanese</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce246358-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4067d6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>German Shepherd Dog</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce246650-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd406a4c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Australian Shepherd</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce2466aa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd406b1e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce244080-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd406bf0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Border Collie</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce240cdc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd406cc2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Boxer</td>
        <td>Pure Breed</td>
        <td>Working</td>
    </tr>
    <tr>
        <td>ce24670e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd406d94-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever-Cane Corso Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce2467cc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd406eb6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce246830-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd406f42-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Miniature Schnauzer</td>
        <td>Pure Breed</td>
        <td>Terrier</td>
    </tr>
    <tr>
        <td>ce24201e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd406fce-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce2469a2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd407050-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce246a06-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4070dc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Bichon Frise-Shih Tzu Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce221e5e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd407168-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Pug</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce246a60-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4071f4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Pointer-Labrador Retriever Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce246a60-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd407280-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever-Basenji Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce246ac4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd407302-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Poodle</td>
        <td>Pure Breed</td>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>ce246be6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd40738e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce2468e4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4074a6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Australian Cattle Dog</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce246ca4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd407528-7144-11e5-ba71-058fbc01cf0b</td>
        <td>German Shepherd Dog</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce246cfe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4075b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce246d62-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd407636-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Doodle</td>
        <td>Popular Hybrid</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce245b2e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4077d0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Pembroke Welsh Corgi-Russell Terrier Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce246e20-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd407866-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce246948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4078e8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td></td>
    </tr>
    <tr>
        <td>ce246ef2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd407a00-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce246f4c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd407a8c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Border Collie-Labrador Retriever Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce246f9c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd407b18-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce24749c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd407e56-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Bulldog-Beagle Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce2471a4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd407f6e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Border Collie</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce2474f6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd407ff0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Bulldog</td>
        <td>Pure Breed</td>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>ce247726-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd40807c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce242a0a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd408194-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Nova Scotia Duck Tolling Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce247668-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd408220-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce247c9e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd408446-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Australian Shepherd</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce248130-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd408662-7144-11e5-ba71-058fbc01cf0b</td>
        <td>English Cocker Spaniel</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce249774-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd408ed2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce249922-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd409224-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Poodle-Maltese Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce249bac-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd409346-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Great Dane-Labrador Retriever Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce243a90-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4093dc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce249c6a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd40945e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce249d1e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4094ea-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Boxer</td>
        <td>Pure Breed</td>
        <td>Working</td>
    </tr>
    <tr>
        <td>ce249eea-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4096fc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Bernese Mountain Dog</td>
        <td>Pure Breed</td>
        <td>Working</td>
    </tr>
    <tr>
        <td>ce24a278-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd409e36-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Havanese</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce24a32c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd40a3c2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce24a4ee-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd40aeee-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Boxer</td>
        <td>Pure Breed</td>
        <td>Working</td>
    </tr>
    <tr>
        <td>ce24a5a2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd40b06a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce221e5e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd40b970-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Brittany-Poodle Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce2258a6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd40bd62-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Shih Tzu</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce249c06-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd40cae6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce24a822-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd40cb7c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Scottish Terrier</td>
        <td>Pure Breed</td>
        <td>Terrier</td>
    </tr>
    <tr>
        <td>ce24a8e0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd40cc94-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce244e2c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd40cdac-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce24aaa2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd40ce38-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Weimaraner</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce24ab56-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd40cf5a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Shih Tzu</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce225b4e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd40d4dc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Weimaraner</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce24ae8a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd40d568-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce24ae30-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd40d5f4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>West Highland White Terrier</td>
        <td>Pure Breed</td>
        <td>Terrier</td>
    </tr>
    <tr>
        <td>ce24b0ec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd40d70c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labradoodle</td>
        <td>Popular Hybrid</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce24b362-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd40d8b0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce24b47a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd40d93c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce24b5d8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd40da54-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Beagle</td>
        <td>Pure Breed</td>
        <td>Hound</td>
    </tr>
    <tr>
        <td>ce24b786-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd40db76-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever-Cardigan Welsh Corgi Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce24ab56-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd40dc02-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce24ba4c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd40debe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce24bad8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd40dfd6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce24bbfa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd40e058-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Bearded Collie-Tibetan Terrier Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce134492-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd40e206-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Shih Tzu</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce245f3e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd40e332-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Australian Shepherd</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce24bdd0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd40e440-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Beagle</td>
        <td>Pure Breed</td>
        <td>Hound</td>
    </tr>
    <tr>
        <td>ce24bdd0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd40e4cc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Beagle</td>
        <td>Pure Breed</td>
        <td>Hound</td>
    </tr>
    <tr>
        <td>ce24bee8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd40e832-7144-11e5-ba71-058fbc01cf0b</td>
        <td>German Shepherd Dog</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce24abb0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd40eb0c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>I Don&#x27;t Know</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce24c050-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd40ebe8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce24c0b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd40ecc4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Miniature American Shepherd</td>
        <td>Pure Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce24c10e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd40eda0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Great Dane</td>
        <td>Pure Breed</td>
        <td>Working</td>
    </tr>
    <tr>
        <td>ce24c1c2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd40ee72-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce24c226-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd40ef4e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever-Labrador Retriever Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce242e74-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd40f02a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce24c50a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd40f4e4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce24c5be-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd40f714-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Icelandic Sheepdog</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce24a106-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd40f822-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce137bce-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd40fa66-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Dachshund</td>
        <td>Pure Breed</td>
        <td>Hound</td>
    </tr>
    <tr>
        <td>ce24c7ee-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd40fcaa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>German Shorthaired Pointer-Labrador Retriever Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce24c8fc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd40fdc2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Miniature Schnauzer</td>
        <td>Pure Breed</td>
        <td>Terrier</td>
    </tr>
    <tr>
        <td>ce24ca6e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd40feda-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Miniature Schnauzer</td>
        <td>Pure Breed</td>
        <td>Terrier</td>
    </tr>
    <tr>
        <td>ce24cbd6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4100ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Pug-Chihuahua Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce24cac8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd410146-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Cockapoo</td>
        <td>Popular Hybrid</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce24cce4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41025e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce24cdf2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd410376-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Weimaraner</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce24cea6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd410402-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Yorkshire Terrier-Bichon Frise Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce24d0cc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41059c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Cockapoo</td>
        <td>Popular Hybrid</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce24d130-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd410632-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Border Collie</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce24d572-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4107cc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce24d0cc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd410858-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Cairn Terrier</td>
        <td>Pure Breed</td>
        <td>Terrier</td>
    </tr>
    <tr>
        <td>ce24d5cc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4108e4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Yorkshire Terrier- Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce24d5cc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd410966-7144-11e5-ba71-058fbc01cf0b</td>
        <td>German Shepherd Dog</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce24d6e4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4109f2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Belgian Sheepdog- Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce24d73e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd410a7e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Australian Cattle Dog-Border Collie Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce24c280-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd410b14-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Old English Sheepdog</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce24d8ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd410c2c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>West Highland White Terrier</td>
        <td>Pure Breed</td>
        <td>Terrier</td>
    </tr>
    <tr>
        <td>ce24d914-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd410cb8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce24da2c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd410e5c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce24da86-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4111f4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce24dae0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd411280-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Miniature Schnauzer</td>
        <td>Pure Breed</td>
        <td>Terrier</td>
    </tr>
    <tr>
        <td>ce24db3a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4112ee-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce24d248-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd411366-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Boston Terrier</td>
        <td>Pure Breed</td>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>ce24db9e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4113d4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Portuguese Water Dog</td>
        <td>Pure Breed</td>
        <td>Working</td>
    </tr>
    <tr>
        <td>ce24dc5c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41144c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>American Pit Bull Terrier-American Staffordshire Terrier Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce24dc5c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4114c4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>American Pit Bull Terrier-American Staffordshire Terrier Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce24ddce-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4115aa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce24de50-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd411690-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce24e116-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4118e8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce24e0bc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd411956-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Danish-Swedish Farmdog</td>
        <td>Pure Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce24e684-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd411b0e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce24a7c8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd411b86-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce24eac6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd411bea-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce24eb52-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd411f00-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Cardigan Welsh Corgi</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce24ea6c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd412022-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Bernese Mountain Dog</td>
        <td>Pure Breed</td>
        <td>Working</td>
    </tr>
    <tr>
        <td>ce24e6e8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd412112-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Boxer</td>
        <td>Pure Breed</td>
        <td>Working</td>
    </tr>
    <tr>
        <td>ce24ec56-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41218a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Boxer</td>
        <td>Pure Breed</td>
        <td>Working</td>
    </tr>
    <tr>
        <td>ce24f50c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd412aae-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Poodle</td>
        <td>Pure Breed</td>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>ce24f50c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd412b62-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce24fb2e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd413080-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labradoodle</td>
        <td>Popular Hybrid</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce24ff3e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd413422-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce24ff3e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4134d6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labradoodle</td>
        <td>Popular Hybrid</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce250646-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd413bde-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce2507fe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd414674-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Shetland Sheepdog</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce2507fe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4146f6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Shetland Sheepdog</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce24efd0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd414764-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Cavalier King Charles Spaniel</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce250858-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4147e6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Beagle</td>
        <td>Pure Breed</td>
        <td>Hound</td>
    </tr>
    <tr>
        <td>ce2508bc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4148cc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Bichon Frise-Cavalier King Charles Spaniel Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce25097a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd414944-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Doodle</td>
        <td>Popular Hybrid</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce250a38-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd414b1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce250af6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd414b92-7144-11e5-ba71-058fbc01cf0b</td>
        <td>French Bulldog</td>
        <td>Pure Breed</td>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>ce250c0e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd414c78-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Cockapoo</td>
        <td>Popular Hybrid</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce250d26-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd414cf0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Rhodesian Ridgeback-Boxer Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce250d26-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd414d68-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce250d26-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd414de0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Poodle-Cavalier King Charles Spaniel Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce250d26-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd414e4e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Poodle-Cavalier King Charles Spaniel Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce250c72-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd414ec6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce250dee-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd414fac-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Beagle-Australian Cattle Dog Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce24b2cc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd415024-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce250ea2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41509c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce250fb0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4151f0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Shetland Sheepdog</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce225d38-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41525e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever-Rottweiler Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce25106e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4152d6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Pomeranian</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce251014-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41534e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Poodle</td>
        <td>Pure Breed</td>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>ce25117c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4154a2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce138be6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41551a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Yorkshire Terrier</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce2511d6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd415588-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Shih Tzu</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce250e48-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd415600-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce2512ee-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd415754-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Border Collie</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce251348-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4157c2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce251406-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41583a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Dalmatian</td>
        <td>Pure Breed</td>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>ce2514c4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd415920-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Chow Chow-Golden Retriever Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce3f044c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd415a7e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>German Shepherd Dog-Beagle Mix</td>
        <td>Cross Breed</td>
        <td></td>
    </tr>
    <tr>
        <td>ce2515dc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd415af6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Siberian Husky</td>
        <td>Pure Breed</td>
        <td>Working</td>
    </tr>
    <tr>
        <td>ce251744-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd415c4a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Other</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce2517a8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd415cc2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>American Staffordshire Terrier-Labrador Retriever Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce2518b6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd415ea2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Yorkshire Terrier</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce251910-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd415f92-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Cockapoo</td>
        <td>Popular Hybrid</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce251974-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd416050-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Affenpinscher</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce2519ce-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd416104-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Maltese</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce251a32-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4161c2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce251af0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd416276-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Border Terrier- Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce251b4a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd416320-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Scottish Terrier</td>
        <td>Pure Breed</td>
        <td>Terrier</td>
    </tr>
    <tr>
        <td>ce251a8c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4163d4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>English Setter</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce251cbc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4166a4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce251d20-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41674e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce251d20-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4168b6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce252144-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd416c62-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Other</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce252298-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd416d48-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Havanese</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce252298-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd416dc0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Havanese</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce252298-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd416e38-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Havanese</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce252342-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd416eb0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Border Terrier</td>
        <td>Pure Breed</td>
        <td>Terrier</td>
    </tr>
    <tr>
        <td>ce252342-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd416f1e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Border Terrier</td>
        <td>Pure Breed</td>
        <td>Terrier</td>
    </tr>
    <tr>
        <td>ce252342-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd416f96-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Border Terrier</td>
        <td>Pure Breed</td>
        <td>Terrier</td>
    </tr>
    <tr>
        <td>ce2523ce-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd417072-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Australian Shepherd</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce2525c2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd417158-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Bulldog</td>
        <td>Pure Breed</td>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>ce25277a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd417248-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Coton de Tulear</td>
        <td>Pure Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce241f92-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41739c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Cardigan Welsh Corgi</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce252c02-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd417572-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce252dba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd417658-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce252cfc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4176c6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Russell Terrier</td>
        <td>Pure Breed</td>
        <td>Terrier</td>
    </tr>
    <tr>
        <td>ce252e78-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4177ac-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Greyhound</td>
        <td>Pure Breed</td>
        <td>Hound</td>
    </tr>
    <tr>
        <td>ce25310c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4179f0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>American Water Spaniel</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce134e42-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd417cac-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce2532d8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd417d1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>German Shepherd Dog</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce25333c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd417d92-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Russell Terrier-Chihuahua Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce2533f0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd417e78-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce253562-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd418044-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Poodle</td>
        <td>Pure Breed</td>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>ce253792-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4186d4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Cavalier King Charles Spaniel</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce253792-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd418756-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mastiff</td>
        <td>Pure Breed</td>
        <td>Working</td>
    </tr>
    <tr>
        <td>ce253792-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4187c4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Cavalier King Charles Spaniel</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce253904-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4188c8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce25395e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4189ae-7144-11e5-ba71-058fbc01cf0b</td>
        <td>I Don&#x27;t Know</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce253a12-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd418a8a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Shiba Inu</td>
        <td>Pure Breed</td>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>ce253a6c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd418b02-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce2537ec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd418bde-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce253b20-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd418ef4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce253b84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd419002-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Border Collie</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce253c38-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4190fc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Beagle</td>
        <td>Pure Breed</td>
        <td>Hound</td>
    </tr>
    <tr>
        <td>ce253cf6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4191ec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce253d50-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd419598-7144-11e5-ba71-058fbc01cf0b</td>
        <td>German Shepherd Dog</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce226030-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd419610-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Miniature Schnauzer</td>
        <td>Pure Breed</td>
        <td>Terrier</td>
    </tr>
    <tr>
        <td>ce253daa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41967e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labradoodle</td>
        <td>Popular Hybrid</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce253e68-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4196ec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Australian Shepherd-Golden Retriever Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce253e04-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41975a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>German Shorthaired Pointer</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce253e04-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd419aca-7144-11e5-ba71-058fbc01cf0b</td>
        <td>German Shorthaired Pointer</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce253e04-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd419b7e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>German Shorthaired Pointer</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce253e04-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd419c28-7144-11e5-ba71-058fbc01cf0b</td>
        <td>German Shorthaired Pointer</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce253f80-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41a1aa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Border Collie-Belgian Tervuren Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce253fda-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41a574-7144-11e5-ba71-058fbc01cf0b</td>
        <td>American Staffordshire Terrier-Australian Shepherd Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce253fda-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41a8a8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Pekingese-Dachshund Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce254098-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41a952-7144-11e5-ba71-058fbc01cf0b</td>
        <td>German Shepherd Dog-Siberian Husky Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce2540fc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41acb8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Doodle</td>
        <td>Popular Hybrid</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce25420a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41b6ea-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Rat Terrier</td>
        <td>Pure Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce2544d0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41b74e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Chihuahua</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce254534-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41b7e4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Portuguese Water Dog</td>
        <td>Pure Breed</td>
        <td>Working</td>
    </tr>
    <tr>
        <td>ce25476e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41baaa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Rottweiler- Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce25482c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41bb7c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td></td>
    </tr>
    <tr>
        <td>ce7b055a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41bca8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce254a5c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41bdde-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce254ab6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41be42-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce254bce-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41bf6e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Pug-Maltese Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce254c28-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41bfd2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Boston Terrier</td>
        <td>Pure Breed</td>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>ce2258a6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41c040-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Boston Terrier</td>
        <td>Pure Breed</td>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>ce254d4a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41c108-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labradoodle</td>
        <td>Popular Hybrid</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce254d4a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41c162-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Border Collie</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce254ed0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41c234-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Cairn Terrier</td>
        <td>Pure Breed</td>
        <td>Terrier</td>
    </tr>
    <tr>
        <td>ce254db8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41c338-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td></td>
    </tr>
    <tr>
        <td>ce135e14-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41c400-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Shih Tzu</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce255146-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41c464-7144-11e5-ba71-058fbc01cf0b</td>
        <td>English Springer Spaniel</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce255466-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41c4d2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce25563c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41c590-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Maltese</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce2556a0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41c5f4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce2554ca-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41c720-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce255c54-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41c84c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce255c54-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41c8b0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Siberian Husky-Australian Shepherd Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce255b64-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41c9f0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Australian Cattle Dog</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce25697e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41cb26-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Chihuahua</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce2570b8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41cc98-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Whippet</td>
        <td>Pure Breed</td>
        <td>Hound</td>
    </tr>
    <tr>
        <td>ce256636-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41cd2e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>West Highland White Terrier</td>
        <td>Pure Breed</td>
        <td>Terrier</td>
    </tr>
    <tr>
        <td>ce2578ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41d030-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Boston Terrier-Bulldog Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce135cf2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41d0c6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce25791e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41d166-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce2578ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41d580-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce257bb2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41d620-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce257c16-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41d684-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Portuguese Water Dog</td>
        <td>Pure Breed</td>
        <td>Working</td>
    </tr>
    <tr>
        <td>ce257c70-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41d74c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Border Collie-Dalmatian Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce257cd4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41d7b0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Other</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce257e46-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41d878-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Cocker Spaniel</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce257e46-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41d8dc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Welsh Springer Spaniel</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce257f04-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41da08-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Poodle-Old English Sheepdog Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce257f68-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41dad0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce257fc2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41db34-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce258080-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41dc60-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce24085e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41dcc4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce2580e4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41dd28-7144-11e5-ba71-058fbc01cf0b</td>
        <td>German Shepherd Dog</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce257d2e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41dd8c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce2581a2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41de54-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce2581a2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41deae-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce258256-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41df12-7144-11e5-ba71-058fbc01cf0b</td>
        <td>German Shepherd Dog</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce2582ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41df76-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labradoodle</td>
        <td>Popular Hybrid</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce25842c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41e0a2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce258490-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41e106-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce2584ea-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41e16a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Border Terrier</td>
        <td>Pure Breed</td>
        <td>Terrier</td>
    </tr>
    <tr>
        <td>ce2585a8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41e232-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce22680a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41e28c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce258666-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41e354-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Staffordshire Bull Terrier</td>
        <td>Pure Breed</td>
        <td>Terrier</td>
    </tr>
    <tr>
        <td>ce2586c0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41e3b8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Australian Shepherd</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce258774-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41e476-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Havanese</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce258896-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41e5a2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce24a9ee-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41e606-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Soft Coated Wheaten Terrier</td>
        <td>Pure Breed</td>
        <td>Terrier</td>
    </tr>
    <tr>
        <td>ce258a6c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41e6ce-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce258a08-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41e796-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce258b84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41ea0c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td></td>
    </tr>
    <tr>
        <td>ce262364-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41eb38-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td></td>
    </tr>
    <tr>
        <td>ce258d00-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41ecc8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Poodle</td>
        <td>Pure Breed</td>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>ce258d00-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41ed2c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Poodle</td>
        <td>Pure Breed</td>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>ce258dc8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41ed90-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Shih Tzu</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce258e2c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41edfe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Border Collie</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce258e2c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41ee6c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Border Collie-Staffordshire Bull Terrier Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce258f4e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41ef3e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Siberian Husky</td>
        <td>Pure Breed</td>
        <td>Working</td>
    </tr>
    <tr>
        <td>ce258f4e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41efa2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>German Shepherd Dog</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce135766-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41f056-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Shih Tzu</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce258d64-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41f0b0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce259368-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41f3bc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>English Springer Spaniel</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce259494-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41f574-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Australian Cattle Dog- Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce259534-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41f5d8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Boxer</td>
        <td>Pure Breed</td>
        <td>Working</td>
    </tr>
    <tr>
        <td>ce252c98-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41f95c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Other</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce2598a4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd41fe20-7144-11e5-ba71-058fbc01cf0b</td>
        <td>I Don&#x27;t Know</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce259944-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd420276-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Doodle</td>
        <td>Popular Hybrid</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce259a3e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4202f8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Bichon Frise-Poodle Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce259ca0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42125c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce259d36-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4212ca-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Shetland Sheepdog</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce259d36-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd421338-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Shetland Sheepdog</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce259dd6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd421400-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce24aa48-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4214be-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce25a088-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd421586-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce25a11e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4215ea-7144-11e5-ba71-058fbc01cf0b</td>
        <td>German Shorthaired Pointer-Catahoula Leopard Dog Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce25a236-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4217d4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce25a2ea-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd421838-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Alaskan Malamute-Collie Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce25a466-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd421a7c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce25a4c0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd421b9e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Doberman Pinscher</td>
        <td>Pure Breed</td>
        <td>Working</td>
    </tr>
    <tr>
        <td>ce25a696-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd421cc0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Collie</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce25a6f0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd421d1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Boxer</td>
        <td>Pure Breed</td>
        <td>Working</td>
    </tr>
    <tr>
        <td>ce25a754-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd421d7e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Weimaraner</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce25a6f0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd421de2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Boxer</td>
        <td>Pure Breed</td>
        <td>Working</td>
    </tr>
    <tr>
        <td>ce25a34e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd421eaa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce25aa38-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4220e4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Poodle</td>
        <td>Pure Breed</td>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>ce25aa9c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd422148-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce25aaf6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4221ac-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Coton de Tulear</td>
        <td>Pure Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce25ab5a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd422210-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Shih Tzu</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce25ac0e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42226a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Shih Tzu</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce25ac72-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4222ce-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever-Newfoundland Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce25accc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd422332-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Shih Tzu</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce25aea2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd422512-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Shetland Sheepdog</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce25aefc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd422576-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce25af56-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4225d0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Border Collie</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce25b014-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4226d4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce25b1ea-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4229fe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Border Collie</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce25ab5a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd422a94-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Shih Tzu</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce25b244-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd422b2a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>American Staffordshire Terrier</td>
        <td>Pure Breed</td>
        <td>Terrier</td>
    </tr>
    <tr>
        <td>ce47264a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd422e0e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Shih Tzu</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce25b5fa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd423034-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce25b5fa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd423098-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Other</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce25b6b8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4231ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Greyhound</td>
        <td>Pure Breed</td>
        <td>Hound</td>
    </tr>
    <tr>
        <td>ce2548e0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42321e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Cavalier King Charles Spaniel</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce2548e0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd423340-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Border Collie</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce25b9d8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4233fe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td></td>
    </tr>
    <tr>
        <td>ce25ba3c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd423462-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce2548e0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4234c6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Boxer-American Staffordshire Terrier Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce25bafa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42352a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Miniature Schnauzer</td>
        <td>Pure Breed</td>
        <td>Terrier</td>
    </tr>
    <tr>
        <td>ce25bb54-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42358e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Cavalier King Charles Spaniel-Poodle Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce25bc12-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4236b0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Other</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce225842-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd423714-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Shih Tzu</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce25bd2a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4237dc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Boston Terrier</td>
        <td>Pure Breed</td>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>ce25bea6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42389a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Poodle</td>
        <td>Pure Breed</td>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>ce25bf64-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4239c6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>German Shepherd Dog</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce25bf64-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd423a20-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Norfolk Terrier</td>
        <td>Pure Breed</td>
        <td>Terrier</td>
    </tr>
    <tr>
        <td>ce25bf64-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd423a84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Norfolk Terrier</td>
        <td>Pure Breed</td>
        <td>Terrier</td>
    </tr>
    <tr>
        <td>ce25bbb8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd423ae8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce25bfc8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd423b42-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever-Rottweiler Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce25c086-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd423c0a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce25900c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd423c6e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Nova Scotia Duck Tolling Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce25b3ca-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd423cd2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>German Shepherd Dog</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce25c144-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd423d90-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Australian Shepherd</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce252e78-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd423dea-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Greyhound</td>
        <td>Pure Breed</td>
        <td>Hound</td>
    </tr>
    <tr>
        <td>ce25c1da-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd423e4e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Staffordshire Bull Terrier-Great Dane Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce25c252-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd423eb2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce25c2ac-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd423f0c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Wire Fox Terrier</td>
        <td>Pure Breed</td>
        <td>Terrier</td>
    </tr>
    <tr>
        <td>ce25c306-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd423f70-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce25c414-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd423fd4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>French Spaniel</td>
        <td>Pure Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce25c4c8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42402e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Samoyed</td>
        <td>Pure Breed</td>
        <td>Working</td>
    </tr>
    <tr>
        <td>ce25c522-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd424092-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Coton de Tulear</td>
        <td>Pure Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce25c87e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42415a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce25c8d8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4241b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce25c9dc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd424218-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Poodle-Maltese Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce250588-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42442a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>German Shepherd Dog</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce25caa4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4244f2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce25ce8c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd424614-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Border Collie</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce25cf22-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd424678-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Maltese-Yorkshire Terrier Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce25c3ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4246dc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Cockapoo</td>
        <td>Popular Hybrid</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce25d436-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd424808-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Collie</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce25d4b8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd424862-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever-German Shepherd Dog Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce25d4b8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4248d0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>I Don&#x27;t Know</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce25d8fa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4249fc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Border Collie</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce25df26-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd424aba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce25e2a0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd424b1e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Pekingese-Shih Tzu Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce25e98a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd424b78-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce25e9ee-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd424bd2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Siberian Husky</td>
        <td>Pure Breed</td>
        <td>Working</td>
    </tr>
    <tr>
        <td>ce25ea52-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd424c36-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Shih Tzu</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce25eaa2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd424ede-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Rhodesian Ridgeback</td>
        <td>Pure Breed</td>
        <td>Hound</td>
    </tr>
    <tr>
        <td>ce25ec46-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd424f9c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce240c50-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42546a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Dalmatian</td>
        <td>Pure Breed</td>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>ce240c50-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4254ce-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Dalmatian</td>
        <td>Pure Breed</td>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>ce2259d2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd425604-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce25ee8a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4258ca-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Staffordshire Bull Terrier-Miniature Schnauzer Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce25efa2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd425dde-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Cocker Spaniel</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce25bd8e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd425e74-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce25ef48-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42620c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Greyhound</td>
        <td>Pure Breed</td>
        <td>Hound</td>
    </tr>
    <tr>
        <td>ce25b4e2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4262f2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Shetland Sheepdog</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce25f06a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42691e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Poodle</td>
        <td>Pure Breed</td>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>ce25f0c4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd426bf8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Puggle</td>
        <td>Popular Hybrid</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce25f128-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4271fc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce25f182-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42727e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Cockapoo</td>
        <td>Popular Hybrid</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce25f2fe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42740e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Beagle- Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce25f420-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4274e0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Rhodesian Ridgeback</td>
        <td>Pure Breed</td>
        <td>Hound</td>
    </tr>
    <tr>
        <td>ce25f47a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd427544-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Havanese</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce25f4de-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4275a8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce25f538-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd427616-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce25f5f6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42767a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Shih Tzu</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce25f5f6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4276de-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Shih Tzu</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce25f2fe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd427742-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce25f6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4278dc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Havanese</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce25f718-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd427940-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Yorkshire Terrier</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce25f772-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4279ae-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Dalmatian</td>
        <td>Pure Breed</td>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>ce25f772-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd427a12-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Dalmatian</td>
        <td>Pure Breed</td>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>ce25f772-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd427a76-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Belgian Malinois</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce2201b2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd427b48-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce25f88a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd427d32-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Rhodesian Ridgeback</td>
        <td>Pure Breed</td>
        <td>Hound</td>
    </tr>
    <tr>
        <td>ce25f8e4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd427d96-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labradoodle</td>
        <td>Popular Hybrid</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce25f948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd427dfa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Australian Shepherd</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce25fac4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd427f30-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Australian Shepherd</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce25fb78-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd427ff8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Dalmatian</td>
        <td>Pure Breed</td>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>ce25fbdc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42805c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Portuguese Water Dog</td>
        <td>Pure Breed</td>
        <td>Working</td>
    </tr>
    <tr>
        <td>ce25f420-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd428124-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Australian Shepherd-Australian Cattle Dog Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce25fdb2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4281f6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>I Don&#x27;t Know</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce25fe0c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42825a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>French Bulldog</td>
        <td>Pure Breed</td>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>ce25fe0c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4282be-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Boston Terrier</td>
        <td>Pure Breed</td>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>ce25f9a2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd428322-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce25fd58-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd428386-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce25feca-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4283ea-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce25ff2e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42844e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Pomeranian</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce25ff88-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4284b2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Whippet</td>
        <td>Pure Breed</td>
        <td>Hound</td>
    </tr>
    <tr>
        <td>ce25f59c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd428520-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Doodle</td>
        <td>Popular Hybrid</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce260046-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd428584-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Border Collie</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce260046-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4285e8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Border Collie</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce260046-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd428656-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Border Collie</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce2600aa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd428714-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Poodle-Labrador Retriever Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce260168-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd428886-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Pug-Wire Fox Terrier Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce2601c2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd428926-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Greyhound</td>
        <td>Pure Breed</td>
        <td>Hound</td>
    </tr>
    <tr>
        <td>ce260168-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4289c6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Pug-Wire Fox Terrier Mix</td>
        <td>Cross Breed</td>
        <td>Terrier</td>
    </tr>
    <tr>
        <td>ce260168-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd428a5c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Pug-Smooth Fox Terrier Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce2604d8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd428c32-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Pembroke Welsh Corgi</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce260596-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd428d68-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Australian Shepherd</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce2202e8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd428e08-7144-11e5-ba71-058fbc01cf0b</td>
        <td>German Shepherd Dog</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce26073a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42906a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce225842-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42913c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Shih Tzu</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce25ffec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4291a0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Wire Fox Terrier</td>
        <td>Pure Breed</td>
        <td>Terrier</td>
    </tr>
    <tr>
        <td>ce24e62a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd429204-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce24e62a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd429272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce26085c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4292d6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Bearded Collie-Beagle Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce26092e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42933a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce254e76-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42939e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Border Collie-Australian Cattle Dog Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce260e6a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4296d2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce2612d4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4298c6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Great Dane</td>
        <td>Pure Breed</td>
        <td>Working</td>
    </tr>
    <tr>
        <td>ce261392-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42992a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Brussels Griffon</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce26111c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4299f2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Poodle</td>
        <td>Pure Breed</td>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>ce25f718-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd429a56-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Irish Setter</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce26021c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd429aba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Poodle</td>
        <td>Pure Breed</td>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>ce261464-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd429b82-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce2614c8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd429bf0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce261590-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd429cb8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Puggle</td>
        <td>Popular Hybrid</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce24291a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd429de4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Shih Tzu</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce25900c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42a03c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Nova Scotia Duck Tolling Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce2619dc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42a10e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Poodle-Dachshund Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce261aa4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42a23a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>German Shepherd Dog</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce261a40-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42a29e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce25f59c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42a302-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce261bd0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42a366-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce261c98-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42a65e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Cocker Spaniel</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce261cfc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42a6c2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Miniature Schnauzer</td>
        <td>Pure Breed</td>
        <td>Terrier</td>
    </tr>
    <tr>
        <td>ce261dba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42a78a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Shetland Sheepdog</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce261dba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42a7ee-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Shetland Sheepdog</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce261edc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42a8c0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce261f40-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42a92e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Pug-Chihuahua Mix</td>
        <td>Cross Breed</td>
        <td></td>
    </tr>
    <tr>
        <td>ce25fe70-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42a992-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce261fa4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42aac8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Coton de Tulear</td>
        <td>Pure Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce261ffe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42ab9a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce262062-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42abfe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce2620bc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42ac62-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Papillon</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce262184-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42acbc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mastiff</td>
        <td>Pure Breed</td>
        <td>Working</td>
    </tr>
    <tr>
        <td>ce225842-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42ad20-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Shih Tzu</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce262120-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42ade8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Polish Lowland Sheepdog</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce2622a6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42b25c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Miniature Pinscher</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce262300-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42b2c0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>English Springer Spaniel</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce262364-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42b32e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>I Don&#x27;t Know</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce2623c8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42b392-7144-11e5-ba71-058fbc01cf0b</td>
        <td>American Eskimo Dog</td>
        <td>Pure Breed</td>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>ce2624ea-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42b78e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>German Shepherd Dog</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce26242c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42b84c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Soft Coated Wheaten Terrier</td>
        <td>Pure Breed</td>
        <td>Terrier</td>
    </tr>
    <tr>
        <td>ce26267a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42bc66-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce25be42-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42c0e4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Chinese Crested-Poodle Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce2627a6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42c486-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce26280a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42c51c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce24c8a2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42c7ec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>American Pit Bull Terrier-Australian Cattle Dog Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce5c19b0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42cd96-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Shih Tzu</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce2628c8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42d50c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Catahoula Leopard Dog-Rottweiler Mix</td>
        <td>Cross Breed</td>
        <td></td>
    </tr>
    <tr>
        <td>ce2629f4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42d732-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Shih Tzu</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce262d3c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42d930-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Belgian Tervuren</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce262d3c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42d994-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Belgian Tervuren</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce262e5e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42db2e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Greyhound</td>
        <td>Pure Breed</td>
        <td>Hound</td>
    </tr>
    <tr>
        <td>ce262ecc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42db92-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Nova Scotia Duck Tolling Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce261edc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42dd22-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce262f8a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42dd90-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Australian Shepherd-Border Collie Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce262fee-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42ddfe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Bernese Mountain Dog</td>
        <td>Pure Breed</td>
        <td>Working</td>
    </tr>
    <tr>
        <td>ce2631d8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42dff2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>West Highland White Terrier</td>
        <td>Pure Breed</td>
        <td>Terrier</td>
    </tr>
    <tr>
        <td>ce137fca-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42e060-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Poodle-Shih Tzu Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce263368-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42e204-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Boxer-Border Collie Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce135e14-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42e33a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Shih Tzu</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce26348a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42e39e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Other</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce2634f8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42e40c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Cavalier King Charles Spaniel</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce2633cc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42e4de-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce2635c0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42e542-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Border Collie</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce263728-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42e678-7144-11e5-ba71-058fbc01cf0b</td>
        <td>I Don&#x27;t Know</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce263782-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42e6e6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce2637dc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42e74a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Great Dane-Irish Wolfhound Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce26355c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42e880-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Pyrenean Shepherd</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce26355c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42e8e4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Pyrenean Shepherd</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce263c0a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42e948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce263d72-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42ea1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce263e3a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42ea7e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>German Shepherd Dog</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce2641dc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42eb50-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Other</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce26429a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42ebfa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Chinese Shar-Pei</td>
        <td>Pure Breed</td>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>ce264330-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42ec90-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mastiff</td>
        <td>Pure Breed</td>
        <td>Working</td>
    </tr>
    <tr>
        <td>ce264330-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42ed6c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>French Bulldog</td>
        <td>Pure Breed</td>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>ce2646e6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42eeac-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Australian Shepherd</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce2646e6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42ef4c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Australian Shepherd</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce2646e6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42efec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Australian Shepherd</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce264876-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42f082-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Dachshund</td>
        <td>Pure Breed</td>
        <td>Hound</td>
    </tr>
    <tr>
        <td>ce2647ea-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42f122-7144-11e5-ba71-058fbc01cf0b</td>
        <td>I Don&#x27;t Know</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce264cd6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42f1b8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce2643b2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42f258-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Maltese</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce265000-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42f2f8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce265000-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42f38e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce263dd6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42f42e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Cavalier King Charles Spaniel</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce263110-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42f51e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labradoodle</td>
        <td>Popular Hybrid</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce263110-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42f5be-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labradoodle</td>
        <td>Popular Hybrid</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce263b4c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42f6b8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Border Collie</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce264c4a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42f726-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Lhasa Apso</td>
        <td>Pure Breed</td>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>ce2656cc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42f78a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Greyhound</td>
        <td>Pure Breed</td>
        <td>Hound</td>
    </tr>
    <tr>
        <td>ce406a26-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42f7ee-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce2258a6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42f85c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Shih Tzu</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce265334-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42f8c0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Dogue de Bordeaux-Boxer Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce265e56-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42f992-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Doodle</td>
        <td>Popular Hybrid</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce265f28-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42fac8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Belgian Malinois</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce265f8c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42fb2c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Greyhound</td>
        <td>Pure Breed</td>
        <td>Hound</td>
    </tr>
    <tr>
        <td>ce265fe6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42fb9a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Belgian Malinois</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce266086-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42fbfe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Akita</td>
        <td>Pure Breed</td>
        <td>Working</td>
    </tr>
    <tr>
        <td>ce2660ea-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42fc62-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce26627a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42fe06-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce2662de-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42fe6a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce266342-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42fece-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce266400-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42ff32-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce2258a6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd42ffa0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Australian Shepherd</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce26639c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd430004-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Bulldog</td>
        <td>Pure Breed</td>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>ce261e1e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd430068-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Doodle</td>
        <td>Popular Hybrid</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce266522-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4300d6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>American Staffordshire Terrier-Catahoula Leopard Dog Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce26657c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd43013a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce26663a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd43020c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce266752-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd430338-7144-11e5-ba71-058fbc01cf0b</td>
        <td>German Shepherd Dog</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce2667b6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd43039c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Scottish Terrier</td>
        <td>Pure Breed</td>
        <td>Terrier</td>
    </tr>
    <tr>
        <td>ce266932-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4304d2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Bulldog</td>
        <td>Pure Breed</td>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>ce26698c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd430536-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Shih Tzu-Pekingese Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce266a54-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4305a4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce266aae-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd43066c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Rat Terrier</td>
        <td>Pure Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce266b6c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4306da-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Border Terrier</td>
        <td>Pure Breed</td>
        <td>Terrier</td>
    </tr>
    <tr>
        <td>ce266bd0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd43073e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Poodle</td>
        <td>Pure Breed</td>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>ce266c2a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4307a2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce266ce8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd43086a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Boxer</td>
        <td>Pure Breed</td>
        <td>Working</td>
    </tr>
    <tr>
        <td>ce266d92-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4308d8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Beagle-Labrador Retriever Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce266df6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd430bc6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Australian Shepherd</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce266f18-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd430dc4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce266f7c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd430e28-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Dachshund-Beagle Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce266f18-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd430e96-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Silky Terrier</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce26709e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd431030-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Shetland Sheepdog</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce26715c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd431102-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce2671c0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd431170-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Doodle</td>
        <td>Popular Hybrid</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce2672d8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4311de-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Scottish Terrier</td>
        <td>Pure Breed</td>
        <td>Terrier</td>
    </tr>
    <tr>
        <td>ce26721a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd431242-7144-11e5-ba71-058fbc01cf0b</td>
        <td>-German Shepherd Dog Mix</td>
        <td>Cross Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce267332-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4312a6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Boxer</td>
        <td>Pure Breed</td>
        <td>Working</td>
    </tr>
    <tr>
        <td>ce267396-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd431300-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce2673f0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd431364-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Keeshond</td>
        <td>Pure Breed</td>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>ce267512-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd431800-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce267576-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd431904-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce2675da-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd431972-7144-11e5-ba71-058fbc01cf0b</td>
        <td>German Shorthaired Pointer</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce26763e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4319e0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Coton de Tulear</td>
        <td>Pure Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce2676fc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd431a44-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce2677c4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd431ddc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce267cce-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4323c2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Belgian Malinois</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce267d82-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd432782-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Chihuahua</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce267e18-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd432818-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Poodle</td>
        <td>Pure Breed</td>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>ce267eae-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd432bf6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>American Pit Bull Terrier</td>
        <td>Pure Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce268016-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd432cf0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Bernese Mountain Dog</td>
        <td>Pure Breed</td>
        <td>Working</td>
    </tr>
    <tr>
        <td>ce26814c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd433632-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce2681d8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd433ce0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce26826e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd433d58-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce268390-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd433dbc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>German Shepherd Dog-Chow Chow Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce268426-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd433e16-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever-Golden Retriever Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce2684b2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd433eb6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Weimaraner</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce2685de-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd433fec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce268700-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd43417c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>German Shepherd Dog</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce26878c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4341ea-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Border Collie-Australian Shepherd Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce268822-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd43424e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever- Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce2688ae-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4342b2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Border Collie</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce268912-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd434320-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce268c1e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd43458c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce268c78-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4345f0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Beagle-Australian Cattle Dog Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce268cdc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd434654-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce2680ac-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd43471c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Pembroke Welsh Corgi-Great Pyrenees Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce268f2a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4347ee-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce269178-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd434b22-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce269178-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd434b86-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce2692a4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd434bf4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce269308-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd434c58-7144-11e5-ba71-058fbc01cf0b</td>
        <td>German Shepherd Dog</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce268b56-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd434d8e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Australian Shepherd-Pembroke Welsh Corgi Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce26942a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd434df2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce26961e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd435068-7144-11e5-ba71-058fbc01cf0b</td>
        <td>German Shepherd Dog</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce26961e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4350d6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce2696dc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd43520c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Cavalier King Charles Spaniel</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce2695ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4352e8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Australian Shepherd</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce269740-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd435388-7144-11e5-ba71-058fbc01cf0b</td>
        <td>French Bulldog</td>
        <td>Pure Breed</td>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>ce2697a4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd435428-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce26986c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd435568-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Doodle</td>
        <td>Popular Hybrid</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce2698d0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd435608-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Chesapeake Bay Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce269aba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd435748-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Poodle</td>
        <td>Pure Breed</td>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>ce269b14-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4357de-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Doodle</td>
        <td>Popular Hybrid</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce269bd2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd43591e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce269c36-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4359be-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Maltese-Poodle Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce269c9a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd435a5e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce269cf4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd435b8a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labradoodle</td>
        <td>Popular Hybrid</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce269d58-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd435c2a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Cavalier King Charles Spaniel</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce269dbc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd435cca-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce269dbc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd435d2e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce269998-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd435d9c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Dachshund</td>
        <td>Pure Breed</td>
        <td>Hound</td>
    </tr>
    <tr>
        <td>ce269e7a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd435e6e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Havanese</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce269f38-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd435ed2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce269f9c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd435f36-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce26292c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd435f9a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Boxer</td>
        <td>Pure Breed</td>
        <td>Working</td>
    </tr>
    <tr>
        <td>ce26a000-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd435ffe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce26292c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd43606c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Boxer</td>
        <td>Pure Breed</td>
        <td>Working</td>
    </tr>
    <tr>
        <td>ce26a064-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4360d0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>German Shepherd Dog</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce26a122-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4361a2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Border Collie</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce26a3a2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd436210-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce26a46a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4362d8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Australian Shepherd</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce26a4ce-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd43633c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Afghan Hound-Golden Retriever Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce2632a0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4363a0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce26a528-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd43640e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Dachshund</td>
        <td>Pure Breed</td>
        <td>Hound</td>
    </tr>
    <tr>
        <td>ce26a58c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd436472-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Shorkie</td>
        <td>Popular Hybrid</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce26a654-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4365a8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Field Spaniel</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce26a712-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd436616-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce26a6b8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd43667a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Cavalier King Charles Spaniel</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce26a83e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd436742-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Poodle</td>
        <td>Pure Breed</td>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>ce26a906-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4367a6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Chihuahua-American Staffordshire Terrier Mix</td>
        <td>Cross Breed</td>
        <td></td>
    </tr>
    <tr>
        <td>ce26a96a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd436814-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Newfoundland</td>
        <td>Pure Breed</td>
        <td>Working</td>
    </tr>
    <tr>
        <td>ce26aa28-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4368dc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce26aaf0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4369ae-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Yorkshire Terrier</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce26ac08-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd436a76-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Australian Cattle Dog</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce26acc6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd436ae4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Other</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce26ad84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd436c10-7144-11e5-ba71-058fbc01cf0b</td>
        <td>English Cocker Spaniel</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce26adde-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd436c74-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Boxer</td>
        <td>Pure Breed</td>
        <td>Working</td>
    </tr>
    <tr>
        <td>ce26ae38-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd436ce2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Portuguese Water Dog</td>
        <td>Pure Breed</td>
        <td>Working</td>
    </tr>
    <tr>
        <td>ce26aeec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd436d46-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labradoodle</td>
        <td>Popular Hybrid</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce26b202-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd436daa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce26b266-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd436e7c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Pug</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce26b266-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd436ee0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Bluetick Coonhound</td>
        <td>Pure Breed</td>
        <td>Hound</td>
    </tr>
    <tr>
        <td>ce26b266-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd436f44-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Pug-Miniature Pinscher Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce26b2c0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4371ce-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Shetland Sheepdog</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce26b3ce-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd43723c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Whippet-Chinese Shar-Pei Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce26b4fa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd437304-7144-11e5-ba71-058fbc01cf0b</td>
        <td>German Shepherd Dog</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce26ba4a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4373e0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>German Shepherd Dog</td>
        <td>Pure Breed</td>
        <td>Herding</td>
    </tr>
    <tr>
        <td>ce26b9c8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd43744e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Boxer-Bulldog Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce26bedc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd437516-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Portuguese Water Dog</td>
        <td>Pure Breed</td>
        <td>Working</td>
    </tr>
    <tr>
        <td>ce26c31e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd437584-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Other</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce26c6c0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4376c4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Portuguese Water Dog</td>
        <td>Pure Breed</td>
        <td>Working</td>
    </tr>
    <tr>
        <td>ce26c6c0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd437728-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Portuguese Water Dog</td>
        <td>Pure Breed</td>
        <td>Working</td>
    </tr>
    <tr>
        <td>ce26c6c0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd43778c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Portuguese Water Dog</td>
        <td>Pure Breed</td>
        <td>Working</td>
    </tr>
    <tr>
        <td>ce26c9d6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4377e6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce26cdbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd43784a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce26d610-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4378a4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labradoodle</td>
        <td>Popular Hybrid</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce26d82c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd437c1e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Papillon</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce26d73c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd437c82-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Poodle</td>
        <td>Pure Breed</td>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>ce26ae92-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd437e58-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce26d958-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4381f0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Poodle</td>
        <td>Pure Breed</td>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>ce26da16-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd438254-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Portuguese Water Dog</td>
        <td>Pure Breed</td>
        <td>Working</td>
    </tr>
    <tr>
        <td>ce26da7a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4385ec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Vizsla- Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce26db9c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd438ac4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Rhodesian Ridgeback</td>
        <td>Pure Breed</td>
        <td>Hound</td>
    </tr>
    <tr>
        <td>ce26dbf6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd438b5a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Vizsla</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce26dc5a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd438eac-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Portuguese Water Dog</td>
        <td>Pure Breed</td>
        <td>Working</td>
    </tr>
    <tr>
        <td>ce26dcb4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd4391d6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce26dd18-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd439dfc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce26dd7c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd439e6a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce26de3a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd439f3c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce26def8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd43a004-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever-Poodle Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce26e01a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd43a13a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce26dfb6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd43a19e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Maltese</td>
        <td>Pure Breed</td>
        <td>Toy</td>
    </tr>
    <tr>
        <td>ce26e074-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd43a202-7144-11e5-ba71-058fbc01cf0b</td>
        <td>American Eskimo Dog-Russell Terrier Mix</td>
        <td>Cross Breed</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce26a8a2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd43a2d4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce26e1fa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd43a39c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>ce26e2b8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd43a400-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Russell Terrier</td>
        <td>Pure Breed</td>
        <td>Terrier</td>
    </tr>
    <tr>
        <td>ce26e13c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd43a464-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>None</td>
    </tr>
    <tr>
        <td>ce26e376-7144-11e5-ba71-058fbc01cf0b</td>
        <td>fd43a4c8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever</td>
        <td>Pure Breed</td>
        <td>Sporting</td>
    </tr>
</table>
<span style="font-style:italic;text-align:center;">20845 rows, truncated to displaylimit of 1000</span>



## 2. Joining More than 2 Tables

In theory, you can join as many tables together as you want or need.  To join multiple tables you take the same approach as we took when we were joining two tables together: list all the fields you want to extract in the SELECT statement, specify which table they came from in the SELECT statement, list all the tables from which you will need to extract the fields in the FROM statement, and then tell the database how to connect the tables in the WHERE statement.

To extract the user_guid, user's state of residence, user's zip code, dog_guid, breed, breed_type, and breed_group for all animals who completed the "Yawn Warm-up" game, you might be tempted to query:

```mySQL
SELECT c.user_guid AS UserID, u.state, u.zip, d.dog_guid AS DogID, d.breed, d.breed_type, d.breed_group
FROM dogs d, complete_tests c, users u
WHERE d.dog_guid=c.dog_guid 
   AND c.user_guid=u.user_guid
   AND c.test_name="Yawn Warm-up";
```
This query focuses the relationships primarily on the complete_tests table.  However, it turns out that our Dognition dataset has only NULL values in the user_guid column of the complete_tests table.  If you were to execute the query above, you would not get an error message, but your output would have 0 rows.  However, the power of relational databases will come in handy here.  You can use the dogs table to link the complete_tests and users table (pay attention to the difference between the WHERE statement in this query vs. the WHERE statement in the query above):

```mySQL
SELECT d.user_guid AS UserID, u.state, u.zip, d.dog_guid AS DogID, d.breed, d.breed_type, d.breed_group
FROM dogs d, complete_tests c, users u
WHERE d.dog_guid=c.dog_guid 
   AND d.user_guid=u.user_guid
   AND c.test_name="Yawn Warm-up";
```

Of note, joins are very resource intensive, so try not to join unnecessarily.  In general, the more joins you have to execute, the slower your query performance will be.

**Question 6: How would you extract the user_guid, membership_type, and dog_guid of all the golden retrievers who completed at least 1 Dognition test (you should get 711 rows)?**


```python
%%sql
SELECT DISTINCT d.user_guid, u.membership_type, d.dog_guid, d.breed
FROM dogs d, complete_tests c, users u
WHERE d.dog_guid=c.dog_guid
AND d.user_guid=u.user_guid
AND d.breed="golden retriever";
```

    711 rows affected.





<table>
    <tr>
        <th>user_guid</th>
        <th>membership_type</th>
        <th>dog_guid</th>
        <th>breed</th>
    </tr>
    <tr>
        <td>ce135ab8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce13507c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce1389d4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd27efb2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce21f122-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd3d03fc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce220bb2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd3d10cc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce2237f4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd3d3b24-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce2243e8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd3d4b8c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce240b24-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd3fe96e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce24476a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd404ff8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce245cdc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd4059e4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce246be6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd40738e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce249774-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd408ed2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce249c6a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd40945e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce24c1c2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd40ee72-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce24a106-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd40f822-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce24d572-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd4107cc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce24d914-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd410cb8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce24de50-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd411690-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce24e116-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd4118e8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce24e684-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd411b0e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce253a6c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd418b02-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce2556a0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd41c5f4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce255c54-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd41c84c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce257fc2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd41db34-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce2585a8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd41e232-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce22680a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd41e28c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce258a08-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd41e796-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce259dd6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd421400-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce25c306-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd423f70-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce25caa4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd4244f2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce25f4de-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd4275a8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce26092e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd42933a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce261a40-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd42a29e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce261bd0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd42a366-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce261ffe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd42ab9a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce262062-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd42abfe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce26280a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd42c51c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce263782-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd42e6e6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce263d72-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd42ea1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce264cd6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd42f1b8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce2660ea-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd42fc62-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce266342-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd42fece-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce266400-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd42ff32-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce26663a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd43020c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce266f18-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd430dc4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce2676fc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd431a44-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce26814c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd433632-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce269178-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd434b22-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce269178-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd434b86-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce26c9d6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd4377e6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce26e01a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd43a13a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce26e1fa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd43a39c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce26e31c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd43a91e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce26ecd6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd43b026-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce2247a8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd43d9de-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce271f44-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd440332-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce2758c4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd44236c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce277214-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd443956-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce27732c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd443a82-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce2785ec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd4471e6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce27d998-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd44a77e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce27e910-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd44ca4c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce27f158-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd44d1fe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce27f40a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd44d50a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce27bf3a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd44d9c4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce280b0c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd44f31e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce283c30-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd45154c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce283c30-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd4515d8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce269114-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd453004-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce267698-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd4539dc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce2853a0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd454c24-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce28623c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd455cbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce286408-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd455e4e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce2869da-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd45697a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce286ba6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd456d58-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce287d80-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd4595bc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce2898ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd45a8e0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce28b5f2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd45b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce246be6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd45d6ee-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce245cdc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd45eb98-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce28d6e0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd45f778-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce28e18a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd460308-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce28e554-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd4607ea-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce28eaa4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd460dbc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce28f828-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd461b72-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce290642-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd464a5c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce295f2a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd46926e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce296d9e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd46b5e6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce297d98-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd46c73e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce29d018-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd471dba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce29eb02-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd473520-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce29ec24-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd4735de-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce29f444-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd473f84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce2a0394-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd476694-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce2a42b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd478cd2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce2a57e0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd47a546-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce2a874c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd47f474-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce2a8d64-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd47f898-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce2ad896-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd484302-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce2b11a8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd489c6c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce2b4c40-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd48ba62-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce2b65fe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd48f554-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce2b6ebe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd48fba8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce2b7274-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd48ff22-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce2b91d2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd491caa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce2b9524-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd49201a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce2c7a52-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd4a0f20-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce2cca84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd4a48dc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce2d5350-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd4a9cb0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce2d5774-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd4a9fb2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce2d6d5e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd4ab2e0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce2dc77c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd4b11f4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce2e0818-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd4b3f76-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce2e1434-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd4b493a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce2e14c0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd4b499e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce2e35a4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd4b667c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce2e4576-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd4b716c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce2e893c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd4bc838-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce2fdbe8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd4d165c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce302a9e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd4e60c0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce316b5c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd501582-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce31aa9a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd504462-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce31b18e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd505f4c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce325c92-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd510a96-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce32666a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd511770-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce32b818-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd5167e8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce32cf38-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd519e02-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce32dee2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd51ab68-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce330840-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd51d43a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce33102e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd51e916-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce3343c8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd520cde-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce33562e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd52235e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce337b18-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd5250f4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce337f1e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd5265c6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce338798-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd526e40-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce33b2e0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd52915e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce33d3c4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd529988-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce33d9f0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd52a018-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce34bc94-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd53b2aa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce351572-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd5406e2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce3515d6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd540778-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce354240-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd544a80-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce355c1c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd54622c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce3562a2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd546650-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce3587a0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd546fc4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce35c8be-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd54cf6e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce35eeca-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd54f96c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce361e7c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd551afa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce361dd2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd551b68-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce36311e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd553238-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce3676f6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd558d3c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce36b60c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd55b442-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce36d9ca-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd55f5ce-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce37481a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd56618a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce3757ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd567166-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce377bd2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd56a834-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce38476a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd57629c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce3875e6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd577660-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce390a60-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd57e7da-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce3a85e8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd5957e6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce3acf6c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd5990f8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce3af226-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd59c988-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce3b7e30-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd5a4fa2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce3b8542-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd5a5560-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce3b9a78-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd5a7b76-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce3c0288-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3</td>
        <td>fd5ae002-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce3c1bf6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd5af7cc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce3c20b0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd5afb5a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce3c38de-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd5b1dec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce3c3f96-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd5b35e8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce3a6ebe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd5b46fa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce3c612e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd5b49b6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce3c7dee-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd5b5280-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce339990-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd5b57a8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce3c84a6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd5b5942-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce3c85d2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd5b5adc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce3cb732-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd5b9f10-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce3cd960-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd5bc148-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce3d0f34-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd5be704-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce3d1c5e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd5c0108-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce3d1fe2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd5c0428-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce3d33d8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd5c1a80-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce3d35d6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd5c1c6a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce3d4724-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd5c29da-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce3d4b0c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd5c2e26-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce3d55fc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd5c4fbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce3d9c42-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd5c7d36-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce3da21e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd5c82f4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce3db5ec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3</td>
        <td>fd5c9b2c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce3db84e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd5ca61c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce3dc2b2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd5cbdc8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce3de242-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd5cdbd2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce3df8fe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd5cf162-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce3e013c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd5cfa04-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce3e2cac-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd5d213c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce3e4dae-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd5d44fa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce3e7630-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd5d850a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce3ef01a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd5e1308-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce3ef39e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd5e16aa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce3f4574-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd64732e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce2258a6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd6487c4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce3f56d6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd649548-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce3f957e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd651554-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce3fd78c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd656d2e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce3fe4c0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd658728-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce3ff38e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd65983a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce40167a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd65cd96-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce401e2c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd65f1e0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce40429e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd65ff8c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce404f78-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3</td>
        <td>fd660a2c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce405dc4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd661f26-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce40813c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd666670-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce409316-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd667afc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce40bb34-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6698ca-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce40d0ce-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd66a1e4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce40ec30-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd66da2e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce40f072-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd66dad8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce413564-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd673e38-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce416ac0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6760f2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce4170ec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd67626e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce41770e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd678168-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce417934-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6788fc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce459546-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd67b5fc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce459fd2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd67be9e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce46ee32-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd67f79c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce46f58a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd68008e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce46fa44-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6805f2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce4702be-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd680a3e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce47109c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3</td>
        <td>fd68127c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce4745d0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd68566a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce478ac2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd687960-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6621e4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd6899cc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6635b2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3</td>
        <td>fd68b150-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6676d0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd68ed6e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6676d0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd68edd2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce66aaec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6913ac-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce66c5b8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd69313e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce66c7de-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd693328-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce66c8f6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd6933f0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce66e7aa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd694688-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce66ffce-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd6975ae-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce67191e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd698a9e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce67208a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd698eea-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6779fe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3</td>
        <td>fd69d7e2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6c3fc0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd69fd76-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6c4ee8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6a0640-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6c663a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6a169e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6ce394-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6a6590-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6cf3fc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6a7490-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6ce308-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6a7850-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6d0216-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6a7d6e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6ce736-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd6aa5b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6d1abc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6aab7c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6d0c98-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3</td>
        <td>fd6aba18-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6d55fe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6ac030-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6d513a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6ac26a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6d7070-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6ad304-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6d7836-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6ad66a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6d81b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6adea8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6d8a38-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd6ae7f4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6d892a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd6af2d0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6d956e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6bd5d8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6d94e2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6bda60-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6d9960-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6bdaf6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6d9e42-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6be046-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6d9ca8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6befb4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6dd6dc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6c024c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6da7de-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd6c0fe4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6dee92-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6c1aca-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6def78-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6c1c8c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6e0fe4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6c655c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6e187c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6c6d18-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6e6c32-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3</td>
        <td>fd6cc6d2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6e6840-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6cd2e4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6e9428-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6d12ae-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6ea6ca-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6d1f06-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6ec3da-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6d2654-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6ec380-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd6d26b8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6ec380-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd6d304a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6ed352-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3</td>
        <td>fd6d5bb0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6ed6f4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6d5c1e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6ee64e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd6d6718-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6ef22e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd6d79d8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6ef170-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6d7aaa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6f055c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6d8db0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6f0a70-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd6d9724-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6ce736-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd6db18c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6f3676-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6dc618-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6f37f2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6dc776-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6f5d72-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6df23c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6f60ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6df818-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6f6290-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6df8f4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6f6812-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6e086c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6f72b2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6e273e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6ee52c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd6e3ec2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6fa476-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6e4070-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6faa98-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6e45c0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6fb27c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6e4b7e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6f66fa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6e4fc0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6fcc30-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6e79dc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6fd054-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6e8760-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6fcffa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd6e8832-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6fc988-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd6e90ac-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6fd8f6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6e91ce-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6fe260-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6e9b1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6fe5e4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6e9e30-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6fe6a2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6ea1a0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce70038a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6ea844-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6ff296-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6eac0e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7012b2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6eb046-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce70173a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6eb4f6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce701afa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6ebadc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce701b5e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6ebb40-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce703f08-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6ef77c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce703e54-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6ef8b2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce704016-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6ef916-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce70a27c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6f68e2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce70a61e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6f6ebe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce70b3a2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6f7efe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce70bbf4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6f869c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce70f8e4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6fc738-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce710d2a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6fdbd8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce71152c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6fe29a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce712076-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd6ff8a2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce712364-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd700c20-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce714f56-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd702714-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce715d7a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd702d4a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce715fc8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd703146-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce714f56-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd7032f4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce716270-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd703574-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7164be-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd70375e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7177ec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd7049f6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce717954-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3</td>
        <td>fd704b18-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce718e6c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd7078c2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce719c18-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd7085ce-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce71dac0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd70b382-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce70917e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd70d376-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce71f21c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd70de48-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce71ff00-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd70f054-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce720018-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd70f180-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce72118e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd71038c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce721558-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd710706-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce724c3a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd71439c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce724e74-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd714612-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce726044-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd715990-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce726738-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd716228-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce726828-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd71628c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce727c82-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd718906-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce729fa0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd71a562-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce728bf0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3</td>
        <td>fd71ac1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce72b27e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd71ba20-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce72c1b0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd71cfa6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce72d722-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd71fe22-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6ce736-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd7204d0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce72e4ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd720e4e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce72e33e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3</td>
        <td>fd72138a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce72e33e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3</td>
        <td>fd72168c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce732470-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd724670-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7343f6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd727802-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce73508a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd728676-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce736994-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd72a642-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce738424-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd72a6ce-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce739220-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd72c7d0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce738ee2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd72cb40-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7395c2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd72cd84-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce739cf2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd72d8a6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce720018-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd72f110-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce73ba16-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd72f98a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce73c718-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd731e92-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce73fb3e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd733ef4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce73fe7c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd7343fe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7400a2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd7345f2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce74142a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd735ace-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce741b1e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd7374d2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce743702-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd739dcc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7397e8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd73b3a2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce74670e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd73b6e0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7486bc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd73f3f8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7493fa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd7404ce-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce74acb4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3</td>
        <td>fd74277e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6e6296-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd744f6a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce74dbda-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd745e92-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce74dd60-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd74619e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce6c663a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd748098-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce750394-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd748e26-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce750c04-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd74b180-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce750ba0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd74b1e4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce751136-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd74b5ea-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7540d4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd74d296-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce755222-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd74e86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce758814-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd753e70-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce75bba4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd756580-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce75c9f0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd75804c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce75ebc4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd759ad2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce76611c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd7610c0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce76611c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd761124-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce76777e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd764ad6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce767cce-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd764fa4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7689ee-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd765c06-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce761be4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd766520-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce769290-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd76678c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce76b608-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd767056-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce76c6f2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd767bc8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce76c88c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd767d62-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce765adc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd768924-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce76d124-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd76a558-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce76d3a4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3</td>
        <td>fd76a878-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce76f7e4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd76c970-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce76f7e4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd76c9d4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce773b32-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd77021e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce775cb6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd772532-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce775cb6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd772596-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7817be-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd77e166-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce78382a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd77edf0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce78484c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd77fab6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce784e78-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd780ace-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7859a4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd7827ca-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7859a4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd782c0c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7859a4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd782c70-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce787164-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd783f6c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce78740c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd784296-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce788046-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd7851e6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7891bc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd7868c0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce789662-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd7883c8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce78aa08-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd788b66-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce78c60a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd789426-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce78ce34-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd789f16-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce78d712-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd78aaec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce78eda6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd78ca36-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce78f7ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd78e9c6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce78fe0e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd78f114-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7901ce-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd78f4f2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7912d6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd790da2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce79307c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd7923aa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce79582c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd795ea6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce795b1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd79645a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce797186-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd797c1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce79776c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd798610-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce797abe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd798dd6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce798e78-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd79bdb0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce798ff4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd79be78-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7996c0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd79c6e8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce79b25e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd79d78c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce79cc8a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd79e6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce79d1d0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd79ed1c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce79ea80-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd7a1daa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce79f4da-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd7a2552-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce79fbf6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd7a2fac-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7a4188-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd7a79a8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7a4cf0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd7a7e58-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7a5b82-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd7a941a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce796e8e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd7a9c62-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7a7068-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd7aa84c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7a7428-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd7aaaae-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7a7428-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd7aab1c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7a84ea-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd7ad632-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7a8f9e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd7ae51e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7a978c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd7aece4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce135cf2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd7af63a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7ad7e2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd7b373a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7ad58a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd7b3870-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7ae282-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd7b42c0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7af66e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd7b56c0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7b0a78-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd7b6fe8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7b2436-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd7ba512-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7b489e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd7baf76-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce738ee2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd7bb4c6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7b7f1c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3</td>
        <td>fd7c0cd2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7b943e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd7c3ca2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7b94a2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd7c41d4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7b75bc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>fd7c5fc0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7bd232-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd7cc4a6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7bd8c2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd7ccd02-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7bd926-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd7cce7e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7b75bc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>fd7cd6e4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7befa6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd7ce904-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7bf8f2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd7cf6b0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7c0efa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd7d30d0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7c0978-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd7d345e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7b75bc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>fd7d48d6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7c3f6a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd7d4f34-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7b75bc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>fd7daba0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7b75bc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>fd7db37a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7b75bc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>fd7ddbc0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7b75bc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>fd7e22ce-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7c6d00-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd7e2616-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7c6f9e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd7e274c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7b75bc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>fd7e52e4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7c4866-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd7e5da2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7c7ade-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd7e6c52-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7c8ac4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd7e9362-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7cb026-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd7ec724-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7cb620-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd7ed494-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7cb7f6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd7ed534-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7cb67a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd7ed66a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7b75bc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>fd7ed9bc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7b75bc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>fd7ee100-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7ccef8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd7f2dea-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7cdc40-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd7f4b9a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7b75bc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>fd7fa7a2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7d2bfa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd7ff41e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7d3df2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd800b52-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7d49e6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd801b56-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7d49e6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd801bba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7d619c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd805620-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7b75bc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>fd806782-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7d8dc0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd80781c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7d9f86-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd80ba3e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7daf76-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd80d820-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7dc60a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd812d66-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7ddf00-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd815ab6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7b75bc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>fd81ab06-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7b75bc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>fd81ae08-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7b75bc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>fd82b5d2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7e3ab8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd82b988-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7b75bc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>fd82bb5e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7e552a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd82d86e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7e7fbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd8308c0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7b75bc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>fd830f00-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7b75bc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>fd837cba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7eb16e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3</td>
        <td>fd8392d6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7eb66e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd83a0d2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7ec078-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd83cd96-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7ef994-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd83ffa0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7efeb2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd8407f2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7f0cd6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd88aee2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7f4d22-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd89160c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7f534e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd891e86-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7f5b00-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd8925d4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7f8382-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fd893290-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7f886e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd893d58-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7fb6a4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd898ede-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7fb6a4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd8990be-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7b75bc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>fd8998ac-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7fcf54-7144-11e5-ba71-058fbc01cf0b</td>
        <td>5</td>
        <td>fd89d628-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7fd3d2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fd89f040-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce801d06-7144-11e5-ba71-058fbc01cf0b</td>
        <td>5</td>
        <td>fd8a2966-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce800af0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fd8a34ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce809be6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fda9db62-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce80d82c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fdaa3e5e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce80e222-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fdaa46ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7e80cc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdaa5cae-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce81275a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdaa937c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce81924e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdab0fb4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce819f96-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdab1d7e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce81a270-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdab1e50-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce81c2aa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdab6162-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce81ca98-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdab67b6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce81cfc0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdab6c5c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce820f76-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdab9506-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce82234e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdabbb1c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce8235fa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdabcbe8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce824090-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdabda5c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce8244c8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdabddf4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce825008-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdabe5b0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce82873a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fdac262e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce823cd0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdac29e4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce828bea-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdac2b24-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce828fd2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdac302e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce829f22-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdac3ee8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce82b084-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdac5130-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce82b4c6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdac569e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce82d26c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdac917c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce833d24-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdacff9a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce838482-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fdad2d3a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce83965c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fdad5666-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce839ac6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fdad5918-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce83aa48-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdad67c8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce83bf42-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdad7c0e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce840db2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdadd604-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce841f46-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdade310-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce842ed2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fdadf260-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce842888-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdadf670-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce850564-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fdaeef44-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce8512d4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdaeff2a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce85240e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fdaf0b0a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce8524a4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fdaf0b6e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce852b5c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdaf1474-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce853020-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdaf1a28-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce8445f2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdaf22ca-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce85325a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdaf232e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce869abe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdafdfe4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce86d1b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fdb0143c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce86dd12-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fdb021f2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce86f9f0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3</td>
        <td>fdb03c8c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce870e7c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fdb06d92-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce871a3e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fdb0759e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce872588-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdb08048-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce822556-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdb0a4a6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce876d54-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdb0b69e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce877d3a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdb0da20-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce879306-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdb0f62c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce87936a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdb0f690-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce879cb6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdb0fce4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce87d9c4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fdb13db2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce87e11c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fdb14424-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce872bdc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fdb1570c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce880c3c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fdb16e0e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce88229e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdb1a446-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce8826d6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fdb1a9c8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce883720-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fdb1b968-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce887668-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fdb1e7da-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce88b722-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdb2459a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce7b75bc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>fdb2b502-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce8920d6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdb2b5c0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce832c44-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdb30494-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce897950-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fdb3102e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce8a730a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fdb3c410-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce8abe32-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fdb408c6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce66ffce-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fdb477ac-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce8ba4a0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdb4d0bc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce8c185e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fdb53250-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce8c1f3e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdb538cc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce8c2bb4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdb546be-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce8c3546-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fdb55000-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce8c6296-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fdb55df2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce90a266-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdb569fa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce90acc0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fdb589a8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce90e456-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fdb5aabe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce912df8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fdb5f5c8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce91745c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fdb6124c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce917a10-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fdb61684-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce918d2a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdb624da-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce91b75a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdb66d3c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce9255ac-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fdb723b2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce925a66-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdb72790-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce9267ea-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdb73758-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce92738e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdb756d4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce92adfe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3</td>
        <td>fdb76912-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce92f2f0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fdb7cace-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce92f8d6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdb7d14a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce930b14-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdb7e716-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce934a3e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fdb82334-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce936848-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fdb83fc2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce937496-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdb85048-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce9378c4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdb8561a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce9390de-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdb88702-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce93da3a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdb8b56a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce93f5d8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdb8eed6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce9450e6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fdb93d14-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce946798-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdb96460-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce946b26-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdb964c4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce949a88-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fdb9999e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce94a60e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdb9b924-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce94da48-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fdb9cae0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce952aac-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fdba36a6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce953ef2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fdba433a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce957a0c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdba7d3c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce957b2e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdba7e68-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce9585ec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fdba8868-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce95a360-7144-11e5-ba71-058fbc01cf0b</td>
        <td>5</td>
        <td>fdbaa140-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce95b530-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdbab77a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce962498-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdbb41a4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce9633fc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdbb527a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce9637ee-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdbb543c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce964d7e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fdbb6ddc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce96a42c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fdbc10ca-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce96b020-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdbc1d9a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce96afbc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdbc1eda-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce96c9a2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdbc31f4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce96ca10-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdbc32bc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce96d8fc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdbc39b0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce96eb6c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdbc3c08-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce96f21a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdbc45ea-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce97179a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdbd6d44-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce9719ca-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdbd7122-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce971dd0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fdbd813a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce972064-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3</td>
        <td>fdbd83d8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce972352-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdbd8d1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce9723b6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdbd8da6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce9724d8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdbd921a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce972974-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdbd9332-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce9734e6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdbda070-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce975e62-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdbdce92-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce976c22-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdbdf26e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce977bb8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdbe0448-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce97aa70-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fdbe461a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce97ab7e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdbe4ae8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce97da68-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fdbe6f3c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce97df36-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdbe78ec-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce97e40e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3</td>
        <td>fdbe80bc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce9801b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdbe9f34-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce980330-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fdbea3c6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce9803ee-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdbea542-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce98020e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdbeb60e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce9816a4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdbed332-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce9862da-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdbf0e88-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce9862da-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdbf139c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce9864a6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdbf1432-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce9875a4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3</td>
        <td>fdbf3868-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce988832-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fdbf4d9e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce988c4c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fdbf5140-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce3fd78c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fdbf6342-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce98b9b0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fdbf66e4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce98696a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdbf9876-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce98eeee-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fdbfbbb2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce995d3e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdc02b92-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce996e28-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fdc05a5e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce99bebe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdc0b418-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce99d732-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3</td>
        <td>fdc0c502-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce99dfac-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdc0d114-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce99e40c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdc0d6fa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce99d37c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdc0d894-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce99e998-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fdc0dd9e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce9a284a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdc117fa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce9a3bb4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdc13104-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce99620c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>fdc184f6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce9ab8f0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fdc1ab52-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce9ac2a0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>3</td>
        <td>fdc1ba5c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce9af144-7144-11e5-ba71-058fbc01cf0b</td>
        <td>4</td>
        <td>fdc1fe40-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
    <tr>
        <td>ce9af95a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>fdc20336-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
    </tr>
</table>



## Practice inner joining your own tables!

**Question 7: How many unique Golden Retrievers who live in North Carolina are there in the Dognition database (you should get 30)?**




```python
%%sql
SELECT u.state, d.breed, COUNT(DISTINCT d.dog_guid)
FROM users u, dogs d
WHERE d.user_guid=u.user_guid 
AND breed="Golden Retriever"
AND state = "NC"
```

    1 rows affected.





<table>
    <tr>
        <th>state</th>
        <th>breed</th>
        <th>COUNT(DISTINCT d.dog_guid)</th>
    </tr>
    <tr>
        <td>NC</td>
        <td>Golden Retriever</td>
        <td>30</td>
    </tr>
</table>



**Question 8: How many unique customers within each membership type provided reviews (there should be 2900 in the membership type with the greatest number of customers, and 15 in the membership type with the fewest number of customers if you do NOT include entries with NULL values in their ratings field)?**


```python

```

**Question 9: For which 3 dog breeds do we have the greatest amount of site_activity data, (as defined by non-NULL values in script_detail_id)(your answers should be "Mixed", "Labrador Retriever", and "Labrador Retriever-Golden Retriever Mix"?**


```python

```

**Practice any other inner joins you would like to try here!**


```python

```
