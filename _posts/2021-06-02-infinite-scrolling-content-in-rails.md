---
layout: post
title: Infinite Scrolling Content in Rails — Without Writing Any Javascript
description: Implement infinite scroll pagination in Rails using Hotwire and Turbo without writing custom JavaScript code.
---

In web development, you’ll often come across the term “pagination”. This is a technique that allows you to divide your content into separate pages so that your user can navigate through the content page by page. This is usually done by providing ‘forward’ and ‘back’ links, as well as links to specific pages. Another technique of pagination is an “infinite scroll” design. Instead of having the user navigate from page to page, they continuously scroll to the bottom of the page and your application automatically loads the next page below.

The traditional page based approach is sometimes trivial to implement and can use standard HTML links to link between pages. The challenge with “infinite scroll” type navigation is that it requires a bit of interactivity to get it working. This interactivity is usually added in the form of Javascript.  Writing and maintaining that Javascript may not be everyone’s cup of tea.

In this post, you’ll learn how to add a simple infinite scroll interface for your existing paged based navigation system. First you’ll go over the libraries involved before diving into the simple implementation. Later, you’ll go over when and when not to use an infinite scroll mechanism.

## Hotwire
Hotwire is a new set of front end libraries that allow you to build interactive user interfaces while writing very little Javascript. Adding the `turbo-rails` gem gives an easy way to implement what Turbo has to offer. 

```
# Gemfile
gem 'turbo-rails'

# Run to install
./bin/rails turbo:install
```

One feature that this gem provides is the `turbo_frame_tag` method.

```
<%= turbo_frame_tag 'frame-id', loading: 'lazy', src: some_path %>
```

This helper method that generates a `<turbo-frame></turbo-frame>` HTML tag. The three attributes will be key in this implementation, more on this later. This `turbo-frame` feature is pretty powerful and is what makes an infinite scroll possible without writing any Javascript.

## Kaminari
The `kaminari` gem is a great library that will add the pagination methods to your models. You could also roll your own if you want, but it’s a handy gem to have in your application since it can be used for both paged and infinite scroll type interfaces.

```
gem 'kaminari'
```

There isn’t much need to configure this gem, and it’ll work for this tutorial right out of the box.

## Controller
As you’ll see in the controller class below, this is a standard `index` action that you’ll commonly see for paged based pagination. You can use the exact same implementation here for an infinite scroll.

```
class EmailController < ApplicationController
  def index
    @emails = Email.all.page(params[:page]).per(20)
  end
end
```

Be sure to adjust the page and per parameters to suit your needs. If you have content that is lengthly you can lower the amount of items to render on the page. If your content is short, increase the number so that each page loads a good amount of records.

## View
Most of the logic you’ll need in this implementation lies in the view and the usage of the `turbo_frame_tag` helper method. Below is an example of what this implementation looks like.

```
<%= turbo_frame_tag "paginate_page_#{@emails.current_page}" do %>
  <% @emails.each do |email| %>
    <%= render 'email', email: email %>
  <% end %>
  <% if @emails.next_page %>
    <%= turbo_frame_tag "paginate_page_#{@emails.next_page}", src: emails_path(page: @emails.next_page), loading: 'lazy' do %>
      Loading...
    <% end %>
  <% end %>
<% end %>
```

You’ll notice that there are two `turbo_frame_tag` calls. The content is rendered within the first tag, and the second tag is left intentionally empty. Copy this into your view and modify as necessary. If your database contains more records than can be shown on a page, you should see the subsequent page of content loaded via an XHR request.

## Why this works
The key to this implementation is the embedded usage of the `turbo_frame_tag`.  Each call to the `index` action of the controller will render the contents of the first `turbo_frame_tag`. The second `turbo_frame_tag` is empty, but has two important attributes: `loading` and `src`. The loading attribute is a feature of Turbo where the content of the frame lazily loads the `src`. Only when the element comes into view, does the call get made. As you can see in the example, the `src` attribute is set to the “next page” of content: `emails_path(page: @emails.next_page)`. So as the user scrolls through the rendered emails of the outer frame, they’ll reach the end and the inner frame tag comes into view, Turbo will perform a request for the next page and load that into the frame. The inner frame tag only renders if we know a next page is available. Once the user gets to the end of the content, no further frames are rendered.

The first parameter of the `turbo_frame_tag` specifies the ID of that frame. A Turbo frame that is lazily loaded will only load the contents of another `turbo_frame_tag` in the result of the source. Notice that the outer frame ID is the current page being rendered, and the next page is the inner frame ID.

Essentially the end result is going to be a bunch of nested `turbo_frame_tag`s. This may or may not be satisfactory for your implementation but it works quite well in most cases. 

## When to use it
An infinite scroll interface works well when you need to display a long list of information and can’t load everything all at once. It works better when content is organized in a timeline fashion and the most relevant content is first. Here are some examples:
* social media streams
* photo albums
* comment sections

## When not to use it
If the user interacts with the data you are displaying then it might make more sense to go with a paged based pagination. If a user is meant to click on a piece of content, and then go back to the list to view another content, it’s harder (although not impossible) to keep track of where the user was in an infinite scroll interface. Also if your content is organized in a way that a user can expect to jump to a specific point, infinite scrolling won’t work well (alphabetized contact lists). Here are some examples where infinite scroll might not work well:
* forums
* index of blog posts
* contact lists

## What’s next
Hopefully you found this post useful. This is a nice quick solution in implementing infinite scroll without any Javascript. There are lot more improvements that can be made with a bit of Javascript, for example updating the browser URL to reflect what “page” you are on, but that’s beyond the scope of this article. Feel free to drop me a DM on twitter if  you have any questions or need any help! If there’s a specific topic you’d like me to cover let me know!
