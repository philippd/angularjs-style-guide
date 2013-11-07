#Introduction

The goal of this style guide is to present a set of best practices and style guidelines for a AngularJS application.

In this style guide you won't find common guidelines for JavaScript development. For this we use the [Google's JavaScript style guide](http://google-styleguide.googlecode.com/svn/trunk/javascriptguide.xml).

This guide was originally forked from https://github.com/mgechev/angularjs-style-guide and was adapted to our needs. It contains additional structuring suggestions and best practices from:
* https://github.com/ngbp/ng-boilerplate
* http://cliffmeyers.com/blog/2013/4/21/code-organization-angularjs-javascript
* http://trochette.github.io/Angular-Design-Patterns-Best-Practices

# General 

* Use:
    * `$timeout` instead of `setTimeout`
    * `$window` instead of `window`
    * `$document` instead of `document`
    * `$http` instead of `$.ajax`

This will make your testing easier and in some cases prevent unexpected behaviour (for example, if you missed `$scope.$apply` in `setTimeout`).

* Use promises (`$q`) instead of callbacks. It will make your code look more elegant and clean, and save you from callback hell.
* Use [restangular](https://github.com/mgonto/restangular) instead of `$http` when possible. Higher level of abstraction saves you from redundancy.
* Don't use globals. Resolve all dependencies using Dependency Injection.
* Do not pollute your `$scope`. Only add functions and variables that are being used in the templates.
* Prefer the usage of [controllers instead of `ngInit`](https://github.com/angular/angular.js/pull/4366/files). The only appropriate use of `ngInit` is for aliasing special properties of `ngRepeat`. Besides this case, you should use controllers rather than `ngInit` to initialize values on a scope.
* Do not use `$` prefix for the names of variables, properties and methods. This prefix is reserved for AngularJS usage.

## Optimize the digest cycle

* Watch only the most vital variables (for example: when using real-time communication, don't cause a digest loop in each received message).
* Make computations in `$watch`  as simple as possible. Making heavy and slow computations in a single `$watch` will slow down the whole application (the $digest loop is done in a single thread because of the single-threaded nature of JavaScript).

# Directory structure

Since a large AngularJS application has many components, it's best to structure them in a directory hierarchy.
We use the following approach:

* High level division by functionality and lower level division by component types.
* High level division = AngularJS module
* Tests in the same directory as the tested component
* HTML templates in the same directory as the corresponding controller
* Routes are defined in JS file of the controller
* The `app.js` file contains the main routes definition and configuration
* Each JavaScript file should only hold a single component. The file should be named with the component's name.
* The `common` module contains cross-cutting concerns and components that should be easlily reusable for other applications

Here is the layout:

    .
    ├── app
    │   ├── app.js
    │   ├── common
    │   │   ├── controllers
    │   │   │   ├── ApplicationCtrl.js
    │   │   │   └── ApplicationCtrl.spec.js
    │   │   ├── directives
    │   │   ├── filters
    │   │   └── services
    │   ├── page1
    │   │   ├── controllers
    │   │   │   ├── PageCtrl.js
    │   │   │   ├── PageCtrl.spec.js
    │   │   │   └── page-1.tpl.html
    │   │   ├── directives
    │   │   │   ├── directive1.js
    │   │   │   └── directive1.spec.js
    │   │   ├── filters
    │   │   │   ├── filter1.js
    │   │   │   └── filter1.spec.js
    │   │   └── services
    │   │       ├── service1.js
    │   │       └── service1.spec.js
    │   └── page2
    │       ├── controllers
    │       │   ├── OtherPageCtrl.js
    │       │   ├── OtherPageCtrl.spec.js
    │       │   └── other-page.tpl.html
    │       └── services
    │           ├── service2.js
    │           └── service2.spec.js
    └── components


Conventions about components naming can be found in each component section.

#Controllers

* Do not manipulate DOM in your controllers. Use directives instead.
* The controllers are named UpperCamelCase (`HomePageCtrl`, `ShoppingCartCtrl`, `AdminPanelCtrl`, etc.).
* The naming of the controller is done using the controllers functionality (for example shopping cart, homepage, admin panel) and the suffix `Ctrl`.
* The controllers should not be defined as globals (no matter AngularJS allows this, it is a bad practice to pollute the global namespace).
* Make the controllers as lean as possible. Abstract commonly used functions into a service.
* Communicate between different controllers using method invocation (possible when children wants to communicate with parent) or `$emit`, `$broadcast` and `$on` methods. The emitted and broadcasted messages should be kept to a minimum.
* Make a list of all messages which are passed using `$emit`, `$broadcast` and manage it carefully because of name collisions and possible bugs.
* When you need to format data, encapsulate the formatting logic into a [filter](#filters) and declare it as dependency:


        module.filter('myFormat', function () {
          return function () {
            //body...
          };
        });

        module.controller('MyCtrl', function ($scope, myFormatFilter) {
          //body...
        });

#Directives

* Name your directives with lowerCamelCase
* Use `scope` instead of `$scope` in your link function. In the compile, post/pre link functions you have already defined arguments which will be passed when the function is invoked, you won't be able to change them using DI. This style is also used in AngularJS's source code.
* Use custom prefixes for your directives to prevent name collisions with third-party libraries.
* Do not use `ng` or `ui` prefixes since they are reserved for AngularJS and AngularJS UI usage.
* DOM manipulations must be done only through directives.
* Create an isolated scope when you develop reusable components.
* Use directives as attributes or elements instead of comments or classes, this will make your code more readable.
* Use `$scope.$on('$destroy', fn)` for cleaning up. This is especially useful when you're wrapping third-party plugins as directives.

#Filters

* Name your filters with lowerCamelCase
* Make your filters as light as possible. They are called often during the `$digest` loop so creating a slow filter will slow down your app.

#Services

* Use lowerCamelCase to name your services.
* Encapsulate business logic in services.
* For the difference between `service` and `factory`, see [this answer on StackOverflow](http://stackoverflow.com/questions/13762228/confused-about-service-vs-factory/13763886#13763886), note: services and factories create a singleton
* For session-level cache you can use `$cacheFactory`. This should be used to cache results from requests or heavy computations.

#Templates

* Use lower case name separated with dash and suffix tpl.html (some-page.tpl.html)
* Use `ng-bind` or `ng-cloak` instead of simple `{{ }}` to prevent flashing content.
* Avoid writing complex code in the template.
* When you need to set the `src` of an image dynamically use `ng-src` instead of `src` with `{{}}` template.

#Routing

* Use `resolve` to resolve dependencies before the view is shown.

#Testing

TBD
