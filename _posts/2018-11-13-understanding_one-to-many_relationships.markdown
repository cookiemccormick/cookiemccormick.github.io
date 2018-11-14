---
layout: post
title:      "Understanding One-To-Many Relationships"
date:       2018-11-14 00:31:37 +0000
permalink:  understanding_one-to-many_relationships
---

I'm currently learning Object Oriented Programming in Ruby.  Object Oriented Programming is used to model real life associations and relationships through classes. 

The most common pattern when modeling object relationships is the one-to-many association. Examples of one-to-many relationships in real life are:  

* an artist has many songs
* a chef has many recipes
* an author has many books  

When we have an artist, chef or author object, we want to be able to get a collection of their work.  This collection (the "has many" part of the relationship) can be returned as an array of objects.

Here are some examples of code accessing the array of objects:

```
beck.songs
# ["the new pollution", "devil's haircut", "loser"]

david_chang.recipes
# ["chawan mushi", "korean sushi rolls", "sparkling white kimchi"]

dr_seuss.books
# ["green eggs and ham", "hop on pop", "how the grinch stole christmas"]
```


**MODELING THE RELATIONSHIP BETWEEN AN ARTIST AND THEIR SONGS**

We will define an `Artist` class and a `Song` class.  We want to retrieve an artist's songs and have the songs know about their artist.

**Step 1:**
We want to initialize the `Artist` class with a name argument.  We also want to create an empty array for the songs for this artist.  We also use `attr_accessor` for both `name` and `songs` so that they can be accessed by code that uses the `Artist` class.

```
class Artist
  attr_accessor :name, :songs

  def initialize(name)
    @name = name
    @songs = []
  end
end
```

**Step 2:**
We want to initialize our `Song` instances with an name argument.  Upon initialization, we push new instances into the class variable `@@all` - `self` here means the new instance of song.  `@@all` is effectively a global variable that will keep track of all `Song` instances.  We expose this global variable using a class method called `all`.  In the definition of the `all` method, `self` refers to the `Song` class.

```
class Song
  attr_accessor :name, :artist

  @@all = []

  def initialize(name)
    @name = name
    @@all << self
  end

  def self.all
    @@all
  end
end
```

**Step 3:**
Now we want to add songs to an artist.  To create the one-to-many relationship we define a method named `#add_song`.  The method takes a `Song` argument and adds it to the artist's array of songs.  Then, it associates that song with the artist by assigning the song's artist to `self`.  We also create a helper method called `#add_song_by_name` that simplifies adding a new song using a string value.  This method takes a `String` argument and uses our initializer and our `#add_song` method to create a new song.  By using the exisiting `#add_song` method, we're able to define this new method without repeating the same code.

```
class Artist
  attr_accessor :name, :songs

  def initialize(name)
    @name = name
    @songs = []
  end

  def add_song(song)
    @songs << song
    song.artist = self
  end

  def add_song_by_name(name)
    add_song(Song.new(name))
  end
end
```

**Step 4:**
Now we can put it all together and use our `Artist` and `Song` classes to demonstrate their one-to-many relationship.

```
beck = Artist.new("Beck")
beck.add_song(Song.new("The New Pollution"))
beck.add_song(Song.new("Devil's Haircut"))
beck.add_song(Song.new("Loser"))

beck.songs.count
# 3

beck.songs.first.name
# "The New Pollution"

beck.songs.last.name
# "Loser"

beck.songs.first.artist.name
# "Beck"

beck.songs.first.artist.songs.last.name
# "Loser"
```

**Summary**

By using Ruby classes we're able to model a real life one-to-many relationship between two objects - `Artist` and `Song`.  An artist has many songs and a song has one artist.  We implemented both the "has many" and the "belongs to" sides of the relationship.  When a song is associated with an artist, the inverse relationship is automatically setup, so we can access the relationship from either side.

