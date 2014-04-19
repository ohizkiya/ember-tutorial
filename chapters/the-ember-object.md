# Ember Concepts

We could start coding right away, but I think that if you haven't seen Ember-flavored javascript before then the code might not make much sense. So before we dive into building out the app I'm going to cover some core Ember concepts and a few of the Ember objects that you'll be seeing.

If you are dying to code you can skip directly the [Our App](/our-app) chapter, but if you want to know what's going on then I recommend that you stick around.

The following few chapters will be a whirlwind tour of Ember Objects, Routing, Routes, Controllers, Views, and Templates.

# The Ember Object

Ember implements its own object system. The base object is called Ember.Object. All of the other objects in Ember extend Ember.Object.

While most of the time you will be using an object like Ember.Controller or Ember.View, you can also use Ember.Object itself. As an example, you might use it to create a service object that handles some non-persisted logic.

If you open your browser's console in your Hello World app you'll be able to follow along with these commands, though you'll need to convert the CoffeeScript to Javascript.

You can create a basic object like this:

```coffee
user = Ember.Object.create()
```

Initialize it with properties by just passing them to create:

```coffee
user = Ember.Object.create(firstName: 'Sam', lastName: 'Smith')
```

You can get a property from the object by calling `.get` on it and passing the string name of the property:

```coffee
user = Ember.Object.create(firstName: 'Sam', lastName: 'Smith')
user.get('firstName') == 'Sam' #=> true
user.get('lastName') == 'Sam' #=> true
```

Inquire about the object with `.toString()`. In this case we see that it's just Ember.Object.

```coffee
user = Ember.Object.create()
user.toString() #=> <Ember.Object:ember{objectId}>
```

## Defining Objects

So far we've just been using Ember.Object. What if you want to create our own object? You can do that like this:

```coffee
App.User = Ember.Object.extend()
```

Boom! Now you can instantiate a user like so:

```coffee
user = App.User.create()
```

Note that I'm putting this User object inside `App`. Ember needs to place all of the app data inside a variable, and Ember devs typically use `App`. So when you define some kind of Object in ember you always want to have it on `App`.

Now what we've done is great and all, but we probably want to add things to our User object. Let's do it!

Objects can have three types of things inside them: properties, functions, and observers. I'll cover each of these.

## Properties

Here's how you could define a property:

```coffee
App.User = Ember.Object.extend
  isHuman: true
  temperature: 98.6
  favoriteDirector: 'Tarantino'
```

`isGreat`, `temperature`, and `favoriteDirector` would now be accessible with `.get`. These are the basic versions of Ember properties. We can also create computed properties that actually do some work and call other properites:

```coffee
App.User = Ember.Object.extend

  fullName: ( ->
    @get('firstName') + ' ' + @get('lastName')
  ).property('firstName', 'lastName')


```

Let's dissect this.

First, we specify the property name with `fullName:`.

Second, we pass it a function that will return the value we want. We can get at other properties in our object by doing `this.get('propertyName')`, or in CoffeeScript, `@get('propertyName')`.

Third, we call `.property()` on our function to tell Ember that it's a computed property.

Finally, we have to tell `property()` which other properties this property depends on. It expects a list of the property names as strings. In this case, `fullName` should change any time `firstName` or `lastName` changes, so it neds to watch both of them.

# Functions

Functions are quite simple. Example:

```coffee
App.User = Ember.Object.extend

  showMessage: (message) -> alert(message)

  showName: -> @showMessage(@get('fullName'))


```

Our `showMessage` function takes one argument: the message we want to alert. The `showName` function calls the `showMessage` function with the `fullName` of our user (assuming we've implemented the `fullName` property).

Here's how you actually call a function:

```coffee
user = App.User.create()
user.showMessage('omg this works!')
```

## Observers

Observers are functions that fire whenever the any of the things they observe change. They look like properties but they end with `observes()` instead of `property()`

```coffee
App.User = Ember.Object.extend

  weightChanged: ( ->
    alert('yikes') if @get('weight') > 400
  ).observes('weight')


```

The above code would fire an alert saying 'yikes' whenever the weight property on this user changes and is greater than 400. You can observe as many things as you'd like:

```coffee
App.User = Ember.Object.extend

  bodyObserver: ( ->
    alert("Lookin' good man.")
  ).observes('weight', 'height')

```

This would fire every time `weight` or `height` changed.

## Extend Existing Objects

The Ember Object system is really great. It makes it easy to write modular, reusable code.

You can extend any existing object in your app like this:

```coffee
App.Animal = Ember.Object.extend
  likesFood: true

App.Human = App.Animal.extend()

human = App.Human.create()

human.get('likesFood') #=> true
```

Properties, functions, and observers in the child object will override those in the parent:

```coffee
App.Bird = App.Animal.extend
  likesFood: false

bird = App.Bird.create()
bird.get('likesFood') #=> false
```

You can even extend multiple objects:

```coffee
App.One = Ember.Object.extend()
App.Two = Ember.Object.extend()

App.Three = App.One.extend(App.Two)
# or
App.Three = Ember.Object.extend(App.One, App.Two)
```

Extending objects is a pattern you will use all the time while developing in Ember. You can use it to extract out common functionality or pull in functionality from some other object.

## Init

All ember objects call an `init` function when they are first initialized. You can use this to do setup work.

```coffee
App.Human = Ember.Object.extend

  init: -> alert("I think, therefore I am")
```

This is fine with basic Ember Objects, but if you are using other, more specific Ember Objects like Route or Controller, then you should try to avoid using init and instead opt for other Ember conventions. If you must use it in one of these objects make sure that you call `@_super()` in the `init` function, otherwise you may break things.

## Reopening Objects

You can go back and add more properties, functions, and observers by calling `reopen` on the object:

```coffee
App.Human = Ember.Object.extend()

App.Human.reopen
  name: 'Señor Bacon'

francis = App.Human.create()
francis.get('name') #=> 'Señor Bacon'
```

As you've seen, objects function like classes do in other languages. You can even define a sort of class method on objects by calling `reopenClass`:

```coffee
App.Human = Ember.Object.extend()

App.Human.reopenClass
  sayUncle: -> alert("uncle")
```

Then call the class method on the object itself:

```coffee
App.Human.uncle()
```

Ember Objects provide us with the ability to write very object-oriented javascript. This is one of Ember's best features.