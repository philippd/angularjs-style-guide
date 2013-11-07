#Introduction

The goal of this style guide is to present a set of best practices and style guidelines for a AngularJS application.

In this style guide you won't find common guidelines for JavaScript development. Such can be found at:

0. [Google's JavaScript style guide](http://google-styleguide.googlecode.com/svn/trunk/javascriptguide.xml)
0. [Mozilla's JavaScript style guide](https://developer.mozilla.org/en-US/docs/Developer_Guide/Coding_Style)
0. [GitHub's JavaScript style guide](https://github.com/styleguide/javascript)
0. [Douglas Crockford's JavaScript style guide](http://javascript.crockford.com/code.html)

For AngularJS development recommended is the [Google's JavaScript style guide](http://google-styleguide.googlecode.com/svn/trunk/javascriptguide.xml).

# Directory structure

Since a large AngularJS application has many components it's best to structure them in a directory hierarchy.
We use the following approach:

* High level division by functionality and lower level division by component types.
* High level division = AngularJS module
* Tests in the same directory as the tested component
* HTML templates in the same directory as the corresponding controller
* Routes are defined in JS file of the controller
* The `app.js` file contains the main routes definition and configuration
* Each JavaScript file should only hold a single component. The file should be named with the component's name.
* The `common` module contains

Here is the layout:

    .
    ├── app
    │   ├── app.js
    │   ├── common
    │   │   ├── controllers
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
    │       ├── directives
    │       │   ├── directive2.js
    │       │   ├── directive2.spec.js
    │       │   └── directive-2.tpl.html
    │       ├── filters
    │       │   ├── filter2.js
    │       │   └── filter2.spec.js
    │       └── services
    │           ├── service2.js
    │           └── service2.spec.js
    └── components


Conventions about components naming can be found in each component section.

#Controllers

* Do not manipulate DOM in your controllers. Use directives instead.
* The naming of the controller is done using the controller's functionality (for example shopping cart, homepage, admin panel) and the substring `Ctrl` in the end. The controllers are named UpperCamelCase (`HomePageCtrl`, `ShoppingCartCtrl`, `AdminPanelCtrl`, etc.).
* The controllers should not be defined as globals (no matter AngularJS allows this, it is a bad practice to pollute the global namespace).
* Use array syntax for controller definitions:



        module.controller('MyCtrl', ['dependency1', 'dependency2', ..., 'dependencyn', function (dependency1, dependency2, ..., dependencyn) {
          //...body
        }]);


Using this type of definition avoids problems with minification. You can automatically generate the array definition from standard one using tools like [ng-annotate](https://github.com/olov/ng-annotate) (and grunt task [grunt-ng-annotate](https://github.com/mzgol/grunt-ng-annotate)).
* Use the original names of the controller's dependencies. This will help you produce more readable code:



        module.controller('MyCtrl', ['$scope', function (s) {
          //...body
        }]);


is less readable than:


        module.controller('MyCtrl', ['$scope', function ($scope) {
          //...body
        }]);


This especially applies to a file that has so much code that you'd need to scroll through. This would possibly cause you to forget which variable is tied to which dependency.

* Make the controllers as lean as possible. Abstract commonly used functions into a service.
* Communicate within different controllers using method invocation (possible when children wants to communicate with parent) or `$emit`, `$broadcast` and `$on` methods. The emitted and broadcasted messages should be kept to a minimum.
* Make a list of all messages which are passed using `$emit`, `$broadcast` and manage it carefully because of name collisions and possible bugs.
* When you need to format data encapsulate the formatting logic into a [filter](#filters) and declare it as dependency:


        module.controller('myFormat', function () {
          return function () {
            //body...
          };
        });

        module.controller('MyCtrl', ['$scope', 'myFormatFilter', function ($scope, myFormatFilter) {
          //body...
        }]);

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

* Use camelCase (lower or upper) to name your services.
* Encapsulate business logic in services.
* Services encapsulating business logic are preferably a `service` instead of a `factory`
* For session-level cache you can use `$cacheFactory`. This should be used to cache results from requests or heavy computations.

#Templates

* Use lower case name separated with dash and suffix tpl.html (some-page.tpl.html)
* Use `ng-bind` or `ng-cloak` instead of simple `{{ }}` to prevent flashing content.
* Avoid writing complex code in the template.
* When you need to set the `src` of an image dynamically use `ng-src` instead of `src` with `{{}}` template.
* Instead of using scope variable as string and using it with `style` attribute with `{{ }}`, use the directive `ng-style` with object-like parameters and scope variables as values:

        ...
        $scope.divStyle = {
          width: 200,
          position: relative
        };
        ...

        <div ng-style="divStyle">my beautifully styled div which will work in IE</div>;

#Routing

* Use `resolve` to resolve dependencies before the view is shown.

#Testing

TBD

# General 
## Optimize the digest cycle

* Watch only the most vital variables (for example: when using real-time communication, don't cause a digest loop in each received message).
* Make computations in `$watch`  as simple as possible. Making heavy and slow computations in a single `$watch` will slow down the whole application (the $digest loop is done in a single thread because of the single-threaded nature of JavaScript).

## Others

* Use:
    * `$timeout` instead of `setTimeout`
    * `$window` instead of `window`
    * `$document` instead of `document`
    * `$http` instead of `$.ajax`

This will make your testing easier and in some cases prevent unexpected behaviour (for example, if you missed `$scope.$apply` in `setTimeout`).

* Use promises (`$q`) instead of callbacks. It will make your code look more elegant and clean, and save you from callback hell.
* Use `restangular` instead of `$http` when possible. Higher level of abstraction saves you from redundancy.
* Don't use globals. Resolve all dependencies using Dependency Injection.
* Do not pollute your `$scope`. Only add functions and variables that are being used in the templates.
* Prefer the usage of [controllers instead of `ngInit`](https://github.com/angular/angular.js/pull/4366/files). The only appropriate use of `ngInit` is for aliasing special properties of `ngRepeat`. Besides this case, you should use controllers rather than `ngInit` to initialize values on a scope.
* Do not use `$` prefix for the names of variables, properties and methods. This prefix is reserved for AngularJS usage.
