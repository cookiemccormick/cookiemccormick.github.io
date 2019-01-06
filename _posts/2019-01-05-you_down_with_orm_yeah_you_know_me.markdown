---
layout: post
title:      "You Down with ORM (yeah you know me)."
date:       2019-01-06 01:12:51 +0000
permalink:  you_down_with_orm_yeah_you_know_me
---

Now that I've got your attention, lets talk about ORM (Object-Relational Mapping).  ORM is a technique that lets you query and operate on data in a relational database using object-oriented programming.  ORM associates classes with database tables and instances of those classes with table rows.  ORM is useful in that it cuts repetitious code and allows conventional patterns for constructing complex systems in an organized manner.

![](http://wiki.expertiza.ncsu.edu/images/6/60/3j_ks_pic1.jpg)

**HOW TO MAP A CLASS TO A TABLE**

In this example, we will be using a database about books.  This program will have a `Book` class and each book instance will have an author and a title (also known as the `attr_accessor`s).  the `attr_acccessor`s are created from the columns of the database. 

```
class Book
  attr_accessor :author, :title
  attr_reader :id

  def initialize(author, title, id = nil)
    @id = id
    @author = author
    @title = title
  end
end
```

The `id` attribute has a default value of `nil`.  The `id` attribute is assigned once the object is saved into the database.

**1.  Database Creation **

Classes are mapped to tables inside a database, not the database itself.  The creation of the database is the responsibility of our program.  It is conventional to pluralize the name of the class to create the name of the table.  Our `Book` class is our "books" table.  

The environment folder should look like this:

```
require 'sqlite3'
require_relative '../lib/book'
 
DB = {:conn => SQLite3::Database.new("db/books.db")}
```

We can access the `DB` connection:

```
DB[:conn]
```

**2.  Table Creation**

We can create a table in our `Book` class using a class method.  `<<-` is known as a [here-document](https://en.wikipedia.org/wiki/Here_document) or heredoc - used to form multiline string literals, saving line breaks and other white space.

```
def self.create_table
  sql = <<-SQL
    CREATE TABLE IF NOT EXISTS books (
      id INTEGER PRIMARY KEY,
      author TEXT,
      title TEXT
    )
  SQL

  DB[:conn].execute(sql)
end
```

**3.  Saving Data**

The `#save` method inserts record into our database that has the author and title values of the book instance.  We create a sql `INSERT` statement with bound parameters. Bound parameters use `?` characters as placeholders for values we pass in as arguments.  This is also where the `id` is assigned to the object.  We grab the `id` of the inserted row and assign it to the value of the `id` attribute.  The `id` is also the row where this object can be found.

We must also check to see if the object has already been persisted.  If the object id already exists, we update the record.  Otherwise, we save the new object.  

```
def save
  if self.id
    self.update
  else
    sql = <<-SQL
      INSERT INTO books (author, title)
      VALUES (?, ?)
    SQL
		
    DB[:conn].execute(sql, self.author, self.title)
    @id = DB[:conn].execute("SELECT last_insert_rowid()")[0][0]
  end
  self
end
```

**4.  Updating Records**

The best way to update a record is to update all the attributes at the same time.  

```
def update
  sql = "UPDATE books SET author = ?, title = ? WHERE id = ?"
  DB[:conn].execute(sql, self.author, self.title, self.id)
end
```

Now, you can then create new books and save them to the database.

```
Books.new("J. K. Rowling", "Harry Potter and the Sorcerer's Stone").save
Books.new("Michelle Obama", "Becoming").save
Books.new("J. Kenji Lopez-Alt", "The Food Lab").save
```

| id | author | title |
| -------- | -------- | -------- |
| 1     | J. K. Rowling       | Harry Potter and the Sorcerer's Stone     |
| 2     | Michelle Obama    | Becoming   |
| 3     | J. Kenji Lopez-Alt | The Food Lab |









