---
layout: post
title: "Notes for AWD with Rails 4"
date: 2014-08-22 01:25:46 +0800
comments: true
categories: [Notes, AWD4]
---
I checked out from LHU library and grab this book last two days:

![Agile Web Development with Rails 4](https://0374e0f55efceb27e890e798ff092822246c9476.googledrive.com/host/0BxwbYxZqk44YNjA4YzZJSm1VZDQ/2014-08-22%2019.43.34.jpg "Agile Web Development with Rails 4")

And I am going to take some notes here so that I can review it easily in the future, and I also would like to record a story of how n00b wrote their thoughts down.

## Creating application with specific Rails version
Here is the example for Rails 4, the command:
``` bash Command Line
$ rails _4.0.0_ new app_name
```

## Creating your own Rails API documentation
``` bash Command Line
$ cd app_name
$ rake doc:rails
```

## Alternative syntax
`%{}` is an alternative syntax for double-quoted string literals, convenient for use with long strings:
``` ruby app/db/seed.rb
Product.create!(title: 'Baozi', description: %{
    A
    n00b
    of
    Rails
})
```
And here why I used `exclamation mark (!)` for `create` method? The reason why is that it will `raise an exception` if records cannot be inserted because of `validation failed`.

## Load specific stylesheet in specific controller
Deponds on `controller_name` method ([API](http://apidock.com/rails/ActionController/Metal/controller_name/class))
``` html app/views/layouts/application.html.erb
<body class="<%= controller.controller_name %>">
  <%= yield %>
</body>
```

## Alternate CSS classes for even and odd numbers
Deponds on `cycle` method ([API](http://apidock.com/rails/v4.0.2/ActionView/Helpers/TextHelper/cycle))
``` html app/views/layouts/application.html.erb
<h1>Listing products</h1>

<table>
  <% @products.each do |product| %>
    <tr class="<%= cycle('list_line_odd', 'list_line_even') %>">
        ...
</table>
```

## Helpful commands for Git
If you overwrite or delete files, directries that you didn't mean to, you can always get back by using this command:
``` bash Command Line
$ git checkout .
```

On the contrary, if you create folders or files you didn't want to, try:
``` bash Command Line
$ git clean -d -f
```
to get back.

## comming soon.