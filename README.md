# Mass Assignment as Metaprogramming

## Objectives

1. Understand how to use mass assignment to metaprogram a Ruby class.

## Introduction

You might recall that metaprogramming is the practice of writing code that writes code for us. So, what does that have to do with mass assignment? We've learned that we can write a Ruby program that sends a web request to an API and returns particular data to the program.

Let's say we want to use the Twitter API to create users for our own application. The scenario is that we are developing a web application and we want our users to be able to sign in via Twitter. Thus, our own users are pulled from Twitter and we need to take the data we get from Twitter––for example a user's name, age and location, and use them to make instances of our own User class. Let's take a look at a code snippet:


```ruby
class User
  attr_accessor :name, :age, :location, :user_name

  def initialize(user_name:, name:, age:, location:)
    @user_name = user_name
    @name = name
    @location = location
    @age = age
  end
end
```
Here we have our user class. It initializes with keyword arguments, i.e., a hash of attributes. For the purposes of this example, we won't get into the specifics of how we request and receive data from the Twitter API. Suffice to say that we send a request to the Twitter API and get a return value of a hash full of user attributes. For example:

```ruby
twitter_user = {name: "Sophie", user_name: "sm_debenedetto", age: 26, location: "NY, NY"}
```

With what we've learned of mass assignment so far, we can use the `twitter_user` hash to instantiate a new instance of our own User class:

```ruby
sophie = User.new(twitter_user)
 => #<User:0x007fa1293e68f0 @name="Sophie", @age=26, @user_name="sm_debenedetto", @location="NY, NY">
```

So far so good. But, what if Twitter changes their API without telling us? (How could they? Don't they know who we are?). After all, we are not in charge of Twitter or their API, they can do whatever they want, whenever they want, with no regard to our application which relies on their data. Let's say Twitter makes a change that we're unaware of. Now when we request data from their API, we get this return value:

```ruby
new_twitter_user = twitter_user = {name: "Sophie", user_name: "sm_debenedetto", location: "NY, NY"}
```

Notice that the `twitter_user` no longer has an age.  Let's see what happens if we try to create new Users using the same old User class code:

```ruby
User.new(new_twitter_user)
=>ArgumentError: missing keyword: age
```
Our program broke! Let's play it with another scenario. Let's say the Twitter API changed and now returns data to us in the following manner:

```ruby
newest_twitter_user = {name: "Sophie", user_name: "sm_debenedetto", age: 26, location: "NY, NY", bio: "I'm a programmer living in NY!"}
```

Now let's see what happens when we try to make a new instance of our User class with the same old User class code:

```ruby
User.new(newest_twitter_user)
=> ArgumentError: unknown keyword: bio
```

Our program breaks! Clearly, we need a way to *abstract away* our User class' dependency on specific attributes. If only there was a way for us to tell our User to get ready to accept some unspecified number and type of attributes.

## Mass Assignment and Metaprogramming

Guess what? We can achieve exactly that goal using metaprogramming and mass assignment. Let's take a look at how it's done, then we'll break it down together. Here's our new and improved User class:

```ruby
class User
  attr_accessor :name, :user_name, :age, :location, :bio

  def initialize(attributes)
    attributes.each {|key, value| self.send(("#{key}="), value)}
  end
end
```

We define our initialize method to take in some unspecified `attributes` object. Then, we iterate over each key/value pair in the attributes hash. The name of the key becomes the name of a setter method and the value associated with the key is the name of the value you want to pass to that method. The ruby `.send` method then calls the method name that is the key’s name, with an argument of the value. In other words:

```ruby
self.send(key=, value)
```

Is the same as:

```ruby
instance_of_user.key = value
```

Where each key/value pair is a member of our hash, one such iteration might read:

```ruby
...
instance_of_user.name = "Sophie"
...
```

And have the same result as:

```ruby
instance_of_user = User.new
instance_of_user.name = "Sophie"
```

### A Closer Look at `.send`

The `.send` method is just another way of calling a method on an object. For example, we know that instances of the User class have a `.name=` method that allows us to set the name of a user to a particular string:

```ruby
sophie = User.new
sophie.name = "Sophie"
```

Now, when we use the `.name` getter method, it will return the correct name:

```ruby
sophie.name
  => "Sophie"
```

Let's look at the same behavior using `.send`

```ruby
sophie = User.new
sophie.send("name=", "Sophie")
```

That is generally considered to be clunky and ugly. It's whats known as "syntactic vinegar". We prefer the "syntactic sugar" of the first approach.

The `.send` method, however, is a very useful tool for our metaprogramming purposes. It allows us to abstract away the specific method call:

```ruby
sophie = User.new
sophie.send("#{method_name}=", value)
```

This is exactly what's happening in our initialize method in the example above, where `self` refers to the User instance that is being initialized at that point in time.

## Why is this useful?

With this pattern, we have made our code much more flexible. We can easily alter the number of attributes in the class and change the hash that we initialize the class with, *without editing our initialize method.* Now, we're programming for the future. If and when that data with which we want to initialize our class changes, we *only have to change our attr_accessors*. Our initialize method is flexible and we can leave it alone. That is one major goal of design in object oriented programming––the writing of code that accommodates future change and doesn't require a lot of modification, even as it grows.

<p data-visibility='hidden'>View <a href='https://learn.co/lessons/mass-assignment-metaprogramming' title='Mass Assignment as Metaprogramming'>Mass Assignment as Metaprogramming</a> on Learn.co and start learning to code for free.</p>
