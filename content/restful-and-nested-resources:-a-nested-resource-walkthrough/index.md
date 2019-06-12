---
title: "RESTful & Nested Resources: A Nested Resource Walkthrough"
description: "A disclosure: This is mostly about Nested Resources, but I think it’s important to start with a brief explanation of REST: In 2000, Roy Fielding introduced the idea of Representational State Transfer…"
date: "2016-05-09T18:56:52.717Z"
categories: 
  - Rails
  - Ruby on Rails
  - Ruby

published: true
canonical_link: https://medium.com/joelachance/restful-nested-resources-a-nested-resource-walkthrough-dad1f13ea54
redirect_from:
  - /restful-nested-resources-a-nested-resource-walkthrough-dad1f13ea54
---

A disclosure: This is mostly about Nested Resources, but I think it’s important to start with a brief explanation of REST:

In 2000, Roy Fielding introduced the idea of Representational State Transfer (REST) in his doctoral dissertation. The idea being, any websites’ URI’s would describe the content of that page. For example, your author’s posts with your author having a primary id of 1:

```
booksarecool.com/authors/1/posts
```

instead of the more enigmatic:

```
booksarecool.com/authors?=1posts234908234
```

I should mention that URI’s aren’t inherently RESTful themselves, it has to do with in what _context to your HTML Verb_ the URI is used. For example, your ‘/index’ URI shouldn’t be a POST request, it should be a GET. Please see the two articles better explaining this at the bottom of this post.

On to nested resources! What does REST have to do with nested resources, you say? Why do _you_ care? Let’s define a nested resource before we continue:

> A resource that is a child of another resource in your application.

What? Let me explain by starting with your associations, and let’s use an artist/song relationship. Your artist ‘has\_many’ songs, and your songs ‘belong\_to’ your artist. _Bad Blood_ belongs to Taylor Swift, not the other way around. You might even say your song is a _child_ of your artist!

As a software developer, you’ve been called to not annoy your peers with repeated code, so, DRY (Don’t Repeat Yourself)! If you’re using a ‘resources’ macro to define your models’ routes, you may be repeating routes that should be nested. Remember, you want to illustrate your parent/child relationship, even within your routes. With the use of nested resources, _Bad Blood_ always belongs to Tay, no one else gets to hold ownership on that song. Consider the following:

```
#config/routes.rb

resources :artists
resources :songs

#each of these gives you the 7 RESTful routes: [:index, :create, :new, :edit, :show, :update, :destroy]
```

Now, some of these actions we want to do within the scope of your artist. Let’s say we want to create, edit, update, destroy a song on it’s own, and not worry our artist over those, who is probably more concerned about the lack of [brown M&M’s](http://www.npr.org/sections/therecord/2012/02/14/146880432/the-truth-about-van-halen-and-those-brown-m-ms) in their green room. Let’s re-define our routes:

```
#config/routes.rb

resources :artists, only: [show] do
  resources :songs, only: [:index, :show]
end

resources :songs, only: [:create, :new, :edit, :update, :destroy]
```

Perfect. Our :index and :show routes for :songs are now nested (Note the do end syntax here, denoting our nested resource). Let’s head over to our controller and make some changes:

```
#controllers/songs_controller.rb

def index
  @songs = Song.all
end

def show
  @artist = Artist.find(params[:artist_id])
  @song = Song.find(params[:id])
end
```

Note: :artist\_id is coming from our nested route. Since your URI now has two dynamic :id calls (www.yaymusic.com/artists/**:id**/songs/**:id**), you need a way to differentiate. Rails gives you ‘parent\_id’ so ‘params\[:id\]’ doesn’t interfere. Also note, you now need to tweak your views to include both id’s:

```
<%= link_to 'Edit Song', edit_artist_song_path(artist_id: @artist, id: @song) %>
```

P.S. Don’t [nest resources more than a single level](http://weblog.jamisbuck.org/2007/2/5/nesting-resources). It’s going to start to get complicated, messy, and confusing!

[**Give me a example of non-RESTful design?**
_Stack Overflow is a community of 4.7 million programmers, just like you, helping each other. Join them; it only takes a…_stackoverflow.com](http://stackoverflow.com/questions/3889099/give-me-a-example-of-non-restful-design "http://stackoverflow.com/questions/3889099/give-me-a-example-of-non-restful-design")[](http://stackoverflow.com/questions/3889099/give-me-a-example-of-non-restful-design)

[**Why Some Web APIs Are Not RESTful and What Can Be Done About It**
_Many Web API designers claim their are RESTful, but their APIs have little in common with REST. What can be done to…_www.infoq.com](http://www.infoq.com/articles/web-api-rest "http://www.infoq.com/articles/web-api-rest")[](http://www.infoq.com/articles/web-api-rest)
