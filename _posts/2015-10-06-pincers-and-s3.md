---
author: amosrivera
layout: post
title: How To: From Pincers to Amazon S3
excerpt: One of our latest crawlers required the ability to extract files from a website and upload them to Amazon Web Services. Here we will explain how we did it.
tags:
    - pincers
    - aws
    - download
---

## Introduction

One of our latest crawlers required the ability to extract files from a website and upload them to Amazon Storage Services. Here we will explain how we did that.

For the sake of simplicity let's imagine you want to download a different space image everyday from the NASA website.

If you're in a hurry [here is the complete script](https://gist.github.com/amosrivera/6ebb695f219df21f2784).

## Browsing

First, browse to the section conveniently named _Image of the Day_.

```rb
pincers.goto "www.nasa.gov"

menu = pincers.search("#nasa-main-menu > li:contains('Galleries')")
menu.search("a:first").hover
menu.search("a:contains('Image of the Day')").click
```

It's also possible to `pincers.goto` directly to that page but why miss the opportunity to use the [recently added pseudo-selectors](http://blog.crabfarm.io/implementing-jquery-pseudo-selectors)?.

<small>Alternative:</small>

```rb
 pincers.goto "https://www.nasa.gov/multimedia/imagegallery/iotd.html"
```

## Downloading

Second, click on the first image. This will trigger a modal box which contains a link to download a full resolution version.

```rb
pincers.search('.gallery-card:first .inner').click
```

Since the link is not present before the modal is shown Pincers [waits](https://github.com/platanus/pincers#waiting-for-a-condition) until it finds the element. You don't have to worry about waiting for animations and such just search for that link and use `download`.

```rb
file = pincers.search("#full-view a[download]").download
```

The `download` method saves the image to memory. It's that simple and it can download all kinds of files as long as the element has an `href` or `src` attribute. If you wanted to store it to disk now you could use the `store` method.

```rb
file.store("image.png")
```

If you run the code as it is now. You will find the latest image of the day in the same folder where the script is.

## Uploading

But instead we want to upload to AWS.

```rb
s3 = Aws::S3::Resource.new(
  credentials: Aws::Credentials.new('key_id', 'key_secret'),
  region: 'us-west-1'
)

object = s3.bucket('bucket_name').object('image.png')
object.put(body: file)
object.public_url
```
The uploading process is taken care of by the `aws-sdk` gem. You just have to pass the binary data or a file path. Read more about it in [it's documentation](http://docs.aws.amazon.com/sdkforruby/api/index.html).

Putting it all together it looks [like this](https://gist.github.com/amosrivera/6ebb695f219df21f2784). As you can see it is a small and very idiomatic script. That natural feel is what we're looking for with Pincers. If you have any suggestions or issues refer to the [Github Repository](https://github.com/platanus/pincers).
