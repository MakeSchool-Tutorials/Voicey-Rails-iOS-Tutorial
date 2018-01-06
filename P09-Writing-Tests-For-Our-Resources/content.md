---
title: "Writing Tests For Our Resources"
slug: writing-tests-for-our-resources
---

We have written a lot of code in the previous sections of this tutorial. We are going to test what we have written so far.

Lets start with the model tests

# Testing the Models

Before we get started with testing, we are going to install a few Gems to aid with our testing.

Lets add the following Gems to our Gemfile:

```ruby
# Gemfile

group :test do
  gem 'rspec-rails', '~> 3.7', '>= 3.7.2'
  gem 'factory_girl_rails', '~> 4.9'
  gem 'shoulda-matchers', '~> 3.1', '>= 3.1.2'
  gem 'faker', '~> 1.8', '>= 1.8.7'
  gem 'database_cleaner', '~> 1.6', '>= 1.6.2'
end
```

Then run:

```shell
bundle install
```

> [info]
> Running bundle install installs the Gems in our Gemfile.

Lets then initialize the _spec/_ directory where specs will reside with:

```ruby
rails generate rspec:install
```

This should create a new folder called _spec_ in the root of our rails application.

![Spec Folder](assets/spec-folder.png)

Then run the rails server to make sure everything is working correctly:

```shell
bundle exec rspec
```

You should see something similar to this output in your terminal:

![rspec Test](assets/spec-test.png)

Lets setup Shoulda Matches to aid with our testing. Add the following to your _spec/rails_helper.rb_ file under the _require 'rspec/rails'_:

```ruby
Shoulda::Matchers.configure do |config|
  config.integrate do |with|
    with.test_framework :rspec
    with.library :rails
  end
end
```

> [action]
>Go to [http://0.0.0.0:30000](http://0.0.0.0:30000) in your browser and test any of the routes.

## Writing the tests

Lets write the test for the _User_ model.

```ruby
rails generate rspec:model User
```

## Attribute Validation

We are going to start by validating model properties.

Replace the line containing:

```ruby
pending "add some examples to (or delete) #{__FILE__}"
```

with:

```ruby
describe "Validations" do
  it "is valid with valid attributes" do
    user = User.new(name: "Eliel", email: "eliel@test.com")
    expect(user).to be_valid
  end

  it "is invalid without a name" do
    bad_user = User.new(name: nil, email: "test@mail.com")
    expect(bad_user).to_not be_valid
  end

  it "is invalid without an email" do
    bad_user = User.new(name: "Eliel", email: nil)
    expect(bad_user).to_not be_valid
  end
end
```

The second and third test should fail, indicating that we did not add any validations to our _User_ model.

Lets add some validations to make sure that a _User_ must have a name and email to be created.

# ActiveRecord Validations

Add the following to the _User_ model, right under the belongs_to:

```ruby
validates_presence_of :name, :email
```

The _User_ model should now look like this:

```ruby
class User < ApplicationRecord
  has_many :memos
  validates :name, :password, presence: true
  validates :email, presence: true, uniqueness: true
end
```

The _validates_presence_of_ tells _ActiveRecord_ to ensure that the name and email field exists before saving the _User_ model.

Lets test it out.

Run your tests again:

```shell
bundle exec rspec
```

The three (3) tests should pass now.

## Association Validation

Lets test that a _User_ has many _Memos_. Add the following test to our user_spec.rb

```ruby
describe "Associations" do
  it "should have many memos" do
    user = User.new(name: "Eliel", email: "eliel@test.com", password: "test")
    expect(user).to have_many(:memos)
  end
end
```

The final user_spec.rb file should look like this now:

```ruby
require 'rails_helper'

RSpec.describe User, type: :model do
  describe "Validations" do
    it "is valid with valid attributes" do
      user = User.new(name: "Eliel", email: "eliel@test.com")
      expect(user).to be_valid
    end

    it "is invalid without a name" do
      bad_user = User.new(name: nil, email: "test@mail.com")
      expect(bad_user).to_not be_valid
    end

    it "is invalid without an email" do
      bad_user = User.new(name: "Eliel", email: nil)
      expect(bad_user).to_not be_valid
    end
  end

  describe "Associations" do
    it "should have many memos" do
      user = User.new(name: "Eliel", email: "eliel@test.com", password: "test")
      expect(user).to have_many(:memos)
    end
  end
end
```

# Testing Memos

> [challenge]
> Can you write the tests for the Memo model?
> You will have to test for the required Memo attributes as well as the belongs_to association to User.
>

Hint:

> [info]
> In the associations describe block, you will have to match against belongs_to, so it will be: it {should belong_to(user)}
>  

<!--  -->

Rspec Test Solution

> [solution]
>
```ruby
require 'rails_helper'
# Memo Tests
RSpec.describe Memo, type: :model do
  subject {
    User.new(name: "Eliel", email: "eliel@test.com")
  }
  describe "Validations" do
    it "is valid with valid attributes" do
      memo = Memo.new(
        title: "My Memo",
        time: DateTime.now.utc,
        text_body: "This is the text body",
        user: subject
      )
      expect(memo).to be_valid
    end
    it "is invalid without a title" do
      bad_memo = Memo.new(
        title: nil,
        time: DateTime.now.utc,
        text_body: "This is the text body",
        user: subject
      )
      expect(bad_memo).to_not be_valid
    end
    it "is invalid without an time" do
      bad_memo = Memo.new(
        title: "My Memo",
        time: nil,
        text_body: "This is the text body",
        user: subject
      )
      expect(bad_memo).to_not be_valid
    end
    it "is invalid without an text body" do
      bad_memo = Memo.new(
        title: "My Memo",
        time: DateTime.now.utc,
        text_body: nil,
        user: subject
      )
      expect(bad_memo).to_not be_valid
    end
    it "is invalid without a user" do
      bad_memo = Memo.new(
        title: "My Memo",
        time: DateTime.now.utc,
        text_body: nil,
        user: nil
      )
      expect(bad_memo).to_not be_valid
    end
  end
  describe "Associations" do
    it "should have many memos" do
      memo = Memo.new(
        title: "My Memo",
        time: DateTime.now.utc,
        text_body: "This is the text body",
        user: subject
      )
      expect(memo).to belong_to(:user)
    end
  end
end
```
>

Memo model validation solution:

> [solution]
>
```ruby
class Memo < ApplicationRecord
  belongs_to :user
  validates :title, :date, :text_body, presence: true
end
```
>

# Testing the Controllers

Lets begin testing the User controller.

```ruby
rails generate rspec:request UserController
```

Run only the controller tests with:

```ruby
bundle exec rspec spec/requests
```

This should fail.

Lets modify our _UserControllerTests_ and create some tests.
