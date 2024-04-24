+++
title = 'I find out that Wasabi is not good ... yet'
date = 2023-05-31
+++

[Wasabi](https://github.com/iskyd/Wasabi.jl) is a simple ORM for the Julia Language. You can read more [here](https://iskyd.github.io/blog/2023/05/13/wasabi-julia-orm.html).

When I started developing Wasabi I have one main goal: rely on my domain object instead of the database representation. A lot of ORMs "force" you to think of your database representation and use that object also as your domain object. Of course you can apply a conversion between your domain object and your database representation but that's an overhead and in my experience almost no-one do this. In my career I've worked mainly with Doctrine (PHP) and SQLAlchemy (Python). I find Doctrine one of the best ORM to ever interact with but in both case you almost always overlapp your database representation and you're domain object. 

They're both ORM for an object oriented programming language: Julia on the other way is not an object oriented programming language so we can't rely on class attributes and methods definition to implement the behaviour, but we have to rely on multiple dispatch.

The main issues that come by overlapping database representation and domain object are:
- Codes will become more complex and hard to mantain
- Even a simple modification can require lot of working
- database modifications often prevents you doing blue/green deployment
 
In Julia using multiple dispatch you can instantiate an object in many different ways. Suppose you have a User object with firstname and lastname column and you change that to just name column. You can still instantiate the "new" object using the old database structure.

```
struct User
    name::String

    User(name::String) = new(name)
    User(firstname::String, lastname::String) = new(firstname * " " * lastname)
end

User("John Doe")
User("John", "Doe")
```

Under the hood the ORM can instantiate the object and the developer just need to provide a valid constructor for every states of the model.
At the moment Wasabi use a method df2model to perform the conversion from the Dataframe (results from the database) to the model and converting types back and forth (for example you can have a String on database that is a DateTime in your model).
You can use multiple dispatch directly on the constructor or you can also override directly the df2model method for more complex scenarios. This way you provide an explicit conversion between your domain object and the database.

So what's wrong at the moment? The main issue I've found is models with relations. Suppose that User has some roles implemented by the Role model:

```
struct Role
    name::String
end

struct User
    name::String
    roles::Vector{Role}

    User(name::String) = new(name, Role[])
    User(name::String, roles::Vector{Role}=Role[]) = new(name, roles)
end
```

But how do you represent this using Wasabi? Well you don't.
You want to represent a many to many relations so using Wasabi you do something like this:

```
struct Role <: Wasabi.Model
    id::Int
    name::String
end

struct User <: Wasabi.Model
    id::Int
    name::String

    User(name::String) = new(name)
end

struct UsersRoles
    role_id::Int
    user_id::Int
end
```

That's absolutely not good because this is the database implementation and we don't care! We want our domain object!

I was wondering how to fix this problem while mantaining a database abstraction that can be able to identify primary keys, foreign keys and so on.
I also would like to provide lazy loading operations (thanks Doctrine for inspiring me all the time).
As we've seen before our domain object is simply our User with roles::Vector{Role} as field.
It could be good if we can provide a method so that Wasabi knows how to map the object on database and we can forget about it, suppose something like this:

```
struct Role
    name::String
end

struct User
    name::String
    roles::Vector{Role}

    User(name::String) = new(name, Role[])
    User(name::String, roles::Vector{Role}=Role[]) = new(name, roles)
end

Wasabi.colmapping(::Type{User}, ::Type{Role}) = Many2Many("user_id", "role_id")
```

This way Wasabi knows exactly how to map your domain object to your database and you can rely on your domain object.
Another good feature could be to add a lazy_load params to the df2model method so that you can choose what fields to load.
Suppose you want to load the UIser object but you don't need the roles, that would result in useless queries to load all attributes from the databse.

```
Wasabi.df2model(m::Type{T}, df::DataFrame, lazy_loads=[]) where {T <: Model}
Wasabi.df2model(User, df, ["roles"])
```

This way you're not going to load roles object from the database, otherwise Wasabi can automatically load the fields from the db and provide you the domain implementation.

I think that this could be pretty good and very extensible, I will work on this as soon as I can :D
