# Hello wire.js

This is a simple Hello World example for [wire.js](https://github.com/briancavalier/wire), a Javascript IOC Container.

# Simple and Ridiculous Example - Hello wire()d!

Here's a very simple wire.js take on Hello World.  Wire.js can use AMD modules, so first, let's use AMD to define a very simple wiring spec.  Wiring specs are simply JSON or Javascript objects.

	define(['hello-wired-spec'], { message: "Hello wire()d!" });
	
In this case our wiring spec is a laughably simple object with a single String property.  Next, let's wire() the spec using wire.js as an AMD plugin. 
	
	require(['wire!hello-wired-spec'], function(wired) {
		alert(wired.message);
	});
	
As you probably guessed, this will alert with the message "Hello wire()d!".  Yes, that was silly, so let's create a more interesting Hello wire()d.

# Simple, but Less Ridiculous Hello wire()d!

You can run this example by cloning the repo and loading `index.html` in your browser.  First, we'll define a component using AMD:

	define([], function() {
		function HelloWired(node) {
			this._node = node;
		}
	
		HelloWired.prototype = {
			sayHello: function(message) {
				this._node.innerHTML = "Hello! " + message;
			}
		};
	
		return HelloWired;
	});
	
This is a simple module that returns a Javascript constructor for our HelloWorld object.  Now, let's create a wiring spec for our tiny Hello wire()d app:

	define({
		message: "I haz been wired",
		helloWired: {
			create: {
				module: 'hello-wired',
				args: { $ref: 'dom!hello' }
			},
			init: {
				sayHello: { $ref: 'message' }
			}
		},
		plugins: [
			{ module: 'wire/dom' }
		]
	});
	
And finally, let's create a page for our little app

	<!DOCTYPE HTML>
	<html>
	<head>
		<title>Hello wire()d!</title>	
		<!-- AMD Loader, in this case curl.js -->
		<script type="text/javascript" src="curl.js"></script>
		
		<!-- Wire the Hello wire()d spec to create the app -->
		<script type="text/javascript">
			require(['wire!hello-wired-spec']);
		</script>
	</head>

	<body>
		<header>
			<h1 id="hello"></h1>
		</header>
	</body>
	</html>

When you load this page in a browser, you'll see the text "Hello! I haz been wired" in the `h1`.

# What Happened?

## The Page

So, what happened when we loaded the page?  Let's start with two interesting parts of the page.  First, there is a script tag to load curl.js, an AMD loader--wire.js uses the loader to load AMD style modules.

	<!-- AMD Loader, in this case curl.js -->
	<script type="text/javascript" src="curl.js"></script>

Then, there is a call to the loader.  Wire.js can be used as either an AMD loader plugin or as an AMD module.  In this case, we're using it as an AMD plugin.  In particular, we're using wire.js to load and process the wiring spec defined in the AMD module named `hello-wired-spec`.

	<!-- Wire the Hello wire()d spec to create the app -->
	<script type="text/javascript">
		require(['wire!hello-wired-spec']);
	</script>

## The Wiring Spec

Now let's walk through the wiring spec to see what happens when wire.js processes it.  First, there is the standard AMD module define wrapper:

	define({
	...
	});
	
Next we have a String property.  This does what you probably expect.  It creates a String property named `message` whose value is `"I haz been wired"`.

	message: "I haz been wired",
	
Then we have the `helloWired` property, which is more interesting:

	helloWired: {
		create: {
			module: 'hello-wired',
			args: { $ref: 'dom!hello' }
		},
		init: {
			sayHello: { $ref: 'message' }
		}
	},

This creates an instance of the `hello-wired` module, invoking it as a constructor, and passing a single argument (we'll see what that argument is below, but you can probably guess).  Then wire.js will invoke the `sayHello` method on the `hello-wired` instance, again passing a single argument.

So, this part of the spec creates an instance of `hello-wired`, and initializes it by calling its `sayHello` method.

Finally, in the wiring spec, we have an Array named `plugins`.  There is nothing special about the name `plugins`--it is not a wire.js keyword (wire.js has very few keywords, and most functionality is provided via pluggable syntax from plugins).

	plugins: [
		{ module: 'wire/dom' }
	]

The array has a single element, which is an object.  That object *does* use one of wire.js's keywords, `module`.  In this case, wire.js will load the AMD module `wire/dom`.  So the result is an array named `plugins` with a single element whose value is the result of loading the module `wire/dom`.

That AMD module just happens to be a wire.js plugin.  Plugins can provide several types of features, but in this case, the `wire/dom` plugin provides a *reference resolver* for DOM nodes.  Reference resolvers provide a way to resolve references to other *things*, like other objects in the wiring spec, or, in this case, DOM nodes on the page, and it does so without your having to worry about DOMReady.

## Ok, Now Back to Those Arguments

You probably guessed that there is some relationship between the `wire/dom` plugin and the `{ $ref: 'dom!hello' }` bit in the `helloWired` object.  Yep, there is.  When instantiating the `helloWired` object, wire.js will pass its constructor a single parameter (it is also possible to pass multiple parameters using an array, but let's keep it simple for now):

	args: { $ref: 'dom!hello' }

That single parameter is a reference to a DOM Node whose `id="hello"`.  This is Dependency Injection at work.  The `hello-wired` instance needs a DOM Node to do it's job, and we have supplied one by referencing it using the `wire/dom` plugin's `dom!` reference resolver.

You can think of this as:
	
	new HelloWired(document.getElementById("hello"))
	
but you'd also need to add your own DOMReady wrapper/check, which wire.js gives you for free.

Then, when wire.js invokes the `sayHello` method on the instance, it also passes a single parameter:

	sayHello: { $ref: 'message' }
	
In this case, the parameter is a reference to the `message` String, which is the first item in the spec.  You can think of this as:

	helloWired.sayHello(message)

Parameters don't have to be references.  For example, we could have just as easily provided a message inline:

	sayHello: "I haz been wired"
	
## Finally, a Note on Order

Wiring specs are *declarative*, and thus order of things in a wiring spec doesn't matter.  For example, in this example, the `wire/dom` plugin was declared after the `{ $ref: 'dom!hello' }` reference.  That's no problem.  Wire.js ensures that the plugin is ready before resolving the DOM Node reference.

Also notice that we didn't have to write any code to wait for DOM Ready.  Again, wire.js ensures that the DOM Node reference is resolved only after the DOM is indeed ready.

As you can imagine, there is an implicit ordering to the things that happen, even in this simple Hello Wire example.  The `wire/dom` plugin must be loaded and ready before the DOM Node reference can be resolved, which must happen before the HelloWired instance can be created, since it requires the DOM Node as a constructor parameter.  And finally, the sayHello initializer method can only be invoked after the HelloWired instance has been created.

This concept of *automatic ordering* is a key feature of wire.js.  You simply write a declarative wiring spec, that is, you describe *what* you want, and wire.js makes it happen, without your having to worry about the order. 

Hello Wire is a trivial example, and the code to do this ordering yourself would be trivial, but as you deal with larger and larger systems with more and more collaborating components, it can be a big advantage to let wire.js take care of these kinds of ordering issues.