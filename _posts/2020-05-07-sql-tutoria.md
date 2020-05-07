---
title: SQL Tutorial - For Co-workers
updated: 2020-05-07 10:20
---

The following is a copy of an introduction to SQL that I wrote for colleagues who are interested in extracting their own raw data for further analysis, instead of relying on a much slower web API.

### SQL Tutorial - Introduction

Web Office is built from a collection of over 500 SQL tables, each with their own unique identifiers which join together to identify an item. An item's attribute, whether that result is its TPR price on May 4th, 2015, or how many were counted into shrink for Calgary Classic on November 20th, 2017, for example, is a result of two or more tables joined with unique identifiers known as the primary and foreign key.

Lets use the following item as an example:

| UPC | Brand | Description |
|--------|:------------:|:-----------:|
| 851107003056 | Brew Dr | Spiced Apple Kombucha |

The item's UPC is: 851107003056, but ECRS's SQL table doesn't care about that. The item is referred to in tables as a combination of the following two values:

| INV_PK | INV_CPK | 
|--------|:------------:|
| 45214 | 1000 |

**Fun fact:** This is how the Web Office framework generates its URLs:
 catapultweboffice.com/weboffice/#inventory:pk=45214%25c7C1000
	
The header names differ from table to table, but the value does not. As far as SQL is concerned, this item's "internal UPC" or unique identifier is 45214 AND 1000. Additionally, the header is not always descriptive, so it's up to the individual to figure out which is which, should documentation not be available.

Lets pull some item information. Run (Windows + R): dbisql -c dsn=prototype 
Enter the following and click run:

```sql
SELECT * FROM v_InventoryMaster
WHERE INV_PK  = 45214
AND INV_CPK = 1000
```

The result should be multiple rows containing very similar values. You have just pulled all of the item information available in the table titled v_InventoryMaster for each store. So what is the difference between rows? Well you may notice that the column labeled inv_lastsold has very different values, hence the need to display the item multiple times. The column labeled sib_baseprice is also different for 3 rows, as another example. 

Before moving on, lets examine the SQL statement:

All of your SQL statements will begin with  SELECT  , followed by your header selections. The asterisk tells SQL that you would like to pull all available headers in a table, however, most of our following statements will be more specific.  SELECT * FROM (Table Name)  is useful for determining the header names in a specific table, if you're unsure what kind of values it contains. 

 FROM v_InventoryMaster  is the table we're pulling information from. This particular table contains plenty of information on a specific item and is very useful to use in JOIN statements, which we'll examine later. 

 WHERE INV_PK = 45214 
 AND INV_CPK = 1000 

All conditional statements where a specific value is specified starts with WHERE. Any other conditional statements are followed by AND.
There is no difference between the above statement and the following:

 WHERE INV_CPK = 1000 
 AND INV_PK = 45214 

Lets select only the information we require. For this example I want UPC, Name, Base Price, Last Sold, and Store.
Now, I said that UPC doesn't matter to SQL, and it doesn't. But it matters to me so I want to pull that information. I can search by UPC, but if I know the primary and foreign key, it's best to use those because no other table contains the UPC, which is useless to me if I want to join tables in the future.

```sql 
SELECT inv_scancode, inv_name, sib_baseprice, inv_lastsold, sto_name
FROM v_InventoryMaster
WHERE INV_PK  = 45214
AND INV_CPK = 1000#
```
The result is the following table:

| inv_scancode | inv_name | sib_baseprice | inv_lastsold | sto_name |
|--------|:------------:|:-----------:|:-----------:|:-----------:|
| 851107003056 | BRW DRKMB KOMBUCHA SPICED APPL	| 0.00	| NULL	|Planet Organic Market |
|851107003056 | 	BRW DRKMB KOMBUCHA SPICED APPL|	4.79	|NULL	|ZZ-Burlington ON|
|851107003056  | 	BRW DRKMB KOMBUCHA SPICED APPL	| 4.79 | 2019-09-24 15:18:34 | Port Credit|
|851107003056|	BRW DRKMB KOMBUCHA SPICED APPL|	4.79	|2019-03-02 17:20:35.222	|Victoria|
|	851107003056	|BRW DRKMB KOMBUCHA SPICED APPL	|4.99	|2019-09-03 17:29:47|	Calgary Royal Oak|
|	851107003056	|BRW DRKMB KOMBUCHA SPICED APPL|	4.99	|2019-09-14 16:29:30|	Calgary Shepard|
|	851107003056	|BRW DRKMB KOMBUCHA SPICED APPL|	4.99	|2019-08-06 16:03:47	|Calgary Classic|
|	851107003056|	BRW DRKMB KOMBUCHA SPICED APPL	|4.99	|2019-08-09 19:15:25	|Calgary Shaganappi|
|	851107003056	|BRW DRKMB KOMBUCHA SPICED APPL|	4.99	|2019-07-15 15:40:16	|Calgary Britannia|
|	851107003056|	BRW DRKMB KOMBUCHA SPICED APPL	|4.99	|2019-09-09 12:33:05	|Edmonton Classic|
|	851107003056	|BRW DRKMB KOMBUCHA SPICED APPL|	0.00	|NULL	|Z-Edmonton Jasper|
|	851107003056|	BRW DRKMB KOMBUCHA SPICED APPL	|4.99	|2019-08-22 19:21:59	|Edmonton Jasper NEW|
|	851107003056|	BRW DRKMB KOMBUCHA SPICED APPL	|4.99	|2019-09-27 18:47:27	|Edmonton Ellerslie|
|	851107003056	|BRW DRKMB KOMBUCHA SPICED APPL	|4.99	|2019-09-22 17:32:53	|Center in the Park|

That's a lot of rows. Lets say we wanted to get last sold information for just the Calgary stores, on one row. We can do that with JOIN commands. 

First, lets establish how v_InventoryMaster refers to individual stores. The column titled sto_name is nice and descriptive, but you'll find that dealing with strings such as "Calgary Royal Oak" can lead to potential errors in your code. Some tables have a value such as Center in the Park           with ten leading spaces. You can't see them, but they're there. If you don't include them, your code doesn't run.

Lets investigate the following columns:

```sql 
SELECT inv_sto_fk, inv_sto_name 
FROM v_InventoryMaster 
WHERE INV_PK = 45214
AND INV_CPK = 1000
```    

Results in: 

| inv_sto_fk | inv_sto_name | 
|--------|:------------:|
|1	|Planet Organic Market|
|	5 	|ZZ-Burlington ON|
|	6	|Port Credit|
|	9	|Victoria|
|	3	|Calgary Shepard|
|	10 |	Calgary Classic|
|	11	|Calgary Shaganappi|
|	14	|Calgary Royal Oak|
|	1029|	Calgary Britannia|
|	12	|Edmonton Classic|
|	13	|Z-Edmonton Jasper|
|	16	|Edmonton Jasper New|
|	22	|Edmonton Ellerslie|
|	27 	|Center in the Park|

Looks like we found the primary key for each store. Much like the primary key for an item, many different values have their own unique primary and foreign key which allows us to join them across individual tables. In fact, if you run the following:

```sql
SELECT * from Stores
```

You'll find that the exact same values are paired with the same store names. Note that the header names are different (STO_PK and STO_Name).
The column titled STO_CPK is the foreign key, but since they all share the same value (1000), we don't need to include that in future SELECT statements.
Now that I can refer to a store by a number instead of a name, lets write our SELECT statement more effectively:

```sql
SELECT DISTINCT t1.inv_ScanCode AS UPC, t2.inv_lastsold AS Shaganappi, t3.inv_lastsold AS Classic, t4.inv_lastsold AS Shepard, t5.inv_lastsold AS  
Royal_Oak, t6.inv_lastsold AS Britannia
FROM v_InventoryMaster as t1
	
LEFT OUTER JOIN v_InventoryMaster as t2
ON t1.INV_PK = t2.INV_PK AND t1.INV_CPK = t2.INV_CPK AND t2.INV_STO_FK = 11  
	
LEFT OUTER JOIN v_InventoryMaster as t3
ON t1.INV_PK = t3.INV_PK AND t1.INV_CPK = t3.INV_CPK AND t3.INV_STO_FK = 10 
	
LEFT OUTER JOIN v_InventoryMaster as t4
ON t1.INV_PK = t4.INV_PK AND t1.INV_CPK = t4.INV_CPK AND t4.INV_STO_FK = 3
	
LEFT OUTER JOIN v_InventoryMaster as t5
ON t1.INV_PK = t5.INV_PK AND t1.INV_CPK = t5.INV_CPK AND t5.INV_STO_FK = 14
	
LEFT OUTER JOIN v_InventoryMaster as t6
ON t1.INV_PK = t6.INV_PK AND t1.INV_CPK = t6.INV_CPK AND t6.INV_STO_FK = 1029
	
WHERE t1.INV_PK = 45214
AND t1.INV_CPK = 1000#
```

Lots going on here. lets start with  t1.INV_ScanCode AS UPC  (and t2 - t5, for that matter)
Because we want to pull inv_scancode multiple times on the same row, we need to specify different versions of the same table. That way, SQL knows that we want conditional statements for each version of inv_lastsold. Adding  AS UPC  is just a way for us to rename our headers, and make the result a little more readable. 

LEFT OUTER JOIN   - Think of this as a VLOOKUP in excel. We're joining the matching values from the left (in this case, the UPC)

 ON t1.INV_PK = t2.INV_PK AND t1.INV_CPK = INV_CPK AND t2.INV_STO_FK = 11 

If the left outer join is the =VLOOKUP( ), this is what's contained in brackets. Here we're telling SQL that we want the primary key (45214) and foreign key (1000) to match. Makes sense, because we want the same item to be referred to each time.

 AND t2.INV_STO_FK = 11 

11 referrers to Shaganappi (10 - Classic, 3 - Shepard, 14 - Royal Oak, 1029 - Britannia (Why 1029, I'll never know)).

 WHERE ti.INV_PK = 45214 

In all of our outer joins, we refer to t(2-5).INV_PK and INV_CPK to being equal to t1.INV_PK (and CPK), but what do they equal? This is where we specify that.

The result is the following table:

| UPC	| Shaganappi	| Classic	| Shepard	| Royal Oak	| Britannia |
|--------|:------------:|:------------:|:------------:|:------------:|:------------:|
|851107003056| 2019-08-09 19:15:25.840	|2019-08-06 16:03:47.407	|2019-09-14 16:29:30.801	|2019-09-03 17:29:47.263	|2019-07-15 15:40:16.955
