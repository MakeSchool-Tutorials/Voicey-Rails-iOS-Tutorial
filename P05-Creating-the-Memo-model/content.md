---
title: "Creating Memos"
slug: creating-memos
---

Lets create the _Memo_ model with the following specifications:

- title: string
- date: date
- text_body: text

# Creating the Memo model

Lets create the _Memo_ model.

> [challenge]
>
Since you have practiced creating the _User_ model. Try to create the _Memo_ model from the specifications above.

<!--  -->

> [solution]
> Solution
>
```ruby
rails generate model Memo title:string date:date text_body:text
```

<!--  -->

> [action]
> Don't forget to run rails db:migrate after generating the model
>
