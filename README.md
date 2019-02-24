# Security and backup with MySQL <img src="https://pngimg.com/uploads/mysql/mysql_PNG9.png" height="40" align="center">

The SQL scripts are tailored to "**classic models**" sample database.<br>
(http://www.mysqltutorial.org/mysql-sample-database.aspx)



An overview of the tables can be seen in this figure:

<p align="center">
<img src="https://waffleio-direct-uploads-production.s3.amazonaws.com/uploads/5b631124103d580013dcf6a4/125516c66e82c728ace21e0d46cb989b30918aaaa2e6bb46a84ed5a66b0a65633b0ba92e7391a172fd17790456571cfd445e5946faab802ba7be6a74de0a0abfd37f4366846c4ff8e7df46f554f832815d84beb68b49c5e93d24e08a8d4d39190c0ac4.png" width="75%">
</p>
<br/>

----

# Table of Contents
* [Exercise 1](#ex1)
  * [Argumentations](#argumentation)
  * [Queries](#queries)
* [Exercise 2](#ex2)
* [Exercise 3](#ex3)

<br/>

----
<a name="ex1"></a>
## Excercise 1 - User privileges ![Generic badge](https://img.shields.io/badge/SQL-queries-yellow.svg)

Five users need to be created for the following roles:

| Role | Privileges | 
|---|---|
| **Inventory**  | Maintains the two tables `products` and `productlines`.  | 
| **Bookkeeping**  | Makes sure that all orders are paid.  | 
| **Human resources**  | Takes care of employees and their offices.  | 
| **Sales**  | Creates the orders for the customers.  | 
| **IT**  | Maintains this database.  | 

<br/>

<a name="argumentation"></a>
### Granted privileges argumentation

- `inventory` - This user is granted *SELECT, INSERT,* and *UPDATE* permissions to both `products` and `productlines` tables. The reason for that is due to the responsibilities that this role is associated with, viz. to maintain the data about Products and Product Lines.
- `bookKeeping` - User № 2 is restricted to only perform *SELECT* operations to the `orders` table since they only need to monitor the Orders. 
- `hr` - The user representing *Human resources* is allowed to execute *SELECT, INSERT, UPDATE* and *DELETE* operations on the database, so that they can fetch, update or delete the data in the `employees` table. As well as *SELECT* the records for all their Offices.
- `sales` - This user is just given *INSERT* permissions to `orders` and `orderdetails`, since user named `bookKeeping` will review the Orders. Additionally, it can *SELECT* from the `customers` table.
- `it` - Maintaining the database would require full access to the entire database, therefore, any operation is permitted.

<br/>

<a name="queries"></a>
### Queries
> Create users
```sql
CREATE USER 'inventory' IDENTIFIED BY '123456';
GRANT SELECT, INSERT, UPDATE ON classicmodels.products TO 'inventory'@'%';
GRANT SELECT, INSERT, UPDATE ON classicmodels.productlines TO 'inventory'@'%';
```

```sql
CREATE USER 'bookKeeping' IDENTIFIED BY '123456';
GRANT SELECT ON classicmodels.orders TO 'bookKeeping'@'%';
```

```sql
CREATE USER 'hr' IDENTIFIED BY '123456';
GRANT SELECT, INSERT, UPDATE, DELETE ON classicmodels.employees TO 'hr'@'%';
GRANT SELECT ON classicmodels.offices TO 'hr'@'%';
```

```sql
CREATE USER 'sales' IDENTIFIED BY '123456';
GRANT INSERT ON classicmodels.orders TO 'sales'@'%';
GRANT INSERT ON classicmodels.orderdetails TO 'sales'@'%';
GRANT SELECT ON classicmodels.customers TO 'sales'@'%';
```

```sql
CREATE USER 'it' IDENTIFIED BY '123456';
GRANT ALL ON classicmodels.* TO 'it'@'%';
```
<br/>

*❗️You can also add the following line before each user creation to make sure privileges are not compiled together with your previous implementation.* 😉 *(Make sure to change the username after `EXISTS` for the respective user.)*

```sql
DROP USER IF EXISTS 'inventory';
```

<br/>

----

<a name="ex2"></a>
## Excercise 2 - Logging ![Generic badge](https://img.shields.io/badge/log-files-blue.svg)

A segment from the database log file is included next to demonstrate the following elements:

* The users and their privileges being added (ref. Exercise 1)
* Three changes to the database:
  * Insert 2 new Employees 
  * Insert 1 new Product
  * Create 1 new Order
* One attempt to make a change by a user with the wrong privileges

</br>

*❗️The required elements are being highlighted (𝗕)*

<pre>

| 20:59:25 | [root] @ localhost []                   | Connect      | root@localhost on  using Socket                                                                                                          |
| 20:59:25 | root[root] @ localhost []               | Query        | select @@version_comment limit 1                                                                                                         |
| 20:58:42 | bookKeeping[bookKeeping] @ localhost [] | Quit         |                                                                                                                                          |
| 20:58:34 | bookKeeping[bookKeeping] @ localhost [] | Query        | UPDATE orders SET status = 'Paid' WHERE orderNumer = 10426                                                                               |
| 20:58:31 | bookKeeping[bookKeeping] @ localhost [] | Query        | SELECT DATABASE()                                                                                                                        |
| 20:58:31 | bookKeeping[bookKeeping] @ localhost [] | Init DB      | classicmodels                                                                                                                            |
| 20:58:31 | bookKeeping[bookKeeping] @ localhost [] | Query        | show databases                                                                                                                           |
| 20:58:31 | bookKeeping[bookKeeping] @ localhost [] | Query        | show tables                                                                                                                              |
| 20:58:31 | bookKeeping[bookKeeping] @ localhost [] | Field List   | orders                                                                                                                                   |
| 20:56:50 | [bookKeeping] @ localhost []            | Connect      | bookKeeping@localhost on  using Socket                                                                                                   |
| 20:56:50 | bookKeeping[bookKeeping] @ localhost [] | Query        | select @@version_comment limit 1                                                                                                         |
| 20:56:44 | sales[sales] @ localhost []             | Quit         |                                                                                                                                          |
|<b> 20:56:35 | sales[sales] @ localhost []             | Query        | INSERT INTO orders VALUES (10426, '2019-02-23', '2019-03-23', NULL, 'In Process', NULL, 141)                                             |
| 20:56:05 | sales[sales] @ localhost []             | Query        | INSERT INTO orders VALUES (10426, '2019-02-23', '2019-03-23', 'NULL', 'In Process', 'NULL', 141)  </b>                                       |
| 20:56:03 | sales[sales] @ localhost []             | Query        | SELECT DATABASE()                                                                                                                        |
| 20:56:03 | sales[sales] @ localhost []             | Init DB      | classicmodels                                                                                                                            |
| 20:56:03 | sales[sales] @ localhost []             | Query        | show databases                                                                                                                           |
| 20:56:03 | sales[sales] @ localhost []             | Query        | show tables                                                                                                                              |
| 20:56:03 | sales[sales] @ localhost []             | Field List   | customers                                                                                                                                |
| 20:56:03 | sales[sales] @ localhost []             | Field List   | orderdetails                                                                                                                             |
| 20:56:03 | sales[sales] @ localhost []             | Field List   | orders                                                                                                                                   |
| 20:56:00 | [sales] @ localhost []                  | Connect      | sales@localhost on  using Socket                                                                                                         |
| 20:56:00 | sales[sales] @ localhost []             | Query        | select @@version_comment limit 1                                                                                                         |
| 20:55:55 | [sales] @ localhost []                  | Connect      | sales@localhost on  using Socket                                                                                                         |
<b>| 20:55:55 | sales[sales] @ localhost []             | Connect      | Access denied for user 'sales'@'localhost' (using password: YES)**                                                                       |
| 20:55:48 | bookKeeping[bookKeeping] @ localhost [] | Quit         |                                                                                                                                          |
| 20:53:50 | bookKeeping[bookKeeping] @ localhost [] | Query        | SELECT * FROM orders  </b>                                                                                                                   |
| 20:53:41 | bookKeeping[bookKeeping] @ localhost [] | Query        | SELECT DATABASE()                                                                                                                        |
| 20:53:41 | bookKeeping[bookKeeping] @ localhost [] | Init DB      | classicmodels                                                                                                                            |
| 20:53:41 | bookKeeping[bookKeeping] @ localhost [] | Query        | show databases                                                                                                                           |
| 20:53:41 | bookKeeping[bookKeeping] @ localhost [] | Query        | show tables                                                                                                                              |
| 20:53:41 | bookKeeping[bookKeeping] @ localhost [] | Field List   | orders                                                                                                                                   |
| 20:53:39 | [bookKeeping] @ localhost []            | Connect      | bookKeeping@localhost on  using Socket                                                                                                   |
| 20:53:39 | bookKeeping[bookKeeping] @ localhost [] | Query        | select @@version_comment limit 1                                                                                                         |
| 20:53:28 | sales[sales] @ localhost []             | Quit         |                                                                                                                                          |
| 20:52:53 | sales[sales] @ localhost []             | Query        | SELECT DATABASE()                                                                                                                        |
| 20:52:53 | sales[sales] @ localhost []             | Init DB      | classicmodels                                                                                                                            |
| 20:52:53 | sales[sales] @ localhost []             | Query        | show databases                                                                                                                           |
| 20:52:53 | sales[sales] @ localhost []             | Query        | show tables                                                                                                                              |
| 20:52:53 | sales[sales] @ localhost []             | Field List   | customers                                                                                                                                |
| 20:52:53 | sales[sales] @ localhost []             | Field List   | orderdetails                                                                                                                             |
| 20:52:53 | sales[sales] @ localhost []             | Field List   | orders                                                                                                                                   |
| 20:52:42 | [sales] @ localhost []                  | Connect      | sales@localhost on  using Socket                                                                                                         |
| 20:52:42 | sales[sales] @ localhost []             | Query        | select @@version_comment limit 1                                                                                                         |
| 20:52:33 | inventory[inventory] @ localhost []     | Quit         |                                                                                                                                          |
| 20:52:25 | inventory[inventory] @ localhost []     | Query        | SELECT * FROM products                                                                                                                   |
|<b> 20:52:22 | inventory[inventory] @ localhost []     | Query        | INSERT INTO products VALUES ('S72_4323', 'UAE AIRLINES: BF-232', 'Planes', '1:700', 'Amazing Vendor', 'Best airliner', 6, 10.23, 15.00)</b>  |
| 20:48:57 | inventory[inventory] @ localhost []     | Query        | SELECT * FROM products                                                                                                                   |
| 20:48:52 | inventory[inventory] @ localhost []     | Query        | SELECT DATABASE()                                                                                                                        |
| 20:48:52 | inventory[inventory] @ localhost []     | Init DB      | classicmodels                                                                                                                            |
| 20:48:52 | inventory[inventory] @ localhost []     | Query        | show databases                                                                                                                           |
| 20:48:52 | inventory[inventory] @ localhost []     | Query        | show tables                                                                                                                              |
| 20:48:52 | inventory[inventory] @ localhost []     | Field List   | productlines                                                                                                                             |
| 20:48:52 | inventory[inventory] @ localhost []     | Field List   | products                                                                                                                                 |
| 20:48:14 | [inventory] @ localhost []              | Connect      | inventory@localhost on  using Socket                                                                                                     |
| 20:48:14 | inventory[inventory] @ localhost []     | Query        | select @@version_comment limit 1                                                                                                         |
| 20:48:01 | hr[hr] @ localhost []                   | Quit         |                                                                                                                                          |
|<b> 20:47:46 | hr[hr] @ localhost []                   | Query        | INSERT INTO employees VALUES (1705, 'Bing', 'Bang', 'x105', 'thisisbigbang@classicmodelcars.com', 3, 1143, 'Sales Rep')                  |
| 20:47:45 | hr[hr] @ localhost []                   | Query        | INSERT INTO employees VALUES (1703, 'Bing', 'Bent', 'x100', 'thisisbigbent@classicmodelcars.com', 3, 1143, 'Sales Rep') </b>                 |
| 20:47:30 | hr[hr] @ localhost []                   | Query        | SELECT DATABASE()                                                                                                                        |
| 20:47:30 | hr[hr] @ localhost []                   | Init DB      | classicmodels                                                                                                                            |
| 20:47:30 | hr[hr] @ localhost []                   | Query        | show databases                                                                                                                           |
| 20:47:30 | hr[hr] @ localhost []                   | Query        | show tables                                                                                                                              |
| 20:47:30 | hr[hr] @ localhost []                   | Field List   | employees                                                                                                                                |
| 20:47:30 | hr[hr] @ localhost []                   | Field List   | offices                                                                                                                                  |
| 20:46:20 | [hr] @ localhost []                     | Connect      | hr@localhost on  using Socket                                                                                                            |
| 20:46:20 | hr[hr] @ localhost []                   | Query        | select @@version_comment limit 1                                                                                                         |
| 20:46:10 | root[root] @ localhost []               | Quit         |                                                                                                                                          |
| 20:34:39 | root[root] @ localhost []               | Query        | select * from employees                                                                                                                  |
| 20:34:36 | root[root] @ localhost []               | Query        | SELECT DATABASE()                                                                                                                        |
| 20:34:36 | root[root] @ localhost []               | Init DB      | classicmodels                                                                                                                            |
| 20:34:36 | root[root] @ localhost []               | Query        | show databases                                                                                                                           |
| 20:34:36 | root[root] @ localhost []               | Query        | show tables                                                                                                                              |
| 20:34:36 | root[root] @ localhost []               | Field List   | customers                                                                                                                                |
| 20:34:36 | root[root] @ localhost []               | Field List   | employees                                                                                                                                |
| 20:34:36 | root[root] @ localhost []               | Field List   | offices                                                                                                                                  |
| 20:34:36 | root[root] @ localhost []               | Field List   | orderdetails                                                                                                                             |
| 20:34:36 | root[root] @ localhost []               | Field List   | orders                                                                                                                                   |
| 20:34:36 | root[root] @ localhost []               | Field List   | payments                                                                                                                                 |
| 20:34:36 | root[root] @ localhost []               | Field List   | productlines                                                                                                                             |
| 20:34:36 | root[root] @ localhost []               | Field List   | products                                                                                                                                 |
| 20:34:10 | [root] @ localhost []                   | Connect      | root@localhost on  using Socket                                                                                                          |
| 20:33:34 | root[root] @ localhost []               | Quit         |                                                                                                                                          |
| 20:25:31 | root[root] @ localhost []               | Query        | GRANT ALL ON classicmodels.* TO 'it'@'%'                                                                                                 |
| 20:25:29 | root[root] @ localhost []               | Query        | DROP USER IF EXISTS 'inventory'                                                                                                          |
|<b> 20:25:29 | root[root] @ localhost []               | Query        | CREATE USER 'inventory'@'%' IDENTIFIED BY <secret>                                                                                               |
| 20:25:29 | root[root] @ localhost []               | Query        | GRANT SELECT, INSERT, UPDATE ON classicmodels.products TO 'inventory'@'%'                                                                |
| 20:25:29 | root[root] @ localhost []               | Query        | GRANT SELECT, INSERT, UPDATE ON classicmodels.productlines TO 'inventory'@'%' </b>                                                           |
| 20:25:29 | root[root] @ localhost []               | Query        | DROP USER IF EXISTS 'bookKeeping'                                                                                                        |
|<b> 20:25:29 | root[root] @ localhost []               | Query        | CREATE USER 'bookKeeping'@'%' IDENTIFIED BY         <secret>                                                                                     |
| 20:25:29 | root[root] @ localhost []               | Query        | GRANT SELECT ON classicmodels.orders TO 'bookKeeping'@'%' </b>                                                                               |
| 20:25:29 | root[root] @ localhost []               | Query        | DROP USER IF EXISTS 'hr'                                                                                                                 |
|<b> 20:25:29 | root[root] @ localhost []               | Query        | CREATE USER 'hr'@'%' IDENTIFIED BY <secret>                                                                                                      |
| 20:25:29 | root[root] @ localhost []               | Query        | GRANT SELECT, INSERT, UPDATE, DELETE ON classicmodels.employees TO 'hr'@'%'                                                              |
| 20:25:29 | root[root] @ localhost []               | Query        | GRANT SELECT ON classicmodels.offices TO 'hr'@'%' </b>                                                                                       |
| 20:25:29 | root[root] @ localhost []               | Query        | DROP USER IF EXISTS 'sales'                                                                                                              |
|<b> 20:25:29 | root[root] @ localhost []               | Query        | CREATE USER 'sales'@'%' IDENTIFIED BY <secret>                                                                                                   |
| 20:25:29 | root[root] @ localhost []               | Query        | GRANT INSERT ON classicmodels.orders TO 'sales'@'%'                                                                                      |
| 20:25:29 | root[root] @ localhost []               | Query        | GRANT INSERT ON classicmodels.orderdetails TO 'sales'@'%'                                                                                |
| 20:25:29 | root[root] @ localhost []               | Query        | GRANT SELECT ON classicmodels.customers TO 'sales'@'%'</b>                                                                                   |
| 20:25:29 | root[root] @ localhost []               | Query        | DROP USER IF EXISTS 'it'                                                                                                                 |
|<b> 20:25:29 | root[root] @ localhost []               | Query        | CREATE USER 'it'@'%' IDENTIFIED BY <secret>                                                                                                      |
| 20:25:28 | root[root] @ localhost []               | Query        | GRANT ALL ON classicmodels.* TO 'it'@'%'</b>
</pre>

<br/>

----

<a name="ex3"></a>
## Excercise 3 - Backup and recovery ![Generic badge](https://img.shields.io/badge/backup-recovery-important.svg)

The backup technique used for this database is a combination of `full local online logical backup`.

You can install the database from the [backup.sql](./backup.sql) file by the following command:

```bash
mysql -u root -p < backup.sql
```
</br>

___
> #### Assignment made by:   
`David Alves 👨🏻‍💻 ` :octocat: [Github](https://github.com/davi7725) <br />
`Elitsa Marinovska 👩🏻‍💻 ` :octocat: [Github](https://github.com/elit0451) <br />
> Attending "Databses for Developers" course of Software Development bachelor's degree
