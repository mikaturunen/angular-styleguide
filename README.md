# Angular Styleguide

Angular.js styleguide - Both ES5 and ES6 - TypeScript version heavily leans on ES6 with the addition of types.

The following is my extremely opinionated view on how one should aim to write highly maintainable Angular.js + JavaScript (or TypeScript) code in teams. The styling sacrifices performance for readability in some cases. Maintainability and readability over everything else. Early optimizations are bad.

Obviously I cannot answer everything that come up but the main lines should be fairly clear and after that it should be fairly easy to use the main lines on the corner cases too.

Now let's get started.

Oh and on that note: the content is probably contain some typos and issues as I've written it together mostly pretty quickly. So I'll be updating it here and there when new practices emerge and when typos / problems are spotted.

## Defining modules

Modules can be defined in multiple ways in Angular.js but in general we'll want to follow functional style as much as possible in addition to the [fluent interface](https://en.wikipedia.org/wiki/Fluent_interface) style.

### Bad

Placing the module into a variable and then separately declaring the modules.

```js
var application = angular.module("application", []);
application.controller( ... );
application.factory( ... );
```

### Good, ES5 and ES6

Following the fluent interfacing style, just calling the separate functions we want to populate.

```js
angular.module("application", [])
	.controller( ... )
	.factory( ... );
```

## Populating Modules

Modules consist of `controllers`, `services`, `factory` and `directive`. Keep the modules in separate files. One module per one file. The functions that make up the module can be located in different files. But a single file should never contain more than one module definition.

### Bad

Often sold and debated over with statements like "you'll get function names in stack traces" and such but that's completely unnecessary as you'll anyway get the file name in the trace that exploded and from there it's extremely simple to track where the actual fault was.

On top of that your files should never be thousands of lines. Aaaand like with everything, it'll go through minification anyway so the strack trace with the "named" function doesn't really look any better than it does now. Instead rely heavily on sourcemaps where they are supported.

This way also promotes overhead on the top part of the file in a sense that when you start reading it you'll find the module definition at the end of the file and not at the top. So you'll be hopping up and down the file at first before you understand it.

```js
function testCtrl () {

}

angular
  .module("application", [])
  .controller("testCtrl", testCtrl);
```
### Good, ES5

```js
angular.module("application", [])
	.controller("testCtrl", [ "$scope", function($scope) {

	}]);
```

### Good, ES6

Always prefer using explicitly defined dependencies as they are strings and uglification/minification does not change constant strings. This way the application also works through minification process.

Admitably it does add a bit of overhead but we'll take minification any day.

The best option is to rely on modules like [ng-annonate](https://github.com/olov/ng-annotate), but I won't mention that here. If you want to automate the injection of modules as part of your build chain, look at ng-annonate. I highly recommend it.

```js
angular.module("application", [])
	.controller("testCtrl", [ "$scope", $scope => {

	}]);

// or with multiple injected deps
angular.module("application", [])
	.controller("testCtrl", [ "$scope", "$dialog", ($scope, $dialog) => {
	}]);
```

## Services

Services are instantiated by Angular.js and are in a sense class-like. They should be usable by any other service and component in general. Services can define private functionality to aid functionality. Even though often it's said services are class-like and even I mention it, I prefer to think of them as singleton instances and stateless as possible. This further decouples them from the context and they can be easily tested and understood (pure functions!).

### Good, ES5

```js
angular.module("application", [])
	.service("FooService", [ "BarService", "$http", function(barService, $http) {
		var magic = function(a) {
			console.log(a);
		};

		/**
		 * Foo's the foo and bar.
		 * @param {number} foo The foo of the calculation.
		 * @param {number} bar The bar of the calculation.
		 * @returns {number} Result of the addition.
		 */
		var foo = function(foo, bar) {
			magic(foo);
			return foo + bar;
		};

		/**
		 * Foo's the foo and bar.
		 * @param {number} foo The foo of the calculation.
		 * @param {number} bar The bar of the calculation.
		 * @param {Function} callback A null, result style callback.
		 */
		var bar = function(foo, bar, callback) {
			var successCallback = function(result) {
				callback(null, result);
			};

			var errorCallback = function(error) {
				callback(error);
			};

			magic(foo);
			$http.get("/foobarify")
				.then(successCallback, errorCallback);
		}

		return {
			foo: foo,
			bar: bar
		};
	});
```

### Good, ES6

```js
angular.module("application", [])
	.service("FooService", [ "BarService", "$http", (barService, $http) => {
		/**
		 * Foo's the foo and bar.
		 * @param {number} foo The foo of the calculation.
		 * @param {number} bar The bar of the calculation.
		 * @returns {number} Result of the addition.
		 */
		const foo = (foo, bar) => foo + bar;

		/**
		 * Foo's the foo and bar over http.
		 * @param {number} foo The foo of the calculation.
		 * @param {number} bar The bar of the calculation.
		 * @param {Function} callback A null, result style callback.
		 */
		const bar = (foo, bar, callback) => $http.get("/foobarify").then(result => callback(null, result), error => callback(error));

		// Another variation of the above long line, which is quite common with more than three parameters
		// const bar = (
		// 		foo,
		// 		bar,
		// 		callback
		// 	) => $http.get("/foobarify").then(result => callback(null, result), error => callback(error));

		// ES6 feature
		return {
			foo,
			bar
		};
	});
```

## Directives

Directives are purely meant for manipulating the HTML DOM. Essentially any modifications, creation or updates to DOM that needs to be done directly, needs to be done inside a directives. All the code that is reusable, both behavior and markup, should be encapsulated and extracted out of the directive.

### Bad

Text book example of modifying the DOM from the controller, this is a big no-no!

```js
angular.module("application", [])
	.controller("FooCtrl", [ "FooService", function(fooService) {
		this.makeActive = function (element) {
			element.addClass("test");
		};
	}]);
```

### Good, ES5

Using directive for DOM manipulation, encapsulating the functionality and markup.

```js
angular.module("application", [])
	.directive("FooDirective", [ "FooService", function(fooService) {
		var template = [
			"<a href="" class='foo-buttoner' ng-transclude>",
				"<i class='icon-foo-sign'></i>",
			"</a>"
		].join("");

		var linker = function ($scope, $element, $attrs) {
			// DOM manipulation/events here!
			$element.on("click", function () {
				$(this).addClass("fooify");
			});
		}

		return {
			restrict: "EA",
			link: linker,
			template: template,
			// or alternatively for template:
			// templateUrl: FooDirective.html
		};
	}]);
```

Taking it a bit further with ES6. Leans on same principles as ES5 and goes a bit further.

```js
angular.module("application", [])
	.directive("FooDirective", [ "FooService", fooService => {
		const linker = ($scope, $element, $attrs) => $element.on("click", function () {
			$(this).addClass("fooify");
		});

		return {
			restrict: "EA",
			link: linker,
			templateUrl: "FooDirective.html"
		};
	}]);
```

## Factories

Factories are singleton modules that are commonly used for communication purposes, but in truth there's not much separating the service and factory other than the name. You can use both of them for the same things and the line gets even thinner by the common concensus of "factory is a pattern/implementation" which in turn means that factory should not be part of the modules name but they should be called "services" naming wise. In fact I strongly suggest you drop the factories out completely and just use services for everything you would use Factories and Services for. This makes things slightly more easier to understand as you've essentially eliminated one concept out of Angular.js that has no other meaning than just excess fluff around the pointy corners.

The services should be bound to the feature and it should be commonly obvious from the name of the service what it does so the separation into factories and services is fairly useless.

### Good, ES6

I'm going to give out only ES6 example of the factory as I encourage leaving them out completely. Notice the naming of the factory `FooisService` instead of `FooisFactory`. This is common among developers and further encourages leaving out factories completely.

```js
angular.module("application", [])
	.factory("FooisService", [ "BarService", "$http", (barService, $http) => {
		/**
		 * Foo's the foo and bar.
		 * @param {number} foo The foo of the calculation.
		 * @param {number} bar The bar of the calculation.
		 * @returns {number} Result of the addition.
		 */
		const foo = (foo, bar) => foo + bar;

		/**
		 * Foo's the foo and bar over http.
		 * @param {number} foo The foo of the calculation.
		 * @param {number} bar The bar of the calculation.
		 * @param {Function} callback A null, result style callback.
		 */
		const bar = (foo, bar, callback) => $http.get("/foobarify").then(result => callback(null, result), error => callback(error));”

		return {
			foo,
			bar
		};
	});
```

## Naming

Angular has taken over the ng-* prefix and you should prefix your directives with something that differentiates them from the core functionality and don't cause clashing with the potential future updates of Angular. I always suggest using the applications or frameworks domain as the prefix.

Do not prefix anything with the following:

* `ng*`
* `$`
* `$$`

These are solely Angular core functionality and you're just calling for trouble if you get mixed in with them.

### Bad

First version of Angular did not have ng-focus and developers released a lot of modules called 'ng-focus', causing lot of issues once Angular.js team decided to release ng-focus as core functionality of Angular. Obviously this broke all the third party modules. Plus when you are looking information for ng-focus you easily get side tracked to the third party components.

```js
angular
	.module("application", [])
	.directive("ngFocus", [ "FooService", function(fooService) {
		return {};
	}]);
```

### Good, ES5

This way we make sure there are no clashes with existing core directives.

```js
angular
	.module("application", [])
	.directive("fooFocus", [ "FooService", function(fooService) {
		return {};
	}]);
```

### Good, ES6

```js
angular
	.module("application", [])
	.directive("fooFocus", [ "FooService", (fooService) => {
		return {};
	});
```

## Global context

If you are using vanilla JavaScript, you can stop the global namespace pollution with the IIFE wrap pattern, create anon function that calls itself instantly.

```js
(() => {

	angular.module("application", [])
		.controller("testCtrl", [ "$scope", "$dialog", ($scope, $dialog) => {

		}])
		.controller("fooCtrl", [ "$scope", $scope => {

		}]);

})();
```

Other ways of avoiding this are to use typed super set of JavaScript and wrap it in a namespace or module loader or sorts (AMD.js for example). There are number of ways to avoid this but that's totally a different topic...

Ps. I always prefer having a module loader and writing my content to be module loader compatible.

## Controllers

NO! I know what you're thinking, for the love of god, please, do not use `ng-controller` logic anywhere. It just wrecks havoc and causes issues here and there (scopes, hard to understand what's going on, there's no single point where things are declared). Not a good thing.

In general with angular you should heavily lean on the functionality of the $routeProvider to tell Angular what views and controllers go together.

```js
<!-- main.jade => Compiled to HTML. Closing tags = annoying. -->
div
    {{ main.someObject }}
<!-- /main.jade -->

<script>
	// ...

	angular.module('app')
		.config([ "$routeProvider", ($routeProvider) {
			$routeProvider.when('/', {
				templateUrl: 'views/main.html',
				controller: 'MainCtrl',
				controllerAs: 'main'
			});
		});

	//...
</script>
```

When going full single page application behavior with Angular, you want to lean on external modules for additional leverage, like the [de-facto ui-router for Angular.js 1.x](https://github.com/angular-ui/ui-router).

## Async resolution through promises

After creating set of services, you'll be injecting them to other services potentially, controllers and maybe some directives. Essentially the shared functionality will spread around. To keep the contollers clean of the service logic, you want to leverage angular-routers (or ui-router.js') resolve property to resolve the view's promises before the page is served to the user. Controllers are instantiated after the data is available.

## Mediocre

```js
angular
	.module("application", [])
	.controller("TestCtrl", [ "FooService", function(fooService) {
		var self = this;
		// unresolved
		self.something;
		// resolved asynchronously
		fooService.doSomething().then(function (response) {
			self.something = response;
		});
	});
```

## Good, ES5

```js
angular
	.module("application", [])
	.config([ "$routeProvider", function($routeProvider) {
		$routeProvider.when("/", {
			templateUrl: "test.html",
			controller: "TestCtrl",
			// Using controllerAs syntax here allows us to easily identify where the informatiol belongs in the html
			// <div> {{ testCtrl.property }} </div>
			// This is not required but advocates good readability, so use it
			controllerAs: "testCtrl",
			resolve: {
				doSomething: function (fooService) {
		  			return SomeService.doSomething();
				}
			}
		});
	});
...

angular
	.module("application")
	.controller("TestCtrl", [ "fooService", function(fooService) {
  		// resolved!
		this.something = fooService.doSomething;
	});
```

## Good, ES6

```js
angular
	.module("application", [])
	.config([ "$routeProvider", $routeProvider => {
		$routeProvider.when("/", {
			templateUrl: "test.html",
			controller: "TestCtrl",
			// Using controllerAs syntax here allows us to easily identify where the information belongs in the html
			// <div> {{ testCtrl.property }} </div>
			// This is not required but advocates good readability, so use it
			controllerAs: "testCtrl",
			resolve: {
				doSomething: fooService => SomeService.doSomething();
			}
		});
	});

...

angular
	.module("application")
	.controller("TestCtrl", [ "fooService", fooService => {
  		// resolved!
		this.something = fooService.doSomething;
	});
```

# Waiting times

When routes are resolved, it commonly means (with the above resolve patterns) that there are involved loading times and this means the user is waiting. We want to indicate progress somehow. By means of showing a spinner or a loading bar. Angular fires `$routeChangeStart` event when navigation occurs and by listening to this we can show user indication of progress and when Angular fires `$routeChangeSuccess` we can hide the indicators.

# $scope.$watch is route to hell

If you can, avoid $route.$watch. You can almost always avoid this by understanding how Angular works and restructuring your front architecture. Unfortunately there are times that you'll still come face to face with Lucifer and you'll accept the satanic performance cost of $scope.$watch. If you do, here are some tips on how to avoid the biggest issues.

## Idiotic

```js
<input ng-model="fooModel">
<script>
	$scope.$watch("fooModel", callback);
</script>
```

## Good

```js
<input ng-model="fooModel" ng-change="callback">

...

	$scope.callback = function() {
		// Magic happens here!
	};

```

# File structuring

One feature, one file. If you have a feature that has services and a controller to it, they all have their individual files.

```

|-- TrackUser
|---- TrackUserCookiesService.js
|---- TrackUserHistoryService.js
|---- TrackUserController.js
|---- TrackUserDirective.js
|-- Utility
|---- UtilityFooService.js
|---- UtilityBarService.js


```

All the external communication and shared functionality is extracted into the services that the controller(s) and directive(s) then can use. Do not have three huge directories that just say `controllers`, `directives` and `services`... That's also a big no-no. It's messy and makes hard to understand what things work with what.

# Order of injections/annonation

When you are injecting content into a module in Angular, you should follow the basic idea of injecting the Angular specific modules first and then your application specific. So Angular's core first and then application specific.

## Bad

```js
...
// Random order
function TestCtrl (TestService, $scope, FooService, $rootScope) {

}
```

## Good

```js
...
// Angular -> application
function TestCtrl ($scope, $rootScope, MyService, AnotherService) {

}
```

# Final words

It's not without its issues, sure, but so far it's one of the most efficient ways to write readable and maintainable Angular.js code in my opinion. It aims to answer the question "where is this defined" as easily as possible and keeps everything in feature-wise locations. It should contain one place for the route definitions, one place that defines all the information what controllers are used where and the HTML should easily answer the "this function call / property is coming from this specific controller". It aims to minimize $watch usage in all cases as it comes with a huge performance cost in bigger applications and avoids all specific scope -magic as that's always causis issue and never really solving any problems.

I think the whole thing aims to be as simple as possible and to the point; for example, encourages dropping out the factory service style completely and instead suggests one should name services based on Features and that the feature naming should already tell the user everything it needs to know about it.

*SIMPLE, MAINTAINABLE, EASY TO READ = GREAT ANGULAR.JS CODE.*

What else? It's open to suggestions and is bound to change when opinions change and better ones come up.

# Additional reading

The following links have server as inspiration, points of entry or just in general relate to my interest in guiding lines for coding:

* https://github.com/airbnb/javascript
* https://github.com/panuhorsmalahti/typescript-style-guide
* https://toddmotto.com/opinionated-angular-js-styleguide-for-teams/
* http://www.jslint.com/
* https://github.com/Microsoft/TypeScript/wiki/Coding-guidelines
