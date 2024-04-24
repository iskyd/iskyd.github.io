+++
title = 'Wasabi.jl - ORM for Julia'
date = 2023-05-13
draft = false
+++

Three days ago I've released Wasabi 0.3.0 on the Julia General Registry. 
Wasabi is a simple yet powerful ORM for the Julia Language that currently support PostgreSQL and SQLite and gives a simple way to manage database changes using Migrations.

**WHY**

There are 3 main reasons why I decided to start this project few months ago:
1. I dind't find any available library for Julia written with Domain Driven Design in mind. I always want to rely on my domain object. BTW I have found an amazing project that act as query DSL called [Octo.jl](https://github.com/wookay/Octo.jl). Another ORM that I've found is [SearchLight](https://github.com/GenieFramework/SearchLight.jl) as part of the GenieFramework
2. I'm starting a new project and I want to build a microservices architecture using Julia. An ORM could help me reduce errors, don't rely on a specific database implementation and have faster iterations. I want to be able to change backend database by simply changing some environment variables.
3. The crime level in Gotham has dropped so I found myself with a lot of free time and I want to have fun.

**WASABI**

At the moment Wasabi supports SQLite and PostgreSQL. This is because I'm going to use SQLite as testing database and PostgreSQL as production database. I hope to add MySQL support as soon as possibile.
Using Wasabi you can define a model and it's constraints such as primary keys, foreign keys and unique constraints. You can than automatically create tables that reflect the model implementation.
Wasabi supports basic methods to insert, update and delete rows directly using the domain object.

**QUERY BUILDER**

Wasabi provide a powerfull query builder to perform select queries without writing SQL and it automatically takes care of query parameters for security. At the moment you can perform join, where, group by, limit, offset and order by using Query Builder. I think that it have pretty good syntax.

**MIGRATIONS**

Wasabi also provide out of the box a Mirgation service to manage database upgrade/downgrade. This is essential for startup projects that needs to change over time and over market experience.

**WHAT'S NEXT**

There are two main things I want to improve in the next weeks
1. Support previous version of Julia using [Requires.jl](https://github.com/JuliaPackaging/Requires.jl)
2. Add update and delete query support in query builder
