---
layout: post
title: "Integrating Sidekiq and Pusher in a Rails Application"
date: 2015-08-22 18:50:04 -0400
comments: true
categories: rails, javascript, ruby, sidekiq, pusher, ajax
---
##The Problem
When you're building an application that relies on multiple API calls and/or updating a database in bulk with a single user interaction it's crucial offload the work to a background job and serve a user a page as quickly as possible in order to maximize user experience. This was our dilemma for [WikiVice](http://wikivice.herokuapp.com/). We see these events all the time now, from YouTube's progress bar, to the loading circles in Facebook's mobile app. Surprisingly, me and my colleague [Heather Petrow](https://github.com/hpetrow), only found a single guide with information on integrating Sidekiq with Pusher written by [Karl Baum](http://cowjumpedoverthecommodore64.blogspot.com/2013/07/scaling-heavy-web-requests-with-sidekiq.html). The article served as a good jumping off point, but it didn't quite cover the needs of our application. 

For one, the examples in the article use CoffeeScript and we decided to stick with JQuery. Secondly, the guide's example is concerned with sending a json response to the client after the server finished the background job. However, we needed to load an ERB partial after background job finished. The guide covers the fundamentals of leveraging Sidekiq and Pusher, but doesn't account for more than one use case of a Rails application.

WikiVice is a rails app that display data visualizations for a Wikipedia article based on its revision history, in order to make the user question their assumptions about the world's most popular encylopedia. We designed our app around a single interaction: the search form. A user searches for a Wikipedia article, and the following page displays a detailed breakdown on the revision history of the article.

When a user submits a search the app sends several queries to [MediaWiki API](https://en.wikipedia.org/w/api.php). The first is for an article's title. We rely on the response from the API to judge whether a search is valid, in order provide form validation. Then we submit several requests to the revisions api until we get around 500 results. With the results in hand, we persist them to the datbase into the Page, Author and Revisions model.

The first prototype of the site took nearly 30 seconds to finish loeading, so improving speed was my primary responisbiltiy on the application. The easy solution involved reducing the amount of requests we send to Wikipedia. However, that comes at the cost of the quality of our dataset. My team and I agreed that 500 revisions per page was the baseline amount we needed to give the user an accurate depiction of the life a Wikipedia article. 

First, I reduced database transactions. I refactored all our models to use eager loading when possible, and found a gem called [activerecord-import](https://github.com/zdennis/activerecord-import) that allows you to bulk insert mulitple rows in database in one transaction. These efforts doubled the speed of the website. But the site still took about 10 seconds to load, and when we deployed to Heroku the site shut down because Heroku timed out the request.

##The Solution
That's when we realized we needed to use Background Jobs. We conceptualized the task as thus: When the user submits a search term, we call the Wikipedia API to validate the title. When we verify the title, we take the user to a page with the title in a heading, and a spinning circle. We want to replace the spinning circle with a dashboard once the background job has finished. It's almost like a reverse ajax call in that the server asynchronously sends a response to the client.

##Setting up Sidekiq
Setting up Sidekiq and Pusher is relatively simple, the challenge is getting the two things to talk one another. [Sidekiq](https://github.com/mperham/sidekiq) is a great little gem that enqueues Redis jobs with minimal setup. Create a class, include the ```Sidekiq::Worker module```, and write a ```#perform``` method like so: 
{% gist bd3de78fc58208f01d0d %}

``` wiki.get_page ``` is the method that sends requests to Wikipedia's API and persists the results in the database. In order to initiate the background job, go to your controller and call ```.perform_async``` on your worker class, e.g.
{% gist bb56ae3a125f9e1dfbe5 %}

In order for your application to start handling background jobs, you must start your redis server and sidekiq listener. Start your redis server with the terminal command ```redis-server```. Start sidekiq with ```bundle exec sidekiq```

It's important to send simple argument to Sidekiq jobs. When you run a background job, Sidekiq serializes the arguments into a JSON string and pushes that string into a Redis queue. The datatypes allowed include number, string, boolean, array, and hash. Therefore, if you want a background job to act on an object, I advise passing a object's id. Once you have the id, you can find the object with an ActiveRecord method inside the perform method.

##Debugging Sidekiq
Debugging Sidekiq can be a little annoying because it involves 3 running process, your rails server, redis server, and sidekiq listener. Whenever you make a change to code in your Worker, I highly advise you to reset all three (rails server, redis server, and sidekiq listener). If you do not reset all three, you can't be sure if you have a background process in queue or not. Also, if you make a change to your worker code, It won't take effect until you reset Sidekiq.

##Pusher
How do you know when the job is finished? This is where [Pusher](https://pusher.com/) comes in to play. Pusher is hosted API that simplifies WebSockets in order to provide bi-directional communiciation between a client and a server, or clients and other clients e.g. chat. The most important step to enabling Pusher is setting up channel. You can call a channel whatever you want, as long as its consistent betwen the server and client. It's best practice to keep the channel names unique to the page or user, because more than one client can use the same channel. 

## Pusher -- Rails
Let's start with setting up our server-side code. This is the easy part. Step one: initialize a Pusher object. Step two, trigger a pusher event to the channel.
{% gist fab0033f00eada0ea437 %}

Line 10 involves the usual method of setting up a client that interacts with a 3rd party api. You have to initialize it with your api key info in order to start a connection to Pusher's web sockets api.

line 17 is where the magic happens. "#trigger" executes the callback function "get_page", passing the hash as a json object, on the page results channel.

## Pusher -- JavaScript
The key event here is ```channel.bind('subscription:succeeded', function() {})```. You will notice that the following code binds an event to another event. I have not seen other tutorials use this implementation, but that might be because they don't cover this particular implementation. 'subscription:succeeded' is a special event that fires when a client has successfully registered a subscription with Pusher. By nesting events within 'subscription:succeeded', you ensure that a client event triggers.

 However, in order to use Client Events, you must update your Pusher account settings. Go to Settings, check 'Enable Client Events', and click 'Update'. You must enable these setting for the following code to work.

{% gist bbba5f5ce6d7ef935415 %}

Lets take a closer look at everything that's happening here. Line 1 includes the source for the pusher js library that sets up the channel. Lines 4 through 7 provide debugging info for Pusher. I highly recommend including the debugger script as it outputs detailed info about the connection state of pusher events.

Line 10, establishes the page id, taken from a hidden html element on the page.

Lines 12-15 initialize the Pusher object, and subscribe to a unique page channel. 

Lines 18 and 19 are crucial. Most pusher tutorials explain that you simply need to bind a callback function to your channel in order to get bi-directional communication started. However, when you're using pusher in connection with a time intensive background job, you need to be careful about the sequence of connections.

You must wait to bind the channel to the event after the channel has been successfully subscribed to. Otherwise, your JavaScript will never receive the message from the server. Binding an event to 'pusher:subscription_succeeded' ensures that future bindings will take place once a channel has been successuflly subscribed to between a client and the server. 

Pusher has a really great debugging console that allows you to monitor connections to the Pusher API. Looking at the console, you want to make sure your connections follow this sequence: Connection, Subscribed, API message. Here's a screen shot of my console for help: 
{% img /images/pusher_console.png %}

 In my example, we added ```pusher.unsubscribe"page_results"+id);``` in order to close the channel after the background job finished.

The most important thing to remember here is to that you want to bind the callback AFTER the channel has been successfully subscribed to. Once the background job subscribes to the channel, it will execute the binded callback function. 

##Rendering an ERB partial with Pusher

In our case, we don't really need to pass data between Sidekiq and Pusher, receiving the data is simply a signal that we're ready to load the ERB template with data visualizations. The dashboard is dependent on the page having a revision history, so we do not want to load the template before the background job finishes.

In order to render an ERB partial in conjunction with Pusher, we simply made AJAX request to separate controller action responsible for rendering a partial asynchronously. We added a GET /dashboard route and mapped it to Page#dashboard. When an AJAX request hits /dashboard, the Pages controller responds by executing "dashboard.js.erb", as is convention with AJAX requests in rails.

In order to take advantage of this convention, you must send an AJAX request with a dataType of "script".

Lastly, "dashboard.js.erb" is responsible for replacing the spinner gif with the dashboard partial: 

{% gist 80476c2743227fcfa992 %}

If we called the AJAX request to ```/dashboard``` before the push notification, nothing would render because the Sidekiq is in the middle of processing the background job. We're simply using Pusher as a timer for when to fire the AJAX request. 

## Conclusion

In conclusion, Sidekiq and Pusher are a powerful tool for maximizing the user experience when you need to crunch a lot of data in single request cycle. In the future, I'd like to develop the progress indicator. Right now, we simply have a spinner gif that displays until the erb template gets injected. However, I'd like to implement a progress bar that indicates what's actually happening behind the scenes for a more informed user experience. Sometimes, the request hangs up, so it would be nice to let the user know when they need to refresh the page, instead of having the user stare at a spinning circle forever.