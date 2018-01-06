---
title: "The Memo Controller"
slug: the-memo-controller
---

In the previous section, we created the controller and added the route for users.

We are going to create a controller and a route for Memos.

# Creating the Memo controller

> [challenge]
> Since you have some experience creating models, controllers and routes in rails. Can you create the _Memo_ controller and route?
>

<!--  -->

Solution for creating the controller:

> [solution]
> rails generate scaffold_controller Memo
>

Solution for creating and adding the _Memo_ route:

> [solution]
>
```ruby
resources :users
```
>

Your route file should now contain two routes one for users and another for memos.

# Adding the User association to Memo controller

Since we added the User relationship to _Memos_, we are going to add some code to the _memo_controller.rb_ file.

```ruby

```

```ruby
class MemosController < ApplicationController
  before_action :set_memo, only: [:show, :update, :destroy]

  # GET /memos
  def index
    @memos = Memo.all

    render json: @memos
  end

  # GET /memos/1
  def show
    render json: @memo
  end

  # POST /memos
  def create
    @memo = Memo.new(memo_params)

    if @memo.save
      render json: @memo, status: :created, location: @memo
    else
      render json: @memo.errors, status: :unprocessable_entity
    end
  end

  # PATCH/PUT /memos/1
  def update
    if @memo.update(memo_params)
      render json: @memo
    else
      render json: @memo.errors, status: :unprocessable_entity
    end
  end

  # DELETE /memos/1
  def destroy
    @memo.destroy
  end

  private
    # Use callbacks to share common setup or constraints between actions.
    def set_memo
      @memo = Memo.find(params[:id])
    end

    # Only allow a trusted parameter "white list" through.
    def memo_params
      params.require(:memo).permit(:title, :text_body, :date)
    end
end
```
