---
title: "Adding Authentication to Voicey"
slug: adding-authentication
---

We worked on fleshing out the details of a _User_ in the previous section. Lets add authentication to our Rails app. _Users_ are going to have to login to view their _Memos_.

# Adding token based authentication to our app

We are going to handle authentication in our Rails app with tokens. We are going to be adding new field in the _users_ table to store our token.

Tokens are representations of authentication. They are useful because they can't be decrypted to anything useful.

```ruby
rails generate migration AddPasswordAndTokenToUsers token:string:index password_hash:string, password_salt:string
```

This will add a token and password field to the _User_ model and add the token field as an unique index - no two _users_ will have the same token.

# Adding bcrypt to our Rails app

We are going to be using the bcrypt Gem for hashing passwords in Ruby. Lets add the brypt Gem to our Gemfile:

```ruby
gem 'bcrypt', '~> 3.1', '>= 3.1.11'
```

> [action]
> Run bundle install to install bcrypt
>

Add these two functions to the _user.rb_ model.

```ruby
def self.authenticate(email, password)
  user = self.find_by_email(email)
  if user && user.password_hash == BCrypt::Engine.hash_secret(password, user.password_salt)
    user
  else
    nil
  end
end

def encrypt_password do
  if password.present?
    self.password_salt = BCrypt::Engine.generate_salt
    self.password_hash = BCrypt::Engine.hash_secret(password, password_salt)
  end
end
```

> [info]
> What is happening here?
>  We have defined two functions, self.authenticate and encrypt_password.
> The self.authenticate function is first searching our database for a User with a specific email passed as a parameter. Then is checks to see if the user is not nil and that the password passed as a parameter hashes to the user stored in the database. This is used when logging in a user.
>
> The encrypt_password function is used to hash the password of a user.
>

Add this line to your _user.rb_ file inside the class definition:

```ruby
# 1. Hash password before saving a User
before_save :encrypt_password

# 2. Generate a token for authentication before creating a User
before_create :generate_token

# 3. Adds a virtual password field, which we will use when creating a user
attribute :password, :string
```

> [info]
> This line, like the name suggests runs the encrypt_password function when you decide to save a user to the database. After you create a user model and run the save function, it will run the encrypt_password function right before saving to the database.
>

Your _user.rb_ file should look like this now:

```ruby
class User < ApplicationRecord
  has_many :memos
  validates :name, :password, presence: true
  validates :email, presence: true, uniqueness: true
  attribute :password, :string
  before_save :encrypt_password
  before_create :generate_token

  # Checks if a user is authenticated
  def self.authenticate(email, password)
    user = self.find_by_email(email)
    if user && user.password_hash == BCrypt::Engine.hash_secret(password, user.password_salt)
      user
    else
      nil
    end
  end

  # Salts and hashes a user's password
  def encrypt_password
    if password.present?
      self.password_salt = BCrypt::Engine.generate_salt
      self.password_hash = BCrypt::Engine.hash_secret(password, password_salt)
    end
  end

  # Generates a token for a user
  def generate_token
    token_gen = SecureRandom.hex
    self.token = token_gen
    token_gen
  end
end
```

Lets modify our _application.rb_ file to set a current user variable and require authentication to use our Rails app.

```ruby
# 1. Import HttpAuthentication library from ActionController
include ActionController::HttpAuthentication::Token::ControllerMethods
# 2. Require authentication for all controller in our app
before_action :require_login

# 3. This will be used/called when we need authentication
def require_login
  authorize_request || render_unauthorized("Access denied")
end

# 4. Helper method to find the current_user in a request
def current_user
  @current_user ||= authorize_request
end

# 5. Renders an message when a user is unauthorized
def render_unauthorized(message)
  errors = { errors: [ { detail: message } ] }
  render json: errors, status: :unauthorized
end

private
# 6. Authenticate a user with by token
def authorize_request
  authenticate_with_http_token do |token, options|
    User.find_by(token: token)
  end
end
```

Our application.rb file should now look like this:

```ruby
class ApplicationController < ActionController::API
  # 1. Import HttpAuthentication library from ActionController
  include ActionController::HttpAuthentication::Token::ControllerMethods
  # 2. Require authentication for all controller in our app
  before_action :require_login

  # 3. This will be used/called when we need authentication
  def require_login
    authorize_request || render_unauthorized("Access denied")
  end

  # 4. Helper method to find the current_user in a request
  def current_user
    @current_user ||= authorize_request
  end

  # 5. Renders an message when a user is unauthorized
  def render_unauthorized(message)
    errors = { errors: [ { detail: message } ] }
    render json: errors, status: :unauthorized
  end

  private
  # 6. Authenticate a user with by token
  def authorize_request
    authenticate_with_http_token do |token, options|
      User.find_by(token: token)
    end
  end
end
```

> [action]
> Run your server with: rails server and try making a GET request to the memo resource. It should now render a 401 unauthorized with errors as a JSON response.
>

<!--  -->

> [info]
> In the application controller, we have required that all requests will require authentication by token unless we explicitly opt out of doing so through the before action called require_login.
>

The current_user function

> [info]
> The current_user function in the application.rb is going to be very useful for us. In authorized requests, we can find out who the logged in user is from this method.
>

Next up is modifying our users_controller.rb file to handle user creation.

1. Change the user_params function in users_controller.rb to this:

```ruby
def user_params
  params.permit(:name, :email, :password)
end
```

2. Add this line under the before_action of the controller:

```ruby
skip_before_action :require_login, only: [:create], raise: false
```

This will tell Rails not to run the require_login function when creating a user. Because a user doesn't have to be logged in to create an account.

Our users_controller.rb file should know look like this:

```ruby
class UsersController < ApplicationController
  before_action :set_user, only: [:show, :update, :destroy]
  skip_before_action :require_login, only: [:create], raise: false

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
      params.permit(:name, :email, :password)
    end
end

```

Lets also modify our memo_controller.rb file to account for the authentication.

Add this line to your memo_controller.rb file in the create function after you initialize a memo

```ruby
# Add the current logged in user as the creator of the memo
@memo.user = current_user
```

Our memo_controller.rb file should look like this now:

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
    # Add the current logged in user as the creator of the memo
    @memo.user = current_user

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
      params.permit(:voice_file, :title, :text_body, :time)
    end
end
```

# Testing our authentication code

## Adding tests for the password field

Lets modify our _User_ model tests to account for authentication.

Go to the spec/model/user_spec.rb and change the following:

1. We will need to add the password field when creating a user.
1. We need to test for a case when there is an empty password, the user should be invalid.

> [challenge]
> Can you modify the tests to account for the password field we added to users?
>

Solution: Our user_spec.rb file should look like this now.

> [solution]
>
```ruby
require 'rails_helper'
RSpec.describe User, type: :model do
  describe "Validations" do
    it "is valid with valid attributes" do
      user = User.new(name: "Eliel", email: "eliel@test.com", password: "test")
      expect(user).to be_valid
    end
    it "is invalid without a name" do
      bad_user = User.new(name: nil, email: "test@mail.com", password: "test")
      expect(bad_user).to_not be_valid
    end
    it "is invalid without an email" do
      bad_user = User.new(name: "Eliel", email: nil, password: "test")
      expect(bad_user).to_not be_valid
    end
    it "is invalid without a password" do
      bad_user = User.new(name: "Eliel", email: nil, password: nil)
      expect(bad_user).to_not be_valid
    end
  end
  describe "Associations" do
    it "should have many memos" do
      assoc = User.reflect_on_association(:memos)
      expect(assoc.macro).to eq :has_many
    end
  end
end
```
>

## Adding tests for the users controller

Lets add a few functions to test our users_controllers.rb file.

Our user_spec.rb file should look like this now:

```ruby
require 'rails_helper'

RSpec.describe "UserControllers", type: :request do
  describe "GET /user_controllers" do
    context "unauthorized" do
      before {
        get "/users"
      }

      it "fails when there is no authentication" do
        expect(response).to_not be_success
      end
    end
    context "authorized" do
      before {
        user = User.new(
          name: "Eliel",
          email: "eliel@test.com",
          password: "test"
        )

        user.save

        # Compose token for request
        full_token = "Token token=#{user.token}"

        get "/users", headers: { 'Authorization' => full_token }
      }
      it "succeeds when there is authentication" do
        expect(response).to be_success
      end
    end
  end

  # Test signing up a user
  describe "POST /user_controllers" do
    context "valid params" do
      before {
        valid_params = {name: "Eliel", email: "eliel@test.com", password: "testpassword"}
        post "/users", params: valid_params
      }

      it "creates and sends success of creating a user with valid params" do
        expect(response).to be_success
      end
    end
    context "invalid params" do
      before {
        invalid_params = {email: "eliel@test.com", password: "testpassword"}
        post "/users", params: invalid_params
      }

      it "should fail and send 400" do
        expect(response).to_not be_success
      end
    end
  end
end
```

What is happening here?

> [info]
> We are testing for certain behaviors of our Rails app when it is authenticated and when it isn't. We test the scenario when there is no authentication - it should fail and when we do have valid authentication - we should get a successful (200...299) response.
>

# Refactoring our authentication code
