---
author: amosrivera
layout: post
title: Hassle-free Web Automation with Pincers
tags:
    - pincers
    - nokogiri
    - webdriver
---

At Platanus we're working on [crawlers everyday](https://github.com/platanus/crabfarm-gem) and from time to time a tool becomes interesting enough to be open sourced. Recently we decided to take a shot at improving the interaction with web automation tools and came up with [Pincers](http://github.com/platanus/pincers).

We needed an agnostic interface to control underlying web tools like Webdriver, Nokogiri, Mechanize, etc. and parse the DOM without having to go back to the documentation to copy and paste code or find how that particular tool did it.

It needed to feel natural. Finding elements, setting values, triggering events, setting the viewport sizes, etc.

```ruby
pincers.goto "www.couchsurfing.com"
pincers.search(href:'/users/sign_in').click

pincers.search(name:'user[login]').set 'user@example.com'
pincers.search(name:'user[password]').set '12345'
pincers.search(value:'Log In').click

puts pincers.url
```

That's what entering into the couchsurfing dashboard looks like. If that got your attention read on because it has some sweet features.

## Installation and Usage

Install Pincers and [Webdriver](https://rubygems.org/gems/selenium-webdriver):
```
gem install selenium-webdriver
gem install pincers
```

Next initialize it in your script with your driver of choice (by default it's `:firefox`):
```ruby
require 'pincers'

pincers = Pincers.for_webdriver :chrome # :firefox, :phantomjs, :chrome
```

## Selecting elements

Pincers offers two method to obtain elements from the DOM. First there is `search` (seen in the example above) which takes key-value pairs to filter elements by their attributes and then there is `css`  

Let's say we wanted to know what's [trending on rottentomatoes](http://i.imgur.com/oxO6sOA.png):

```rb
pincers.goto "www.rottentomatoes.com"
trending = pincers.css(".trendingEl li:not(.header) a")

trending.each do |link|
  puts link.text
  puts link.attribute('href') # same as link[:href]
end
```

`css` will return a single element or an enumerable object depending what the selector matched. Once returned you have [different methods available](https://github.com/platanus/pincers#first-element-properties) to get the properties.

Just scraping is not that fun though, let's move on.

## Interaction and events

This is about automatization. Which means being able to interact with the page like a regular user. Click, hover, change input values, submit forms, etc.

Say we wanted to know the status of a package sent through the Chilean post office. A user would go to the site, write the package number in the textarea and click the submit button.

Now imagine this had to be done a hundred times. Pincers has got you covered in a few lines of code.

```ruby
# submit form
pincers.goto "www.correos.cl"
pincers.search(name: 'envio').set "RT257532695HK"
pincers.css("#btnseguimiento").click
```

This site has a catch though. The package status is rendered in a specific `iframe`.

```ruby
pincers.goto frame: pincers.css("#ifSeguimiento")
```

No problem. Passing a `frame:` property to the `goto` method changes Pincers' context to that frame. And then just parse the goods.

```ruby
status = pincers.css(".tracking tr:not(:first-child)")
status.each do |row|
  puts row.text
end
```

Gives us..
```
ENVIO ENTREGADO   16-06-2015 12:05     SANTIAGO CDP 03
ENVIO EN REPARTO   16-06-2015 8:09     SANTIAGO CDP 03
...
```
_<small>That's spanish for 'delivered'</small>_

Too simple? How about a list of Burger King restaurants for a particular location.

```ruby
pincers.goto "www.bk.com/locations"
pincers.search(name:'locationSearch').set "Seattle"
pincers.css(".locInput a.submit").click

locations = pincers.css("#locationsList .location")
```

Goes to the site, fills the location input and clicks search. You've seen that before but there is a detail. The page has already loaded and the element we're looking for is not there because it's waiting for the ajax response.

It's cool just tell Pincers to keep looking for it (with a time limit).

```ruby
locations.wait(:present, timeout: 30).each do |row|
  puts row.css(".mainAddress").text
end
```

Outputs:

```
2021 RAINIER AVENUE, SOUTH
3301 FOURTH AVENUE SOUTH
11723 NE 8TH. STREET
4015 FACTORIA BLVD SE
```

Still unfazed? Maybe something more elaborate like obtaining the lowest fare from Delta for a specific city on a specific date range. remember to use a browser as the driver to see the interactions happening:

```ruby
pincers.goto "http://www.delta.com/"

pincers.css("#originCity").set "SCL" # Santiago
pincers.css("#destinationCity").set "SDQ" # Dominican Republic

pincers.css("#departureDate").set "11/16/2015"
pincers.css("#returnDate").set "11/30/2015"

pincers.css(".delta2UpCal.end .calClose").click # close date picker
sleep(1) # wait for date picker animation

pincers.css("#flexDaysBtn").click # flexible dates
pincers.css("#findFlightsSubmit").click

lowest = pincers.css(".lowestFare").wait(:present, timeout: 30)
puts lowest.first.text
```

Outputs
```
$1,190.30
```

Once you have crawled the data it's up to you to find something useful to do with it! Check out [the docs](http://github.com/platanus/pincers) and take it for a spin.

This is a tool that we think has a future, part of an ongoing process to develop our crawling ecosystem which we're actively working on. If have any suggestion or [issues](https://github.com/platanus/pincers/issues/new) using Pincers let us know.
