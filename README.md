# angular-styleguide
Angular.js styleguide - Both ES5 and ES6 formats - TypeScript  version heavily leans on ES6 with the addition of types.


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

### Good, ES6

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
