---
title: "The User Controller"
slug: the-user-controller
---

In the previous sections, we created the models for both _User_ and _Memo_. In this section, we are going to be creating the controllers for both of them.

# Rails controllers

Rails controllers are responsible for routing incoming traffic from the network.

## The User Controller

Lets create a new controller for _User_.

```ruby
rails generate scaffold_controller User
```

This will create a new file called _users_controller_ in app/controllers.

## Structure of a Rails controller & Rest

Your _users_controller_ should now contain the following functions that correspond to _REST_ and _CRUD_:

- index - For showing all items of a resource. eg. All Users, all Memos. Translates to a _GET_ request.
- show - For showing a singular item of a resource by a specific identifier. eg. one user by id, one memo by title. Translates to a _GET_ request using url parameters to fetching the specific resource.
- create - You usually creates the appropriate model in this resource. Corresponds to a _POST_ request.
- update - Updates an existing resource. Corresponds to a put/patch request.
- destroy - Deletes a specific resource. Corresponds to a delete request.

> [info]
> Using the rails scaffold_controller generator creates some boilerplate code for all the _CRUD_ actions in a controller
>

Lets run our rails app now to verify that everything is working:

# Routes

Before we test our new models and controllers, we have to add some routes to our rails app.

Routes map _URLs_ to controllers.

Open the _routes.rb_ file located under config folder and add the following code.

```ruby
resources :users
```

Your _routes.rb_ file should look like this now:

```ruby
Rails.application.routes.draw do
  resources :users
end
```

## Previewing our routes

We can tell Rails to show us all the routes in our app by running the following command in terminal.

```ruby
rails routes
```

This should print the following:

```
Prefix Verb   URI Pattern          Controller#Action
 users GET    /users(.:format)     users#index
       POST   /users(.:format)     users#create
  user GET    /users/:id(.:format) users#show
       PATCH  /users/:id(.:format) users#update
       PUT    /users/:id(.:format) users#update
       DELETE /users/:id(.:format) users#destroy
```

We can see here that it created _RESTful_ routes for our _User_ resource.

Lets test our _User_ resource by running our rails server and going to:
 [http://localhost:3000/users](http://localhost:3000/users)

This should print out a _JSON_ array of _Users_.

> [action]
> Can you trace what is happening here?
>

<!--  -->

> [info]
> When we make a get request to [http://localhost:3000/users](http://localhost:3000/users), Rails routes this action to the index function of the users_controller.
>
