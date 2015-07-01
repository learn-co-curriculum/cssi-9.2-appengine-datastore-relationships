---
tags: appengine, datastore, database, relationships, python
level: 3
languages: python
---

# App Engine Datastore Relationships

## Objectives

+ Understand why we would want to represent relationships between data in a database

## Motivation / Why Should You Care?

In many web applications, we don’t just care about the data itself - we care about the relationships between data. A giant list of all the posts on facebook wouldn’t be very interesting - we want to associate posts to users, so that you see posts from your friends, not from random people everywhere.

## Lesson

Here we will talk about different ways we could potentially show the relationships between data.

### Relating objects by not relating objects

We know what the first way would look like already - just more properties on kindergartener!

```python
#The bad way to do things
class Kindergartener(ndb.Model):
	#info
	name = ndb.StringProperty()
	birthday = ndb.DateProperty()
	fav_food = ndb.StringProperty()

	#lunchbox contents
	drink = ndb.StringProperty()
	food = ndb.StringProperty(repeated=True)
	logo = ndb.BlobProperty()
	insulated = ndb.BooleanProperty()
```
This will work, but it’s confusing - Kindergarteners have logos? What is a insulated kindergartener? That doesn’t make sense! If we gave this code to someone else, they would have to ask us questions about it.

We don’t want to answer questions! The code should speak for itself.

### Relating objects with `StructuredProperty`

The second way is more interesting, and it uses a new `ndb` property - `StructuredProperty`. `StructuredProperty` lets us use a model as a property.

```python
class Lunchbox(ndb.Model):
	foods = ndb.StringProperty(repeated=True)
	drink = ndb.StringProperty()
	insulated = ndb.BooleanProperty()
	logo = ndb.BlobProperty()

class Kindergartener(ndb.Model):
	name = ndb.StringProperty()
	birthday = ndb.DateProperty()
      fav_food = ndb.StringProperty()
	lunchbox = ndb.StructuredProperty(Lunchbox)

# using the models to create a kindergartener...robot!
robot_kindergartener = Kindergartener(
    name="Friendly",
    birthday = datetime.date.today(),
    fav_food = "Human Brains",

    lunchbox = Lunchbox(
         drink="motor oil",
         foods = ["Brains","Flesh"],
         logo = "string of logo image data",
         insulated = False)
)

key = robot_kindergartener.put()
```

Voila! The code makes sense!

Kindergarteners each have a Lunchbox; not food, drinks, logos, and insulated properties. We also get some other benefits from this separation: if we ever wanted to have Lunchboxes independent of Kindergarteners - say there were high schoolers who also brought lunch - we can reuse our code. Nice!

### Objects related by reference to their keys

The third way is even more interesting! It uses another new ndb property - KeyProperty.

```python
class Kindergartener(ndb.Model):
	name = ndb.StringProperty()
	birthday = ndb.DateProperty()
	fav_food = ndb.StringProperty()

class Lunchbox(ndb.Model):
	foods = ndb.StringProperty(repeated=True)
	drink = ndb.StringProperty()
	insulated = ndb.BooleanProperty()
	logo = ndb.BlobProperty()

       # here's the new property to keep track of the relationship
        owner_key = ndb.KeyProperty(Kindergartener)


robot_kindergartener = Kindergartener(
    name="Friendly",
    birthday = datetime.date.today(),
    fav_food = "Human Brains",
)

kinder_key = robot_kindergartener.put()

robo_lunch = Lunchbox(
	foods = ["Brains","Spare Parts"],
	drink = "motor oil",
	insulated = False,
	logo = "a string with the logo image data",
	owner_key = kinder_key
	)

lunch_key = robo_lunch.put()

# Get all the lunchboxes
lunchboxes = Lunchbox.query().fetch()

# Get the kindergartener associated with the first lunchbox
first_lunch_owner = lunchboxes[0].owner_key.get()

# Get the lunchbox associated with the kindergartener we made
special_lunch = Lunchbox.query(Lunchbox.owner_key=kinder_key).fetch()
```

There are some benefits and drawbacks to this way of doing things. In this example, it's tricky to get from a particular kindergartener to their lunchbox - we need Lunchbox.query(Lunchbox.owner_key=kinder_key).fetch(). Before, we only had to query to find our kindergartener, and the lunchbox was right there. So, why would we do it this way?

A few reasons!

Before, we put the whole Lunchbox object inside of the Kindergartener with a StructuredProperty - making the kid carry it everywhere. In the database, it meant that each Kindergartener object was larger, which might slow down our reads and writes. Now, we put a property on the lunchbox, saying which kindergartener it belongs to - like putting lunches with nametags in a big box. The kindergarteners can run and play, without carrying their Lunchboxes around with them.

With this ‘nametags’ approach (keeping KeyProperties on objects), we can also represent more complicated relationships. When building big awesome web apps, there are lots of times we want to keep track of things that are more complicated than just a few properties.

What if one day, a Kindergartener brought in two lunchboxes? Or a pair of siblings shared a lunchbox? It's important to structure your database relationships in a way that makes sense for your application.

Managing keys which point to objects makes our code more comprehensible and works faster and better. We'll use Keys to represent relationships in our apps.
