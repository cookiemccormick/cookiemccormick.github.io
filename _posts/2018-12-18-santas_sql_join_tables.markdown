---
layout: post
title:      "Santa's SQL Join Tables"
date:       2018-12-18 21:10:08 -0500
permalink:  santas_sql_join_tables
---


Happy Holidays everyone!  I thought it would be fun to explain SQL Join Tables in a jolly, merry way.  This year, Santa is putting down his parchment list and checking it only once.  He's asked one of his elves to create a SQL (structured query language for managing data) database to keep track of the naughty and nice kids and their wishes.  Santa will be able to see which kids should get presents, which presents they wished for and and also be able to see the most popular toys.

![](https://img.memecdn.com/buddy-the-elf_o_927460.jpg)

**Creating the database:**

```
sqlite3 christmas_database.db
```

**Creating the tables:**


Santa's database will have 3 tables:

`kids` Table:  Consists of the kid's names, behavior - whether they are naughty or nice, age and address.  `is_nice` will be a boolean representing naughty as false and nice as true.

```
CREATE TABLE kids (
  id INTEGER PRIMARY KEY,
  name TEXT NOT NULL,
  is_nice INTEGER NOT NULL,
  age INTEGER NOT NULL,
  address TEXT NOT NULL
);
```

`wishes` Table:  Consists of the kid's wishes.

```
CREATE TABLE wishes (
  id INTEGER PRIMARY KEY,
  name TEXT NOT NULL
);
```

`kids_wishes` Join Table: Here we are creating a many-to-many association between kids and wishes.  Kids have many wishes and the wishes are wanted by many kids.  This is our join table and it will have two columns to relate to the `kids` table and `wishes` table through their `kid_id` and `wish_id`.  Each row represents one kid/wish relationship.

```
CREATE TABLE kids_wishes (
  id INTEGER PRIMARY KEY,
  kid_id INTEGER NOT NULL,
  wish_id INTEGER NOT NULL
);
```

**Insert Data:**

```
INSERT INTO kids (name, is_nice, age, address) VALUES ("Michael", 1, 7, "37 Wrestle Street North, St. Pete, FL 33703");
INSERT INTO kids (name, is_nice, age, address) VALUES ("Lisa", 1, 6, "777 Brown Street North, Aiea, HI 96701");
INSERT INTO kids (name, is_nice, age, address) VALUES ("Cara", 1, 5, "1111 Cargo Ave, Pittsburgh, PA 15106");
INSERT INTO kids (name, is_nice, age, address) VALUES ("Zac", 0, 6, "743 Scotland Street N, Dunedin, FL 33755");
INSERT INTO kids (name, is_nice, age, address) VALUES ("Abe", 0, 7, "Charles Street, Dunedin, FL 33755");

INSERT INTO wishes (name) VALUES ("bicycle");
INSERT INTO wishes (name) VALUES ("doll");
INSERT INTO wishes (name) VALUES ("water gun");
INSERT INTO wishes (name) VALUES ("computer");
INSERT INTO wishes (name) VALUES ("coloring book");

INSERT INTO kids_wishes (kid_id, wish_id) VALUES (1, 3);
INSERT INTO kids_wishes (kid_id, wish_id) VALUES (2, 5);
INSERT INTO kids_wishes (kid_id, wish_id) VALUES (4, 1);
INSERT INTO kids_wishes (kid_id, wish_id) VALUES (3, 5);
INSERT INTO kids_wishes (kid_id, wish_id) VALUES (5, 5);
INSERT INTO kids_wishes (kid_id, wish_id) VALUES (1, 4);
INSERT INTO kids_wishes (kid_id, wish_id) VALUES (2, 2);
```

Here's a look at the tables:

`kids` Table

| id | name | is_nice | age | address |
| -------- | -------- | -------- |
| 1 | Michael | 1 | 7 | 37 Wrestle Street North, St. Pete, FL 33703 |
| 2 | Lisa | 1  | 6 | 777 Brown Street North, Aiea, HI 96701 |
| 3 | Cara | 1 | 5 | 1111 Cargo Ave, Pittsburgh, PA 15106 |
| 4 | Zac | 0 | 6 | 743 Scotland Street N, Dunedin, FL 33755 |
| 5 | Abe | 0 | 7 | 6027 Charles Street, Dunedin, FL 33755 |

`wishes` Table

| id | name |
| -------- | -------- |
| 1 | bicycle |
| 2 | doll |
| 3 | water gun |
| 4 | computer |
| 5 | coloring book |

`kids_wishes` Table

| id | kid_id | wish_id |
| -------- | -------- | -------- |
| 1 | 1 | 3 |
| 2 | 2 | 5 |
| 3 | 4 | 1 |
| 4 | 3 | 5 |
| 5 | 5 | 5 |
| 6 | 1 | 4 |
| 7 | 2 | 2 |

**Queries**

Santa is now able to conduct queries based on this data.

What are all the wishes for Michael?

```
SELECT wishes.name 
FROM kids_wishes
INNER JOIN kids ON kids_wishes.kid_id = kids.id
INNER JOIN wishes ON kids_wishes.wish_id = wishes.id
WHERE kids.name = "Michael";
```

| name |
| -------- |
| water gun |
| computer |

Which kids are not getting a present because they are naughty?

```
SELECT name FROM kids WHERE is_nice = 0;
```

| name |
| -------- |
| Zac |
| Abe |

What are the names of all the kids who want a coloring book?

```
SELECT kids.name
FROM kids_wishes
INNER JOIN kids ON kids_wishes.kid_id = kids.id
INNER JOIN wishes ON kids_wishes.wish_id = wishes.id
WHERE wishes.name = "coloring book";
```

| name |
| -------- |
| Lisa |
| Cara |
| Abe |

What is the name for the most popular toy?

```
SELECT wishes.name AS popular_toy
FROM wishes
INNER JOIN kids_wishes ON wishes.id = kids_wishes.wish_id
GROUP BY wishes.name
ORDER BY COUNT (wishes.name) DESC
LIMIT 1;
```

| popular_toy |
| -------- |
| coloring book |














