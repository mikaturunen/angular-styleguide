# angular-styleguide

Angular.js styleguide - Both ES5 and ES6 formats - TypeScript  version heavily leans on ES6 with the addition of types.

The following is my extremely opinionated view on how one should aim to write highly maintainable Angular.js + JavaScript (or TypeScript) code in teams. The styling sacrifices performance for readability in some cases. Maintainability and readability over everything else. Early optimizations are bad.

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

DOM is purely meant for manipulating the HTML DOM. Essentially any modifications, creation or updates to DOM that needs to be done directly needs to be done inside Directives. All the code that is reusable, both behavior and markup, should be encapsulated and extracted out of the directive.

### Bad

Text book example of modying the DOM from the controller, this is a big no-no!

```js
angular.module("application", [])
	.controller("FooCtrl", [ "FooService", function(fooService) {
		this.makeActive = function (elem) {
			elem.addClass('test');
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
			restruct: "EA",
			link: linker,
			template: template,
			// or alternatively for template:
			// templateUrl: FooDirective.html
		};
	}]);
```

## Factories

Factories are singleton modules that are commonly used for communication purposes, but in truth there's not much separating the service and factory other than the name. You can use both of them for the same things and the line gets even thinner by the common concensus of "factory is a pattern/implementation" which in turn means that factory should not be part of the modules name but they should be called "services" too naming wise. In fact I strongly suggest you drop the factories out completely and just use Services for everything you would use Factories and Services for. This makes things slightly more easier to understand as you've essentially eliminated one concept out of Angular.js that has no other meaning than just excess fluff around the pointy corners.

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

Taking it a bit further with ES6. Leans on same principles as ES5 and goes a bit further.

```js
angular.module("application", [])
	.directive("FooDirective", [ "FooService", fooService => {
		const linker = ($scope, $element, $attrs) => $element.on("click", function () {
			$(this).addClass("fooify");
		});

		return {
			restruct: "EA",
			link: linker,
			templateUrl: "FooDirective.html"
		};
	}]);
```

## Naming

Angular has taken over the ng-* prefix and you should prefix your directives with something that differentiates them from the core functionality and don't cause clashing with the potential future updates of Angular.

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
	}]);
```
