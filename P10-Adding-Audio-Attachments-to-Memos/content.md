---
title: "Adding Audo Attachments to Memos"
slug: adding-audio-attachments-to-memos
---

Voicy allows users to add voice memos. So far, we have created and tested our User and Memo resource.

In this section, we are going to add an audio attachment to _Memos_ to enable us to have voice memos.

# Using Paperclip

Paperclip is a Ruby Gem that enables us to have attachments to Rails models.

You can use it have audio, pdf, image, and video attachments.

You can take a look at the documentation here:

[Paperclip Homepage](https://github.com/thoughtbot/paperclip)

We will also be storing all our audio files in Amazon s3. We will go over creating an account on Amazon soon.

Lets add the paperclip and AWS-SDK Gem to our Gemfile:

```ruby
# Gemfile
gem 'paperclip', '~> 5.1'
gem 'aws-sdk', '~> 2.3.0'
```

## ImageMagick

Paperclip uses _ImageMagick_ so we have to install it:

```shell
brew install imagemagick
```

Now generate the migration for paperclip with:

```shell
rails generate paperclip memo voice_file
```

> [action]
> Run rails db:migrate
>

Add this configuration file to the _application.rb_ file in _config_ folder.

```ruby
config.paperclip_defaults = {
  storage: :s3,
  s3_credentials: {
    bucket: ENV.fetch('S3_BUCKET_NAME'),
    access_key_id: ENV.fetch('AWS_ACCESS_KEY_ID'),
    secret_access_key: ENV.fetch('AWS_SECRET_ACCESS_KEY'),
    s3_region: ENV.fetch('AWS_REGION')
  }
}
```

Your application.rb file should now look like this:

```ruby
require_relative 'boot'

require "rails"
# Pick the frameworks you want:
require "active_model/railtie"
require "active_job/railtie"
require "active_record/railtie"
require "action_controller/railtie"
require "action_mailer/railtie"
require "action_view/railtie"
require "action_cable/engine"

Bundler.require(*Rails.groups)

module VoiceyApi
  class Application < Rails::Application

    config.load_defaults 5.1

    config.paperclip_defaults = {
      storage: :s3,
      s3_credentials: {
        bucket: ENV.fetch('S3_BUCKET_NAME'),
        access_key_id: ENV.fetch('AWS_ACCESS_KEY_ID'),
        secret_access_key: ENV.fetch('AWS_SECRET_ACCESS_KEY'),
        s3_region: ENV.fetch('AWS_REGION')
      }
    }

    config.api_only = true
  end
end
```

> [action]
> Keep the environment properties in mind as we will come back to them in the Deploying to Heroku section.
>

## Model Attachments

Then add the following code to your _Memo_ model.

```ruby
has_attached_file :voice_file
validates_attachment :voice_file, :content_type =>['audio/mpeg', 'audio/x-mpeg', 'audio/mp3', 'audio/x-mp3', 'audio/mpeg3', 'audio/x-mpeg3', 'audio/mpg', 'audio/x-mpg', 'audio/x-mpegaudio']
```

Your _Memo_ model should now contain the following:

```ruby
class Memo < ApplicationRecord
  belongs_to :user
  validates_presence_of :title, :time, :text_body, :user
  has_attached_file :voice_file
  validates_attachment :voice_file, :content_type =>['audio/mpeg', 'audio/x-mpeg', 'audio/mp3', 'audio/x-mp3', 'audio/mpeg3', 'audio/x-mpeg3', 'audio/mpg', 'audio/x-mpg', 'audio/x-mpegaudio']
end
```

## Updating the Memo controller

We need to update the memo controller to account for adding the _voice_file_ attachment to _Memos_.

Replace the folling line in the _memo_params_ function in the _memo_ _ _controller.rb_ file.

```ruby
params.require(:memo).permit(:voice_file, :title, :text_body, :date)
```

Your memo controller should know look like this:

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
      params.require(:memo).permit(:voice_file, :title, :text_body, :date)
    end
end
```
