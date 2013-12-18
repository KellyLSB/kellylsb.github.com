---
layout: post
title: "Rails Structure Explained: Part 2"
description: "Explaining the App folder Part 1 (Assets and Controllers)"
category: rails
tags: [ruby, rails, optimization, organization]
---
{% include JB/setup %}

Weeks later (way later). I am returning to explain some of the pain points of the app folder. Before I get into it I want to remind that these are my personal development opinions and do not necissarily reflects the views of Ruby, Rails or other ruby enthusiasts. That's ok though, diversity in ideas is important.

# The App folder...

    ./app
    ├── assets
    ├── controllers
    ├── helpers
    ├── mailers
    ├── models
    └── views

Most of us working with rails and many other ruby applications are already familiar with this structure and what it means. What most of us don't do is use them "properly." Of course proper is relative to the type of application being written and the team writing it, but I can't tell you how many times I been asked to look at a "awesome" code base and I had to dig in it for a bit to make heads or tails of what I was looking at. Now then again I suck at sarcasm (understanding it anyway) so who knows they may have been sarcastic.

# Assets

This really oughta go away. I am not fond of the assets pipeline. Yes, it is really helpful but this is one of those parts of rails that is not explained well. It's great if you want to load very specific parts of your assets of your site into very specific controllers and views, but no one really does that. Seems to me that the asset pipeline almost always turns into a site wide reusable stylesheet that can be written in SASS, LESS, whatever and gets minified and compiled down.

Great! But I can do that easier and faster with a tool like Codekit.

What I think the asset pipeline oughta be used like is to load controller specific and application specfic assets into the various controllers and views as required. In example. In the default layout of Rails `./app/views/layouts/application.html.erb` you generally will find a line that looks like this `<%= stylesheet_link_tag 'application' %>`. Now most people will think "OH! Yah that's what includes the ./app/assets/stylesheets/application.css.scss file." Which is true, but did you know that you can use those in any view are layout (generally you still want it in the head, but due to HTML being so loose, it is not required and could be placed anywhere). This is helpful because I could create a secondary header for my UserController and place `<%= stylesheet_link_tag 'user' %>` in the head. If I do this across all my controllers I will wind up not only including the minified versions of the css like I would before. I will also only be loading the css required for the one page.

It also has the benefit of loading multiple small files over one massive file which breaks up the request and speeds up the transfer (which is helpful for those with throttled bandwidth per request, etc).

The best part about this is this does not just apply to stylesheets. There is a similar mechanism for JS and even images (less so on the images front), but organizin your images to follow the convention is still very helpful.

Otherwise just use something like Codekit and don't make me wait for rails to recompile my assets every load (in dev).

## Gzipped Assets

Rails built in asset precompiled `bundle exec rake assets:precompile` also generates gzipped versions of your assets. If you allow setup your webserver to send the pregenerated file instead of zipping on the fly you can save some server resources.

I also know that there was a security hole (filled my CORS) that would allow a hacker to break your SSL key through compressed data transfers. I don't know for certain, but the precompressed file might help protect against that type of attack.

I have a talk about it from Toorcon San Diego 2013 on DvD, I was hoping to find it on youtube to link, but I can't seem to find it. If you can I would recomend looking for it. The talk was called "Breaching SSL One Byte at a Time." I would upload it myself, but I don't want to exploit the DvD sales :P

# Controllers

Yah the controllers folder is my favorite, why because I spend almost no time there every (hopefully). The controllers folder is the "Rakk Application Directory" as I like to call it (yes I said Rakk, because Borderlands taught me to hate them sons of bitches).

The controllers in Rails are designed to implement a basic CRUD model which for those of you who are not familiar is 'create, retrieve, update, destroy' with restful access.

I didn't want to go into the `./config/routes.rb` file in those post, but I'm going to wrap it in a little because I feel it is slightly necessary.

## Routes as it relates to resourceful routes

So in short the `routes.rb` is essentially a glorified wrapper for `Rack::Cascade`, now I'm going to stop the indepth on that one right there because now that's a derailment *chuckles* (not as bad as BART though, haha). `routes.rb` has a couple helper methods like `resource` and `resources` which implement the restful access and CRUD method access in the routes automatically.

So `resource :user` would generate the following routes and mappings

    GET       /user/new  -> UserController.new
    POST      /user      -> UserController.create
    GET       /user      -> UserController.show
    GET       /user/edit -> UserController.edit
    PATCH/PUT /user      -> UserController.update
    DELETE    /user      -> UserController.destroy

and `resources :users` would generate the following routes and mappings

    GET       /users          -> UsersController.index
    GET       /users/new      -> UsersController.new
    POST      /users          -> UsersController.create
    GET       /users/:id      -> UsersController.show
    GET       /users/:id/edit -> UsersController.edit
    PATCH/PUT /users/:id      -> UsersController.update
    DELETE    /users/:id      -> UsersController.destroy

WOW! that did a ton of work for us automatically! Now let's step back to the controllers. With all this I should never have to write any non resourceful routes (theoretically). I'm not going to go into "classic" routes because well they suck, are not generally needed and I hate them.

All you really oughta know is you can add member actions and collection actions with something along the lines of.

    resources :users do
      member do
        get :update_profile
      end

      collection do
        get :search
      end
    end

Which would do the same as the `resources :users` example above with the addition of

    GET /users/:id/update_profile -> UsersController.update_profile
    GET /users/search             -> UsersController.search

Still hungry for routes info: [Rails Routing for the Outside In](http://guides.rubyonrails.org/routing.html)

## Controller Methods

Controllers are great, just please please please do not abuse filters. They are great, but they are really there for things like security or loading a class on a nested controller (if it's nested, and determining if it is nested). Also please don't nest a controller farther then two.

I guess I do have to go back to routes. If you are not aware you can nest a controller like this.

    resources :users do
      resource :profile_photo
    end

That would place the profile photo routes under the namespace of `/users/:id/profile_photo` and then include all the routes of that resource! What a life and time saver!

Every method in a controller has access to the `params`, `request` and `session` attributes (if applicable, session is not always used). These are your key attributes of your controller.

Params is a hash of all the HTTP params passed (both get and post) though starting in Rails 4 it is all [strong parameters](https://github.com/rails/strong_parameters) which can be included into your Rails 3 project when working towards an upgrade. Rails strong parameters allow you to securly decide which data to pass into your models (which I have not gotten to yet). Params will also include url attributes (i.e. `/users/1/show` would set `params[:id]` to '1', as decribed above).

Request is a hash of all the server request data and will have things like USER_AGENT and REMOTE_ADDR, etc... in it. It's mostly useful for security, analytics and session tracking, etc.

Session is a hash of session data. I would always recomend using a database for you sessions because domains are restricted to 4093 bytes of data for cookie storage. If your not using a database I would at least have a SQLite database that gets truncated everyday and every application start to keep track of some data. It is most commonly used to store things like the currently logged in users id or occasionally the entire model (that has been [Marshalled](http://www.ruby-doc.org/core-2.0.0/Marshal.html))

Haha this post has gotten a lot longer than I initially anticipated, can you tell I'm starting to lose focus? I will pick up with Helpers later.
