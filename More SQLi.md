PicoCTF 2023 200 point question (4000 solves on picoGym)

We start with a normal login page with two username and password fields. When we enter in some random credentials, we get a screen with the credentials we entered, and the actual SQL query that is being done on the server side:
```SQL
username: admin
password: admin
SQL query: SELECT id FROM users WHERE password = 'admin' AND username = 'admin'
```
We can bypass it using a very basic SQLi payload: 

admin: `admin`
password: `' OR 1=1 --`

The quote at the start escapes the string that the query is searching for in the WHERE clause. The `OR 1=1` is for making the WHERE clause return true for any password, which in this case was an empty string. The -- is the most common way to comment out the rest of a SQL query. 
 
Once we log in, there is a search page that searched for the city in the list, and returns a table with 3 columns. Now, we want to know what database software we are actually using. There are a few ways to do this. https://security.stackexchange.com/a/94420

However, how are we going to actually see the data that is sent back from the server? We can use the UNION SQL command. But when you first try to use it, you might be confused because whatever you try might not seem like it works. That's probably because you need to have the same number of columns that you return in both SELECT statements, i.e.

`SELECT 1,2,3 FROM table UNION SELECT 1 FROM other_table` would NOT work, but 
`SELECT 1,2,3 FROM table UNION SELECT 1, NULL, NULL FROM other_table` would work 

Now, with this information, we can find the version of the SQL server with a little trial and error with this query in the search bar:

`' UNION SELECT sqlite_version(), NULL, NULL --` and we find that we are using SQLite

The rest is pretty standard, with some union selects with the master database table which contains information about the tables and columns in a database:

`' UNION SELECT name, sql, NULL from sqlite_master --`

We find the `more_table` table with a flag column. we use another UNION select to get the contents, giving us the flag.