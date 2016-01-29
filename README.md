

# App Engine Datastore Relationships

## Objectives

+ Understand why we would want to represent relationships between data in a database

## Motivation / Why Should You Care?

In many web applications, we don’t just care about the data itself - we care about the relationships between data. A giant list of all the posts on facebook wouldn’t be very interesting - we want to associate posts to users, so that you see posts from your friends, not from random people everywhere.

## Lesson

Here we will talk about different ways we could potentially show the relationships between data. As an example, we'll build a database that keeps track of a Kindergarten class. One thing we know about kindergarteners - they’re always losing track of their lunchboxes! Let’s see what we can do make sure that we know whose lunch is whose.

Let's start with our models. 

Kindergarteners: name, birthday, favorite food
Lunchboxes: Drink, foods, logo, insulated

Each kindergartener has a lunchbox, each lunchbox has a kindergartener. How could we represent this in datastore? There are many ways we could do it, but we’re going to focus on three general ways: 

1. Lunchbox properties as properties on Kindergarteners
2. Lunchbox as an ndb model and a structured property of a Kindergartener
3. Lunchbox as another ndb model referenced by the kindergartener.  

### 1. Relating objects by not relating objects

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

### 2. Relating objects with `StructuredProperty`

The second way is more interesting, and it uses a new `ndb` property - `StructuredProperty`. `StructuredProperty` lets us use a model as a property. So each kindergartner has a lunchbox, but instead of that lunchbox have a datatype of string or integer or boolean, it had a datatype that is _**structured**_ like the Lunchbox model. 

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

# using the models to create a kindergartener who also happents to be a robot!
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

Kindergarteners each have a Lunchbox. We also get some other benefits from this separation: if we ever wanted to have Lunchboxes independent of Kindergarteners - say there were high schoolers who also brought lunch - we can reuse our code. Nice!

### 3. Objects related by reference to their keys

The third way is even more interesting! It uses a new ndb property - KeyProperty. In this case, each lunchbox will be associated with their owner using a unique identification, a nametag if you will, which is the coding world is called a key.

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

## here we make a new key, or unique identifier that containts all of the information for robot_kindergartener
robot_kinder_key = robot_kindergartener.put()

robo_lunch = Lunchbox(
	foods = ["Brains","Spare Parts"],
	drink = "motor oil",
	insulated = False,
	logo = "a string with the logo image data",
	##robo_lunch is a Lunchbox that belongs to the child who has the robot_kinder_key 
	owner_key = robot_kinder_key 
	)

## now we need to make sure we make a key for this specific Lunchbox so we can access it later
lunch_key = robo_lunch.put()

# Get all the lunchboxes
lunchboxes = Lunchbox.query().fetch()

# Get the kindergartener associated with the first lunchbox
first_lunch_owner = lunchboxes[0].owner_key.get()

# Get the lunchbox associated with the robot kindergartener 
special_lunch = Lunchbox.query(Lunchbox.owner_key=robot_kinder_key).fetch()
```

There are some benefits and drawbacks to this way of doing things. In this example, it's tricky to get from a particular kindergartener to their lunchbox - we need Lunchbox.query(Lunchbox.owner_key=robot_kinder_key).fetch(). Before, we only had to query to find our kindergartener, and the lunchbox was right there. So, why would we do it this way?

A few reasons!

#####1. Lunchboxes and Kindergartners are seperate objects
Before, we put the whole Lunchbox object inside of the Kindergartener with a StructuredProperty - making the kid carry it everywhere. In the database, it meant that each Kindergartener object was larger, which might slow down our reads and writes. But with KeyProperty, we put a property on the lunchbox, saying which kindergartener it belongs to - like a smart teacher would put nametages on lunchboxes. The Kindergarteners can run and play, without carrying their Lunchboxes around with them.

#####2. Ability to add more models and grow
With this ‘nametags’ approach (keeping KeyProperties on objects), we can also represent more complicated relationships. When building big awesome web apps, there are lots of times we want to keep track of things that are more complicated than just a few properties.

What if one day, a Kindergartener brought in two lunchboxes? Or a pair of siblings shared a lunchbox? It's important to structure your database relationships in a way that makes sense for your application.

####Conclusion
Managing keys which point to objects makes our code more comprehensible and works faster and more efficiently. We'll use KeyReference to represent relationships in our apps.

<p data-visibility='hidden'>View <a href='https://learn.co/lessons/cssi-9.2-appengine-datastore-relationships' title='App Engine Datastore Relationships'>App Engine Datastore Relationships</a> on Learn.co and start learning to code for free.</p>
