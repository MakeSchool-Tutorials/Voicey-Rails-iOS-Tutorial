---
title: "Deploying to Heroku"
slug: deploying-to-heroku
---

Next up is deploying voicey to Heroku.

# Configuring for Heroku

Add fill in of this file with your Amazon s3 keys.

```shell
heroku config:set S3_BUCKET_NAME=your_bucket_name

heroku config:set AWS_ACCESS_KEY_ID=your_access_key_id

eroku config:set AWS_SECRET_ACCESS_KEY=your_secret_access_key

heroku config:set AWS_REGION=your_aws_region
```
