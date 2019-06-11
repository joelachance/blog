---
title: "Breakdown: AngularJS Custom Directives"
description: "To a newcomer, Angular seems almost magical. It’s the framework you never knew you needed: easy binding and iteration right in your DOM: Keep in mind this isn’t responsive — depending on if you were…"
date: "2016-06-12T14:06:39.476Z"
categories: 
  - Angularjs
  - JavaScript

published: true
canonical_link: https://medium.com/joelachance/breakdown-angularjs-custom-directives-4a041a25b6d3
redirect_from:
  - /breakdown-angularjs-custom-directives-4a041a25b6d3
---

To a newcomer, Angular seems almost magical. It’s the framework you never knew you needed: easy binding and iteration right in your DOM:

```
// app/js/app/directives/TabDirective.js
var tabs = [Tab 1, Tab 2, Tab 3];

// index.html
<div ng-repeat=”tab in tabs”>
  <p>{{ tab }}</p>
</div>
```

rendered, gives you:

```
// index.html
Tab 1
Tab 2
Tab 3
```

If you did this in Rails sans Angular, this would require…a few more lines:

```
# index.html
<% @tabs.each do |tab| %>
  <p><%= tab %></p>
<% end %>

// app/js/website.js
$('p').click(function() {
  $.get('/tabs', function(response) {
    $('p').html(response["tab"]);
  };
});
```

Keep in mind this isn’t responsive — depending on if you were to change your tabs, jQuery doesn’t figure out you’ve persisted any data unless you’ve made an AJAX call that’s attached to an eventListener that depends on your user clicking the correct…. you get the idea. Angular updates the attributes you need when they change. Nothing else. Pretty awesome, right?!

---

So, custom directives. You have a very basic taste of why you would use Angular, and to ‘append’ my previous explanation (awful jQuery joke), you can give your DOM custom elements, which are called directives! They might look like:

```
<custom-directive>
  //your very important data goes here
</custom-directive>
```

‘Custom, you say? There must be built-in directives then…’ Yes. There are. Here are a few:

```
ng-click
ng-bind
ng-app
ng-controller
ng-model
ng-form......
```

These are pretty great, but custom stuff is all the rage these days. And, they aren’t that complicated! With a small caveat: there can be a lot of associated properties to a custom directive if you need them. Let’s walk through these properties, and build a custom directive.

#### Your Basic Directive

```
function BasicDirective() {
  return {
    // your directive properties go here!!
  };
}

angular.module('app').directive('basicDirective', BasicDirective)
```

Above you can see some boilerplate code that is the beginning of your custom directive. Please note the last line: this is telling angular this files’ associations (this belongs to the ‘app’ module), and what this file is (a directive!). If you need to refer to this directive, you’ll be using ‘basicDirective’. Easy!

#### Property #1: Template

This is what is injected into the DOM when your custom directive is called. This needs to be a string, but you can do some fancy JS to make a larger template more readable with ‘.join(‘’)’.

```
template: [
 ‘<div class=”tabs”>’,
  ‘<ul class=”tabs_list”>’,
   ‘<li ng-repeat=”tab in tabs.tabs”>’,
    ‘<a href=”” ng-bind=”tab.label” ng-       click=”tabs.selectTab($index);”></a>’,
   ‘</li>’,
  ‘</ul>’,
 ‘</div>’
 ].join(‘’)
```

#### Property #2: Restrict

The restrict property tells your custom directive how things should be rendered. Here’s a basic list and what they look like:

\`A\` — used as an attribute  
\`E\` — used as an element  
\`C\` — used as a class name  
\`M\` — used as a comment

By default restrict is set to ‘EA’, which assigns your directive to an element and an attribute.

```
Attribute — <div my-directive></div>
Element — <my-directive></my-directive>
Class: <div class=”my-directive”></div>
Comment — <! — directive: my-directive →
```

You’ll assign your restrict property like so:

```
restrict: ‘E’
```

#### Property #3: Controller

Controllers contain the business logic for your directive. You want to keep your controllers slim — any robust logic should be outsourced to an Angular Service and injected into your controller when it’s instantiated in your view. Along with business logic, a property called ‘$scope’ is also instantiated when your controller is created. You can then attach properties onto your controller’s scope to be used later. Think of using your ‘$scope’ similar to a JavaScript constructor.

```
index.html
<div ng-controller="ourController">
  {{ hello }}
</div>

OurController.js
function OurController($scope) {
  $scope.hello = alert('Hello!');
}
```

In the above example, when ‘ng-controller’ is called in our view, an instance of OurController is created along with ‘$scope’. We attached our ‘hello’ method to our $scope. That way, we’re able to bind ‘hello’ to our DOM!

#### Property #4: controllerAs

You may have noticed above example wasn’t using a directive — we created a standalone controller and instantiated it in our DOM. What if we want to nest elements in our DOM and they’re using different controllers? Our controllers’ scope gets confusing, which is where controllerAs comes in.

```
index.html
<div ng-controller="firstController as first">
  {{ first.hello }}
  <div ng-controller="secondController as second">
    {{ second.hello }}
  </div>
</div>

Controller.js
function FirstController() {
  this.hello = alert('Hello!');
}
```

Notice two things: we’ve now assigned a variable name to our controllers in our view, and we’re using that to differentiate our ‘hello’ method calls. This gives us a way to simplify complicated elements in our DOM. Secondly, we now use ‘this’ instead of ‘$scope’ in our controllers. Think of using ‘this’ as referring to that instance of ‘$scope’. To use inside a directive, use:

```
controllerAs: ‘main’
```

This allows us to call our controller without ‘ng-controller’.

#### Property #5: Scope

This last one is a simple concept, yet necessary, especially when you’re trying to isolate your scope. The items you’ll place inside of your scope bind to the attributes in your DOM elements, giving you access to their values, like so:

```
scope: {
  name: '='
}
```

This is saying you can now use the value from this element inside of your directive:

```
<div name="Joe"></div>
```

Note: the ‘=’ syntax is for when your attributes’ name is the same name you want to bind inside of your scope and use in your directive.

---

Note: There are obviously other properties I’ve purposely left out to keep this post simple. Other important notable properties:

```
bindToController
transclude
replace
require
link
```
