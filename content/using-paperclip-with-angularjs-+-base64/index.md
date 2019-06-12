---
title: "Using Paperclip with AngularJS + Base64"
description: "When I started to work through AngularJS, I started to realize how many gems aren’t abstracted the same way they are in Rails. As I was working through Angulargram, I realized how some of the…"
date: "2016-07-09T02:18:18.950Z"
categories:
  - Angularjs
  - Rails
  - Ruby
  - JavaScript

published: true
canonical_link: https://medium.com/joelachance/using-paperclip-with-angularjs-base64-cf497a248be2
redirect_from:
  - /using-paperclip-with-angularjs-base64-cf497a248be2
---

When I started to work through AngularJS, I started to realize how many gems aren’t abstracted the same way they are in Rails. As I was working through [Angulargram](http://angulargram.herokuapp.com/#/), I realized how some of the features I assumed would be easy…weren’t. So, let’s make photo uploads with Angular + Rails easy.

```
rails new ng-file-upload
```

Let’s start by creating a new Rails project. I’ve also created a repo if you want to follow along: [https://github.com/fiveinfinity/ng-file-upload](https://github.com/fiveinfinity/ng-file-upload)

```
gem ‘paperclip’
gem ‘bower-rails’
gem 'angular-rails-templates'
gem 'active_model_serializers'
```

Include gems, and…

```
bundle
```

Let’s get bower going:

```
rails g bower_rails:initialize json
```

And include in your bower.json file:

```
“angular”: “v1.5.3”,
“angular-ui-router”: “latest”,
“angular-base64-upload”: “latest”
```

This is going to install all of your bower dependencies you just added above. Note the last dependency — this one is important, it’s going to convert your photo file into base64 in order for Paperclip to read it properly in Rails.

```
rake bower:install
```

A couple more housekeeping steps to get Angular + Rails up and running:

```
#config/routes
‘root ‘application#index’

#application.html.erb
<body ng-app=”app”>

#application_controller.rb
def index
end

#application/index.html.erb
<div ui-view></div>
```

I’m not going to go into what the above code does as it’s not the focus of this post, but it’s all very standard Rails layout. The ng-app in our application view and our ui-view in our index view is how we’re able to render all of our Angular in our root directory.

Some more dependencies in our asset pipeline:

```
//app/assets/javascripts/application.js

//= require angular
//= require angular-ui-router
//= require angular-rails-templates
//= require angular-base64-upload
```

At this point, we have Angular almost working and a basic framework in Rails. Now for the meat and potatoes. Let’s get some models set up in Rails:

```
rails g model post title
rails g paperclip post photo
```

And run

```
rake db:migrate
```

This is going to give you the following schema. Note what Paperclip is adding to your Post model:

```
create_table “posts”, force: :cascade do |t|
 t.string “title”
 t.datetime “created_at”, null: false
 t.datetime “updated_at”, null: false
 t.string “photo_file_name”
 t.string “photo_content_type”
 t.integer “photo_file_size”
 t.datetime “photo_updated_at”
 end
```

Let’s get our Post model and Serializer going:

```
#post.rb
class Post < ActiveRecord::Base
 has_attached_file :photo, styles: {original: “700x700>”}
 validates_attachment_content_type :photo, content_type:     /\Aimage\/.*\Z/
end

#post_serializer.rb
class PostSerializer < ActiveModel::Serializer
 attributes :id, :title, :photo
end
```

Ok, we have one more Rails step to work on (Post Controller!), but we’re going to save that for last. Let’s get our Angular setup and working so we can actually upload our photo from the JS side. Create the following file:

```
app/assets/javascripts/app/app.js
```

This is going to create your ‘app’ module file as well as your app.js module file. Here’s what goes inside:

```
angular
 .module(‘app’, [‘ui.router’, ‘templates’, ‘naif.base64’])
 .config(function($stateProvider) {
 $stateProvider
 .state(‘home’, {
 url: ‘/’,
 templateUrl: ‘home.html’,
 controller: ‘HomeController as home’
 });
 });
```

A couple things here: note we’re injecting ui.router for our Angular routing This isn’t necessary for our app as you could just as easily use RouteProvider if you wanted, however, it’s my preference. More importantly, note the ‘naif.base64’ injection. This is what is required by our ‘angular-base64-upload’ library so Angular can parse your sweet photo into base64 and send it to Rails. Notice your terminal when you upload — you’ll get to see base64 in all of its glory. Two more files to finish off the Angular side of our application:

```
//HomeController.js
function HomeController($scope, $http) {
 var ctrl = this;
 ctrl.post = {};

ctrl.submit = function() {
 $http.post(‘/posts.json’, ctrl.post).then(function(res) {
 ctrl.upload = res.data.photo;
 });
 }
}

angular
 .module(‘app’)
 .controller(‘HomeController’, HomeController);

//app/templates/home.html
<form ng-submit="home.submit()">
  <input type="file" ng-model="home.post.image" accept="image/*" base-sixty-four-input>
  <input type="submit" value="submit">
</form>

<img src="{{ home.upload }}">
```

Here we have our controller for our home view and the view itself. Normally in a more robust app you’ll have your $http calls inside of a service, but for simplicity and the brevity of this app I’m keeping it inside of the controller. Below is what our $scope.image is returning when we submit a photo:

```
$scope.image = Object {filetype: “image/jpeg”, filename: “pic9.jpg”, filesize: 1520767, base64: “/9j/4AAQSkZJRgABAgEAtAC0AAD/4RVYRXhpZgAASUkqAAgAAA…vtsJcXZnLwxxAsnYwOsQfqJmbGsAQp9v+mGyqQoJs0Wf/2Q==”}
```

You’re probably thinking, ‘Ok, great. What the hell do I do with a base64 attribute? None of this was in our schema, we can’t persist this!’ Let’s head (finally!) to our Rails Post Controller and tie everything together (including that base64 attribute!):

```
class PostsController < ApplicationController
 skip_before_filter :verify_authenticity_token
 respond_to :json

 def create
  @post = Post.create!(title: ‘created title!’, photo: decode_base64)
  respond_to do |f|
  f.json { render :json => @post }
  end
 end

 def decode_base64
  decoded_data = Base64.decode64(params[:image][:base64])
  data = StringIO.new(decoded_data)
 data
 end
end
```

As it’s probably obvious at this point, the magic is really happening in our ‘decode\_base64’ method. With our ‘angular-base64-upload’ library we’re given the ‘decode64’ method that can parse your photo into a string that Paperclip can read. Once you upload your photo from Angular, you’ll see it load below in the same window.

Again, feel free to fork [this repo](https://github.com/fiveinfinity/ng-file-upload) to follow along and play with Angular, Rails, Paperclip, and Base64.
