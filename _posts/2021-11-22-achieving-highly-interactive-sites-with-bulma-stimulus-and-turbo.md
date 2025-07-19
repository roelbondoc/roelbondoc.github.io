---
layout: post
title: Achieving Highly Interactive Sites with Bulma, Stimulus, and Turbo
description: Build highly interactive web applications with minimal frontend knowledge by combining Bulma CSS, Stimulus, and Turbo with Ruby on Rails.
---

Part of being a Fullstack Developer means building features that touch every layer of your application stack. If you’re like me, with a background and preference for server side development, you may not have strong front end development skills. To overcome these shortcomings, it’s worthwhile to leverage pre-built frameworks that compress the underlying knowledge needed to create highly interactive features.

In this post, I’ll show how to integrate a few front end frameworks together to achieve some nice and simple user interface components. Using CSS and Javascript frameworks, together with Rails, allows you to focus on delivering robust user interfaces with less foundational knowledge of front end development. I do recommend that if front end development interests you, that you invest some time in learning more foundational theory. However, that would beyond the scope of this post and beyond the limits of my ability. So for now, this post is going to focus on getting the most efficiently and effectively as possible with as little as possible.

## Bulma
Bulma is a highly opinionated CSS framework. If you haven’t used any CSS frameworks before, this is a good one to get started with. It has detailed documentation with easy “copy and paste” examples. There are tons of other great CSS frameworks out there, but I choose to use Bulma in this post since it’s what I’m most familiar with.

Bulma provides you with a set of CSS classes and philosophy that defines both a good structure of content and style. By following the design system, you can be sure that you’ll have something that’s easily maintainable, accessible, and visually appealing. 

## Stimulus
Stimulus is a Javascript framework that ships easily with Rails. Self-described as a “modest” framework, it strives to provide functionality by augmenting your existing HTML (or ERB view templates). Stimulus does a great job separating out your JavaScript code from your views. Instead you decorate your HTML with special stimulus specific data attributes which activate the stimulus controller functionality.

If you’ve struggled with many of the popular Javascript frameworks like I have, you may also find Stimulus to be a breath of fresh air. It’s simple API and conventions make it quick to learn. I found that it provides a great foundation in adding interactive elements to your site.

## Turbo
If you’ve used TuboLinks in the past, you can think of Turbo as the next generation. Turbo integrates with Rails, via the `turbo-rails`  gem, that helps make AJAX a thing of the past.

Turbo lets you create user interactions that rely on server calls without having to write a single line of Javascript. In my view, it’s the sort of thing you end up building after wrangling traditional Javascript frameworks for far too long.

## Use Case Examples
The best way to learn something is by building out real world use case scenarios. Bulma doesn’t come with any Javascript libraries. Instead it gives you the CSS framework that you can augment by writing  your own Javascript. By combining Bulma, Stimulus, and Turbo together, you’ll be able to deliver rich, highly interactive, and beautiful user experiences. Here are a few common examples that you might find useful.

### Opening and Closing Modals
A very useful UI design pattern is a modal. Modals provide a good way to show more information to a user without having to navigate away from what they are currently seeing. The modal implementation in Bulma is great, and works well with a few lines of Stimulus. Here’s what a simple modal markup looks like.

```
<div data-controller=“modal”>
  <button class=“button” data-action=“modal#open”>
    Open Modal
  </button>
  <div class="modal" data-modal-target=“modal”>
    <div class="modal-background"></div>
    <div class="modal-content">
      Content goes here.
    </div>
    <button class="modal-close is-large"
            aria-label="close"
            data-action=“modal#close”></button>
  </div>
</div>
```

You’ll notice that attached is the Stimulus controller called `modal`. This controller looks like this:

```
// modal_controller.js
import { Controller } from "stimulus";

export default class extends Controller {
  static targets = ["modal”];

  open() {
    this.modalTarget.classList.add("is-active");
  }

  close() {
    this.modalTarget.classList.remove("is-active");
  }
}
```

The attribute `data-controller=“modal”` tells stimulus to bind the `modal_controller.js` to the “scope” of this element. A button to launch the modal contains the `data-action=“modal#open”` attribute, and a button to close the modal has `data-action=“modal#close”`. These attributes map the `click` event to the `open()` and `close()` functions respectively. In the stimulus controller, the modal is opened and closed by adding and removing the `is-active` class provided by the Bulma framework.

### Progress Bar Increment
Web application progress bars have always been a bit tricky to implement, but are very useful for long running operations. If implemented carefully, they can provide a good way to convey how long a user has to wait for something to finish. In the past, a popular tactic would be a pooling mechanism which would put more load on the server than necessary, in addition to writing unsightly Javascript. But taking advantage of TurboStreams and Rails’ ActionCable, a cleaner implementation is possible.

Using ERB, a user loads a page that will display the progress for a long running operation:

```
<%= turbo_stream_from @payroll %>
<div id=“progress-bar”>
  Please wait, running payroll.
  <progress class="progress"
            value="0"
            max="<%= @payroll.employees.count %>">0%</progress>
</div>
```

Then, on the server, a background task will perform various operations, and move the progress bar forward, whenever progress is made.

```
# payroll_job.rb
class PayrollJob < ApplicationJob
  def perform(payroll)
    payroll.employees.each_with_index do |employee, index|
      if PaymentMaker.pay(employee)
        payroll.update(completed: index + 1)
        payroll.broadcast_replace_to(
          payroll,
          target: 'progress-bar',
          partial: 'payrolls/progress_bar_update'
        )
      end
    end
  end
end
```

The `PaymentMaker.pay` call is a long running call and the code will broadcast each time an employee is paid. This broadcast will update the progress bar using TurboStreams. The `_progress_bar_update.html.erb` partial will look something like this:

```
<% if payroll.incomplete %>
  Please wait, running payroll.
  <progress class="progress"
            value="<%= payroll.completed %>"
            max="<%= payroll.employees.count %>">
              <%= (payroll.completed / payroll.employees.count * 100).round %>%
  </progress>
<% else %>
  Payroll run has been completed!
<% end %>
```

This Job is meant to continuously run for each employee on the payroll. In effect, this will communicate back to the user what the progress is of the payroll. Once all employees have been iterated through, the final progress update will be the completed message.

### Dynamic Search Panels
Updating content in a container after entering information from a form is a common pattern you’ll see in web applications. One such usage are search panels. By using turbo frames and a small stimulus controller, it’s easy to achieve this useful user interface.

First you’ll define a form that will gather your input from the user, along with a container to display your results:

```
<%= form_with url: results_path,
              data: {
                turbo_frame: ‘results’,
                controller: ‘auto-submit’,
                auto_submit_target: 'form'
              } do %>
  <%= text_field_tag :query, data: { action: 'auto-submit#submit' } %>
<% end %>
<%= turbo_frame_tag ‘results’ %>
```

The `action` data attribute will tell Stimulus to assign the `change` callback to the functional in the `auto-submit` controller (described later). The controller action that your path maps to can perform your normal ActiveRecord query, and then render a view. This view would look like this:

```
<%= turbo_frame_tag 'results' do %>
  <% @results.each do |result| %>
    <%= render result %>
  <% end %>
<% end %>
```

This will render each of the results found. The `turbo_frame_tag` will replace the empty one in the initial view above. So now, anytime the form is submitted, the results will display below the form. The next step is to add better usability by building out the Stimulus controller that will handle the auto submit functionality.

```
// auto_submit_controller.js
import { Controller } from "stimulus";
import Rails from "@rails/ujs";

export default class extends Controller {
  static targets = ["form”];

  submit() {
    Rails.fire(this.formTarget, "submit");
  }
}
```

This simple implementation will use the `Rails.fire` function to submit the form whenever the text field is updated. This function can later be updated to include a “debounce” so that unnecessary calls to the server are not made.

## Takeaways
Hopefully this article has given you a good introduction into easily building rich user interfaces. By putting together these front end frameworks, you can bring your web applications to life with less code. The less code you have the more maintainable your application will be, and the more efficient you will become.

Give it a shot and let me know how it goes or let me know what you think! If you want me to cover any more specific topics, feel free to reach out to me on Twitter!
