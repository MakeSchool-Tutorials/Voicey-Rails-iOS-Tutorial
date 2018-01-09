---
title: "The Memo Controller"
slug: the-memo-controller
---

In the previous section, we created the controller and added the route for users.

We are going to create a controller and a route for Memos.

# Creating the Memo controller

> [challenge]
> Since you have some experience creating models, controllers and routes in rails. Can you create the _Memo_ controller and its route?
>

Hint

> [info]
> Use the rails scaffold_controller generator and don't forget to add your route to the routes.rb file.
>

<!--  -->

Solution for creating the controller:

> [solution]
> rails generate scaffold_controller Memo
>

Solution for creating and adding the _Memo_ route:

> [solution]
Add this to routes.rb
>
```ruby
resources :users
```
>
Your routes.rb file will look like this now:
>
```ruby
Rails.application.routes.draw do
  resources :users
  resources :memos
end
```
>

Your route file should now contain two routes, one for users and another for memos.

# Parameter sanitizing for the memo controller

Lets modify the memo_params function to allow parameters for creating a memo.

```ruby
def memo_params
  params.permit(:title, :text_body, :date)
end
```
