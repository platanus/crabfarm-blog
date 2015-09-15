---
author: amosrivera
layout: post
title: Pincers now supports jQuery pseudo-selectors
tags:
    - pincers
    - nokogiri
---

We're happy to announce that [Pincers](https://github.com/platanus/pincers) now supports element selectors like `:has`, `:contains`, `:first`, `:disabled` and so on. If you're not in the loop Pincers is a web automation tool designed to be as easy to use as possible (check out our [previous blog post](blog.crabfarm.io/hassle-free-web-automation-with-pincers)).

Pseudo-selectors are keywords used to specify the state of an element. They are cleaner and much more readable than their verbose alternatives. For example the selector `:checkbox` returns all `<input>` elements of type checkbox instead of using `input[type='checkbox']`.

#### What's new?

We are are big fans of jQuery so we figured it would be awesome to support all of it's pseudo-selectors. Which basically means that _if you know jQuery you know Pincers_.

Here is a brief example. Say we wanted to obtain links related to José Mourinho from BBC's sport section.

```ruby
pincers.goto "http://www.bbc.com/sport/0/"

headlines = pincers.search("a:contains('Mourinho')")

headlines.each do |headline|
  puts headline.text
  puts headline.attribute("href")
end
```

We're betting on Mourinho's controversial personality to keep him in the front page (and thus this example working) but feel free to change the example to anything you see fit.

For a list of all the available selectors and options read the [documentation](https://github.com/platanus/pincers/wiki).

#### What's next?

Now that selectors are in the bag we're taking a look at DOM traversing methods that will allow Pincers to move through the DOM tree from a selected node.

Element selection and interaction is probably where people put most of the time when creating bots and crawlers. That's where Pincers does a stellar job. If you have any suggestion or problems submit an issue [on Github](https://github.com/platanus/pincers/issues).
