---
title: Extending Classes And Interfaces
description: Learn how NativeScript supports the native Android development scenarios which involve inheriting and extending existing classes or implementing interfaces. 
position: 2
tags: extending classes, nativescript android, nativescript interfaces
slug: extending-classes
---

# Extending Classes And Interfaces

As part of the native Android development you often have to inherit from classes and/or implement interfaces. NativeScript supports these scenarios as well.

## Classes

```Java
public class MyButton extends android.widget.Button {
	public MyButton(Context context) {
		super(context);
	}
	
	@Override
	public void setEnabled(boolean enabled) {
		super.setEnabled(enabled);
	}
}

MyButton btn = new MyButton(context);
```
```kotlin
class MyButton(context: Context) : android.widget.Button(context) {
    override fun setEnabled(enabled: Boolean) {
        super.setEnabled(enabled)
    }
}

val btn = MyButton(context)
```

Here is how the above is done in NativeScript:

``` JavaScript
let constructorCalled = false;
let MyButton = android.widget.Button.extend({
	// constructor
	init: function() {
		constructorCalled = true;
	},
	
	setEnabled: function(enabled) {
		this.super.setEnabled(enabled);
	}
});

let btn = new MyButton(context);
// constructorCalled === true
```
``` TypeScript
class MyButton extends android.widget.Button {
	static constructorCalled: boolean = false;
	// constructor
	constructor() {
		super();
		MyButton.constructorCalled = true;

		// necessary when extending TypeScript constructors
		return global.__native(this);
	}

	setEnabled(enabled : boolean): void {
		this.super.setEnabled(enabled);
	}
}

let btn = new MyButton(context);
// MyButton.constructorCalled === true
```

> **Note:** In the above `setEnabled` function the `this` keyword points to the JavaScript object that proxies the extended native instance. The `this.super` property provides access to the base class method implementation.

Creating an anonymous Java class which extends from the base Java `java.lang.Object` class:

``` JavaScript
let MyClass = java.lang.Object({
	// constructor
	init: function() {
	},
	
	toString: function() {
		// override Object's toString
	}
});

let myClassInstance = new MyClass();
```
``` TypeScript
class MyClass extends java.lang.Object {
	// constructor
	constructor() {
		super();
		// necessary when extending TypeScript constructors
		return global.__native(this);
	}

	toString(): string {
		// override Object's toString
	}
}

let myClassInstance: any | java.lang.Object = new MyClass();
```

Creating a named Java class which extends from the `java.lang.Object` class will allow referring to the class by its full package name (in AndroidManifest.xml, for example):
``` JavaScript
let MyClass = java.lang.Object("my.application.name.MyClass", {
	// constructor
	init: function() {
	},
	
	toString: function() {
		// override Object's toString
	}
});

let myClassInstance = new MyClass();
let myClassInstance2 = new my.application.name.MyClass();
```
``` TypeScript
class MyClass extends java.lang.Object {
	// constructor
	constructor() {
		super();
		// necessary when extending TypeScript constructors
		return global.__native(this);
	}

	toString(): string {
		// override Object's toString
	}
}

let myClassInstance: any | java.lang.Object = new MyClass();
let myClassInstance2: any | java.lang.Object = new my.application.name.MyClass(); // may produce a TypeScript compilation error, because the namespace will not be recognized, it's safe to ignore
let myClassInstance3: any = new (<any>my).application.name.MyClass(); // TypeScript compiler-safe
```

> One important thing to note when dealing with extending classes and implementing interfaces in NativeScript is that, unlike in Java - where you can extend an **Abstract** class with a **new java.arbitrary.abstract.Class() { }**, in NativeScript the class needs to be extended as per the previous examples - using the `extend` function on the `java.arbitrary.abstract.Class`, or using the `extends` class syntax in TypeScript.

## Interfaces
The next example shows how to implement an interface in Java/Kotlin and NativeScript. The main difference between inheriting classes and implementing interfaces in NativeScript is the use of the `extend` keyword. Basically, you implement an interface by passing the *implementation* object to the interface constructor function. The syntax is identical to the [Java Anonymous Classes](http://docs.oracle.com/javase/tutorial/java/javaOO/anonymousclasses.html).

```Java
button.setOnClickListener(new View.OnClickListener() {
	public void onClick(View v) {
		// Perform action on click
	}
});
```
```Kotlin
button.setOnClickListener {
    // Perform action on click
}
```

``` JavaScript
button.setOnClickListener(new android.view.View.OnClickListener({
	onClick: function() {
		// Perform action on click
	}
}));
```
``` TypeScript
button.setOnClickListener(new android.view.View.OnClickListener({
	onClick: function() {
		// Perform action on click
	}
}));
```

Alternatively you can use the following pattern for a named interface implementation:

``` JavaScript
let ClickListener;

function initializeClickListener() {
    if (ClickListener) {
        return;
    }

    ClickListener = java.lang.Object.extend({
        interfaces: [android.view.View.OnClickListener], /* the interfaces that will be inherited by the resulting class */
        onClick: function() {
            // Perform action on click
        }
    });
}

// [...]

{
	// [...]

	initializeClickListener();
	nativeView.setOnClickListener(new ClickListener());
}
```
``` TypeScript
interface ClickListener {
    new(): android.view.View.OnClickListener;
}

let ClickListener: ClickListener;

function initializeClickListener(): void {
    if (ClickListener) {
        return;
    }

	@Interfaces([android.view.View.OnClickListener])
	class ClickListenerImpl extends java.lang.Object implements android.view.View.OnClickListener {
		constructor() {
			super();

			// necessary when extending TypeScript constructors
			return global.__native(this);
		}

		onClick(view: android.view.View): void {
			// Perform action on click
		}
	}

	ClickListener = ClickListenerImpl;
}

// [...]

{
	// [...]

	initializeClickListener();
	nativeView.setOnClickListener(new ClickListener());
}
```

### Implementing multiple interfaces in NativeScript 

Suppose you have the following interfaces in Java/Kotlin:

```Java
public interface Printer {
	void print(String content);
	void print(String content, int offset);
}

public interface Copier {
	String copy(String content);
}

public interface Writer {
	void write(Object[] arr);
	void writeLine(Object[] arr)
}
```
```Kotlin
interface Printer {
    fun print(content: String)
    fun print(content: String, offset: Int)
}

interface Copier {
    fun copy(content: String): String
}

interface Writer {
    fun write(arr: Array<Any>)
    fun writeLine(arr: Array<Any>)
}
```

Implementing the interfaces is as easy in Java as writing:

```Java
public class MyVersatileCopywriter implements Printer, Copier, Writer {
	public void print(String content) {	...	}

	public void print(String content, int offset) { ... }

	public String copy(String content) { ... }

	public void write(Object[] arr) { ... }

	public void writeLine(Object[] arr) { ... }
}
```
```Kotlin
class MyVersatileCopywriter: Printer, Copier, Writer{
    
    override fun print(content: String) { ... }

    override fun print(content: String, offset: Int) { ... }

    override fun copy(content: String): String { ... }

    override fun write(arr: Array<Any>) { ... }

    override fun writeLine(arr: Array<Any>) { ... }
}
```

The same result can be achieved in NativeScript by [extending]({%slug how-extend-works%}) any valid object that inherits [Java Object](https://docs.oracle.com/javase/7/docs/api/java/lang/Object.html). 

- In JavaScript - Declare an **interfaces** array in the implementation
- Using Typescript syntax - apply a **decorator** to the extended class (note `@Interfaces([...]`))

Using Javascript syntax - attach `interfaces` array to implementation object of the extend call

``` JavaScript
let MyVersatileCopyWriter = java.lang.Object.extend({
	interfaces: [com.a.b.Printer, com.a.b.Copier, com.a.b.Writer], /* the interfaces that will be inherited by the resulting class */
	print: function() { ... }, /* implementing the 'print' methods from Printer */
	copy: function() { ... }, /* implementing the 'copy' method from Copier */
	write: function() { ... }, /* implementing the 'write' method from Writer */
	writeLine: function() { ... }, /* implementing the 'writeLine' method from Writer */
	toString: function() { ... } /* override `java.lang.Object's` `toString */
});
```
``` TypeScript
@Interfaces([com.a.b.Printer, com.a.b.Copier, com.a.b.Writer]) /* the interfaces that will be inherited by the resulting MyVersatileCopyWriter class */
class MyVersatileCopyWriter extends java.lang.Object { 
	constructor() {
		super();
		return global.__native(this);
	}

	print() { ... }
	copy() { ... }
	write() { ... }
	writeLine() { ... }
}
```


### Limitations
* Implementing two interfaces with the same method signature will generate just 1 method. It is the implementor's responsibility to define how the method will behave for both interfaces
* Implementing two interfaces with the same *method name*, *parameter number*, but **different return type** (`void a()` vs `boolean a()`) will result in a compilation error

### Notes
> Java/Kotlin method overloads are handled by the developer by explicitly checking the `arguments` count of the invoked function

``` JavaScript
let MyVersatileCopyWriter = ...extend({
	...
	print: function() {
		let content = "";
		let offset = 0;

		if (arguments.length == 2) {
			offset = arguments[1];
		}

		content = arguments[0];

		// do stuff
	}
	...
})
```
``` TypeScript
class MyVersatileCopyWriter extends ... {
	constructor() {
		super();
		return global.__native(this);
	}
	...
	print() {
		let content = "";
		let offset = 0;

		if (arguments.length == 2) {
			offset = arguments[1];
		}

		content = arguments[0];

		// do stuff
	}
	...
}
```

> In addition to implementing interface methods, you can override methods of the extended class, and also declare your own methods that the new class should have.

## See Also
* [How Extend Works](./how-extend-works.md)
* [Gotchas](./gotchas.md)
