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

This will create a new file called users_controller.rb in app/controllers.

```
├── ./controllers
│   ├── ./controllers/application_controller.rb
│   ├── ./controllers/concerns
│   └── ./controllers/users_controller.rb
```


## Structure of a Rails controller

Your _users_controller_ should now contain the following functions that correspond to _REST_ and _CRUD_:

- index - For showing all items of a resource. eg. All Users, all Memos. Translates to a _GET_ request.
- show - For showing a singular item of a resource by a specific identifier. eg. one user by id, one memo by title. Translates to a _GET_ request using url parameters to fetching the specific resource.
- create - You usually creates the appropriate model for a resource. Corresponds to a _POST_ request.
- update - Updates an existing resource. Corresponds to a put/patch request.
- destroy - Deletes a specific resource. Corresponds to a _DELETE_ request.

> [info]
> Using the rails scaffold_controller generator creates some boilerplate code for all the _CRUD_ actions in a controller
>

Lets run our rails app now to verify that everything is working:

```shell
rails server
```


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

```shell
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
> When we make a get request to [http://localhost:3000/users](http://localhost:3000/users), Rails routes this action to the index function of the users controller.
>

# Parameter Sanitizing

With our current controller setup, Rails will allow any parameter into our controller. Lets fix that by only whitelisting parameters for creating a user.

Modify the user_params function to add this code:

```ruby
def user_params
  params.permit(:name, :email)
end
```

The users_controller.rb file should now look like this:

```ruby
class UsersController < ApplicationController
  before_action :set_user, only: [:show, :update, :destroy]

  # GET /users
  def index
    @users = User.all

    render json: @users
  end

  # GET /users/1
  def show
    render json: @user
  end

  # POST /users
  def create
    @user = User.new(user_params)

    if @user.save
      render json: @user, status: :created, location: @user
    else
      render json: @user.errors, status: :unprocessable_entity
    end
  end

  # PATCH/PUT /users/1
  def update
    if @user.update(user_params)
      render json: @user
    else
      render json: @user.errors, status: :unprocessable_entity
    end
  end

  # DELETE /users/1
  def destroy
    @user.destroy
  end

  private
    # Use callbacks to share common setup or constraints between actions.
    def set_user
      @user = User.find(params[:id])
    end

    # Only allow a trusted parameter "white list" through.
    def user_params
      params.permit(:name, :email)
    end
end
```

# Summary

- Controllers can manage CRUD on a resource by providing us with methods to handle diffent types of HTTP requests.
- Routes map URLs to the appropriate controllers.
- We define our routes in the routes.rb file under the config folder.
- The parameter sanitizer only allows whitelisted parameters into our controllers.
