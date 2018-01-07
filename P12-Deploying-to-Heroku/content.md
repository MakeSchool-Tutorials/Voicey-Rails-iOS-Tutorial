---
title: "Deploying to Heroku"
slug: deploying-our-app-to-heroku
---

Next up is deploying voicey to Heroku.

# Configuring for Heroku

1. Create a Heroku account if you don't have one already.

[Heroku Homepage](www.heroku.com)

1. Create a new Heroku app. Don't add the angle brackets <>:

```shell
heroku create <enter the name of app>
```

1. Add fill in of this file with your Amazon s3 keys.

```shell
heroku config:set S3_BUCKET_NAME=your_bucket_name

heroku config:set AWS_ACCESS_KEY_ID=your_access_key_id

eroku config:set AWS_SECRET_ACCESS_KEY=your_secret_access_key

heroku config:set AWS_REGION=your_aws_region
```

1. Provision a postgres database with:

```shell
heroku addons:create heroku-postgresql:hobby-dev
```

1. Add our heroku database to the database.yaml file.

```ruby
production:
  adapter: postgresql
  encoding: unicode
  pool: 5
  database: <%= ENV['DATABASE_URL'] %>
  username: <%= ENV['DATABASE_USERNAME'] %>
  password: <%= ENV['DATABASE_PASSWORD'] %>
```

1. Push to heroku:

```shell
git push heroku master
```

1. Run our migration for our new database

```ruby
heroku run rails db:migrate
```

You should now have a running API on heroku!
