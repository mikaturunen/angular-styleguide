# angular-styleguide
Angular.js styleguide - Both ES5 and ES6 formats - TypeScript  version heavily leans on ES6 with the addition of types.

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
```
