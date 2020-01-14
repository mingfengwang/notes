## preventExtensions, seal and freeze  
ECMAScript version 5 is the latest, official, approved standard version of the ECMAScript programming language. If you've never heard of ECMAScript, well, don't feel too bad as it is better known, used, loved (and hated) by its other name - JavaScript. As far as names go JavaScript is probably not a whole lot better than ECMAScript either - as Douglas Crockford so eloquently (half joking actually) put it in a recent session that he did during Microsoft's MIX event:

"Maybe it's time to change the name of it. Because if it isn't Java and if it isn't script then the name is completely wrong!"

Douglas was actually responding to a comment from Allen Wirfs-Brock that JavaScript really isn't a "scripting" language if you think of scripting languages as being programming languages that are somewhat limited in capabilities or are designed to be used with very specific use cases in mind. JavaScript is a full-fledged general purpose programming language that is turing complete which has been used with great success in contexts other than the browser with the node.js project perhaps being the most renowned. It kind of goes to show how things might end up being given names that have little to do with the thing that is being named!

So then, ECMAScript version 5 (referred to as ES5 henceforth) brings a nifty set of features designed, among other things, to introduce an element of discipline to JavaScript development. As far as being a dynamic language is concerned, JavaScript is about as dynamic as a language could probably get! While that dynamism grants considerable expressive power, it also makes it rather trivial to write incorrect code. Consider the following snippet for instance (Note: I have used the JavaScript Eval Console for testing all of the code given below; so when you see calls to functions such as print and sprintf please understand that those are functions provided by the console; please read that post to see what those functions do if it isn't clear from the context):

```
var data = [
    {
        month: 1,
        revenue: 102010,
        expense: 95000
    },

    {
        month: 2,
        revenue: 143232,
        expense: 98000
    },

    {
        month: 3,
        revenue: 323212,
        expense: 195000
    }
];

function Results() {
    this.totalRevenue = 0;
    this.totalExpense = 0;
    this.netProfit = 0;
}

var r = new Results();

//
// iterate through the data set and compute total
// revenue and expense
//
data.forEach(function(d) {
    r.totalrevenue += d.revenue;
    r.totalExpense += d.expense;
});

//
// net profit is revenue - expense
//
r.netProfit = r.totalRevenue - r.totalExpense;
print(r.netProfit);

```

Here's the output we get.

```
	-388000
	
```

Clearly, something is not right here because when you look at the data the revenue exceeds expenses for every single month - so we really shouldn't be ending up with a net loss. Let's take a closer look at the code that's iterating through the array and computing total revenue and expense. Pay particular attention to the part highlighted in bold:

```
data.forEach(function(d) {
    r.totalrevenue += d.revenue;
    r.totalExpense += d.expense;
});

```

As you can see, there's a typo there. The 'r' in totalrevenue should have been upper case. Instead of flagging that as an error the JavaScript engine just went right ahead and created a new property on the Results instance with a lower case 'r'. This of course messed up the net profit computation later on. Wouldn't it be nice if we could somehow prevent the arbitrary creation of new members on objects? It is precisely for situations such as this that the ES5 spec includes a slew of new methods (well, three of them at least) on the Object type that allow you to exercise greater control on the extensibility aspects of your objects. Let's review what they are and what they do.

###Object.preventExtensions

Object.preventExtensions does exactly what it sounds like it would do - it prevents arbitrary properties from being tacked on to the object in question. Here's an example:

```
var o = {};

o.newFangledProperty1 = "zoo";
print(typeof(o.newFangledProperty1));

Object.preventExtensions(o);

o.newFangledProperty2 = "moo";
print(typeof(o.newFangledProperty2));

```
And here's the output we get:

```
string
undefined
```

As is evident from the output, the attempt to create newFangledProperty2 was not successful. The JavaScript engine silently ignored the creation of the new property and it did this because we marked the object as permanently non-extensible by calling Object.preventExtensions on it. You can verify whether an object is extensible or not by calling Object.isExtensible on it which would return a boolean indicating true if it is extensible and false otherwise.

```
print(Object.isExtensible(o));
```

Output:

```
false
```

We can easily use this method on our Results object above to prevent the creation of arbitrary properties on it like so:

```
var r = new Results();
Object.preventExtensions(r);
```

Now the line with the typo will end up being a no-op statement having no effect on the object. The result will still be wrong of course since the totalRevenue property will continue to have the value zero. So what's the point of using preventExtensions you might wonder. In scenarios such as this, using Object.preventExtensions makes the most sense when used in tandem with ES5's support for "strict mode". ES5's "strict mode" is a new feature that causes the JavaScript engine to interpret the relevant section of code (i.e. the part that you have marked as being strict) with a stricter set of rules than what is allowed in ES3. One of the consequences of switching the engine into strict mode is that it'll throw more errors in situations where it would otherwise have simply silently failed/ignored the error. Assigning an arbitrary property to an object that has been marked as not being extensible via preventExtensions for instance causes an exception to be thrown. Here's the updated profit computation snippet using strict mode and preventExtensions. Take note of the lines highlighted in bold.

```
"use strict"; // make engine switch into strict mode

var data = [
    {
        month: 1,
        revenue: 102010,
        expense: 95000
    },

    {
        month: 2,
        revenue: 143232,
        expense: 98000
    },

    {
        month: 3,
        revenue: 323212,
        expense: 195000
    }
];

function Results() {
    this.totalRevenue = 0;
    this.totalExpense = 0;
    this.netProfit = 0;
}

var r = new Results();
Object.preventExtensions(r); // disallow arbitrary extensibility

//
// iterate through the data set and compute total
// revenue and expense
//
data.forEach(function(d) {
    r.totalrevenue += d.revenue;
    r.totalExpense += d.expense;
});

//
// net profit is revenue - expense
//
r.netProfit = r.totalRevenue - r.totalExpense;
print(r.netProfit);
```
And here's the output we get on running the script in a browser that can understand strict modes (IE10 Platform Preview 1 in this case; but recent versions of Chrome, FireFox, Safari and Opera should be able to handle it as well).

```
Cannot create property for a non-extensible object
```

As you can tell, an exception was thrown upon encountering the line where an attempt was made to create a new property on the object r.

###Object.seal

Object.seal is a superset of Object.preventExtensions in functionality in that while it prevents tacking on of arbitrary properties to objects it also prevents altering the attributes of the properties that already exist on the object. Also, it disallows deleting properties. First we'll define a little helper function that will allow us to enumerate and print all the "own" properties (i.e. properties defined directly on the object instead of its prototype) that are present on a given object which we'll use to examine the object's property state as we go along.

```
//
// prints the property descriptors for all the "own"
// properties on the object "o"
//
function printDesc(o) {
    Object.getOwnPropertyNames(o).forEach(function(p) {
        print(sprintf("%s: %s", p,
            JSON.stringify(Object.getOwnPropertyDescriptor(o, p),
            null, " ")));
    });
}
```

With this function in hand let's go ahead and define a simple object with a single property called "name" using Object.defineProperty and print it to the console via printDesc.

```
//
// define object with one property
//
var person = {};
Object.defineProperty(person, "name", {
    value: "no name",
    writable: true,
    enumerable: true,
    configurable: true
});
print("Default state:");
printDesc(person);
```

Here's the output we get on running this:

```
Default state:
name: {
  "value": "no name",
  "writable": true,
  "enumerable": true,
  "configurable": true
}
```

Let's next examine the effect calling Object.preventExtensions has on the attribute state:

```
//
// prevent extensions
//
Object.preventExtensions(person);
print("preventExtensions:");
printDesc(person);
```

Output:

```
preventExtensions:
name: {
  "value": "no name",
  "writable": true,
  "enumerable": true,
  "configurable": true
}
```

As you can tell nothing at all has changed on the attributes of the property name. This makes sense as the only effect that preventExtensions has on the object is to make it non-extensible. Let's next attempt to edit the property descriptor for the property name and see what happens:

```
//
// manually change property descriptor
//
Object.defineProperty(person, "name", {
    value: "no name",
    writable: false,    // make property read only
    enumerable: false,  // disallow enumeration
    configurable: true  // allow editing prop descriptor
});
print("Manual config reset:");
printDesc(person);
```

Output:

```
Manual config reset:
name: {
  "value": "no name",
  "writable": false,
  "enumerable": false,
  "configurable": true
}
```
So far everything seems to be working as expected. Let's continue the experiment and seal the object and see what happens.

```
//
// seal
//
Object.seal(person);
print("seal:");
printDesc(person);
```

Output:

```
seal:
name: {
  "value": "no name",
  "writable": false,
  "enumerable": false,
  "configurable": false <-- this changed!
}
```

Sealing the object seems to have had the effect of setting the configurable attribute on the property descriptor to false. So what happens when we attempt to delete a property or tack on a new property?

```
//
// delete property "name"
// add a new property
//
delete person.name;
person.age = 10;
print("Delete/add properties:");
printDesc(person);
```

Output:

```
Delete/add properties:
name: {
  "value": "no name",
  "writable": false,
  "enumerable": false,
  "configurable": false
}
```

As you can tell, nothing happened! Calling Object.seal on person causes the JavaScript engine to ignore "delete" calls and attempts to introduce new properties. In strict mode both of these attempts would have caused an error to be thrown. Finally, let's try and redefine the property descriptor on the sealed instance of person and see what happens.

```
//
// manually change property descriptor again
//
Object.defineProperty(person, "name", {
    value: "no name",
    writable: true,
    enumerable: true,
    configurble: true
});
print("Second manual config reset:");
printDesc(person);
```

Output:

```
Cannot redefine non-configurable property 'name'

```

The JavaScript engine throws an error indicating that this operation is not allowed on a sealed object. Note that this error is thrown even when we are not running in strict mode. You can check if an object has been sealed by calling Object.isSealed which returns true if the object has been sealed and false otherwise.

###Object.freeze

Object.freeze is a superset of both Object.seal and Object.preventExtensions in that it subsumes the functionality of both of those functions with the added responsibility of making all of the existing properties on the object read-only. Here's an example:

```
var ShapeType = {
    None: 0,
    Line: 1,
    Ellipse: 2,
    Rectangle: 3,
    RoundedRectangle: 4
};

Object.freeze(ShapeType);

ShapeType.None = 5;
print(ShapeType.None);

ShapeType.Polygon = 5;
print(typeof(ShapeType.Polygon));
```

Output:

```
0
undefined
```
Once an object has been frozen attempts to assign a value to an existing property or to define a new property are silently ignored in normal mode. In strict mode this would cause the engine to throw an error. To determine if an object has been frozen call Object.isFrozen.

Here's a table that summarizes the functionalities of preventExtensions, seal and freeze taken from MSDN:

Function | Object made non-extensible | "configurable" set to false for all properties | "writable" set to false for all properties
------------ | ------------- | ------------
Object.preventExtensions | Yes  | Yes| No
Object.seal | Yes | Yes | No
Object.freeze | Yes | Yes | Yes

In summary therefore, preventExtensions, seal and freeze allow you to exercise very precise control on how extensible or modifiable your objects are going to be. In combination with "strict mode" these new enhancements to JavaScript should help developers catch errors much earlier in the development cycle. And the fact that these capabilities are provided through new methods on the Object type should make it trivial to gracefully degrade when your site runs on older browsers that do not have support for ES5 features. In short there's no reason why you should not start using these features in your web apps from, like, right now!

PS: <http://blogorama.nerdworks.in/preventextensionssealandfreeze/>

