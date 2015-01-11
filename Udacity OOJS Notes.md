#Udacity OOJS Notes

##Clojures
- Declare global array to keep track of functions even after they have been returned in a lexical scope.

```
var sagas = [];
var hero = newHero();

var newSaga = function(){
	var foil = newFoil();
	sagas.push(function(){  // Push will store a function object to the sagas array which will persist.
		var deed = newDeed();
		log(deed + log + foil);
	});
}
newSaga();

	/* 
		Context of a function will always be created as a child of the context it was defined within 
	*/
sagas[0]; // Logs "BoyEyesRat"

	/* 
		Creates new context in NEWSAGA1 context ... new deed var will be created 
	*/
sagas[0]; // Logs "BoyDigsRat"

	/* 
		Creates new function to be pushed to sagas array in a new NEWSAGA2 context 
	*/
newSaga(); 

	/* 
		Will still make lookup to deed var in NEWSAGA1 context 
	*/
sagas[0]; // Logs "BoyPinsRat"

	/* 
		Will make lookup to deed var in NEWSAGA2 context 
	*/
sagas[1]; // Logs "BoyGetsET"

	/*
		DOES NOT CREATE NEW CONTEXT. uses SAGAS1.3 Context since sagas[1] has been called
	*/
sagas[0]; // Logs "BoyPinsRat"

```
####Mock Compiler 
```
####MAIN 
    sagas = [{f}, {f}]
    hero = "Boy"
    newSaga = {f}
    ####NEWSAGA1
        foil = "Rat"
        ####SAGAS1.1
			deed = "Eyes"
		####SAGAS1.2
			deed = "Digs"
		####sAGAS1.3
			deed = "Pins"
	####NEWSAGA2
		foil = "ET"
		####SAGAS2.1
			deed = "Gets"
```
##'this' Keyword
- this will be automatically bound to the correct object automatically
- How the interpreter knows which binding is correct, resembles the rules for positional function parameters

####What it DOESN'T map to
```	
var obj = { // .. the object created by the literal this appears within
	fn = function(a,b){ // .. the fn this appears within
		log(this);
	}
};

var obj2 = {method: obj.fn}; // .. an object that happens to have that fn as a property

obj.fn(3,4); // .. an "execution context"

```

####What it DOES map to
```
obj.fn(3,4); // The object to the left of the dot (different from the literal definition!)
```

####Examples
```
var fn = function(one, two){
	log(this, one, two);
};

var r = {}, g = {}, b = {}, y = {};
r.method = fn;

/* 
	this becomes the object fn is bound to 
*/
r.method(g,b) // r{}, g{}, b{}

/* 
	this becomes the global scope fn is within 
*/
fn(g,b)       // <global>, g{}, b{}

/* 
	.call overrides the <global> reference and sets it to the first param *
/ 
fn.call(r,g,b) // r{}, g{}, b{}

/* 
	.call also overrides the method access rules, will be bound to y 
*/
r.method.call(y,g,b) // y{}, g{}, b{}

//USE IN CALLBACKS
/* 
	fn is forced to be invoked without parameters in setTimeout implementation. setTimeout has no way of knowing the input params here
*/
setTimeout(fn, 1000) // <global>, undefined, undefined 

/* 
	Same as above 
*/
setTimeout(r.method, 1000) // <global>, undefined, undefined

/* 
	Define a function without params (since it will be invoked without params anyway) and have it contain the params you desire.
*/
setTimeout(function(){  // r{}, g{}, b{}
	r.method(g,b); 	
})

//Mock Implementation
var setTimeout = function(cb, ms){
	arbitraryDelay(ms);
	cb();
}

log(this) // <global>

new r.method(g,b) // brand new object
```

##Prototype Chains
Can copy an object to another object s.t. second object has all properties first one has. OR Can make new object behave like the original by delegating failed lookups to the original one.

**Why?**
To save memory/ share code.

```
var gold  = {a:1};
log(gold.a); // 1
log(gold.z); // undef

/* This is a ONE-TIME PROPERTY COPY */
var blue = extend({},gold); // Assume there is for loop copying the values of gold to blue...
log(blue.z); // undef

/* this is ONGOING LOOKUP-TIME DELEGATION */ 
var rose = Object.create(gold); //Constantly uses gold as "fallback" for property checking
rose.b = 2;
log(rose.a) // 1
log(rose.z) // undef

gold.z = 3;

log(blue.z); // undef
log(rose.z); // 3

```
####The chain
...Rose has a fallback too, the Object prototype. All objects in JS have a top level prototype to fall back to, the **Object** prototype. Arrays have an independant array prototype, which is delegated to by the Object prototype.

```

rose.toString(); //toString() is a top level function that all JS objects have

var array1 = [];
array1.constructor() = Array

var obj1 = {};
obj1.constructor() = Object

/*The two vars above have their own constructors dependant on their prorotype! */

```

##Object Decorator Pattern

Reusing code!

```
//run.js
var amy = {loc:1};
amy.loc++;
var ben = {loc:9};
ben.loc++;

//library.js
var move = function(car){ // This is a decorator pattern
	car.loc++;
};

//run.js
var amy = {loc:1};
move(amy);
var ben = {loc.9};
move(ben);

//library.js
var carlike = function(obj, loc){ // This is a decorator pattern
	obj.loc = loc;
	return obj;
};

var move = function(car){
	car.loc++;
};

//run.js
var amy = carlike({}, 1);
move(amy);
var ben = carlike({}, 9);
move(ben);

//library.js
var carlike = function(obj, loc){ // This is a decorator pattern
	obj.loc = loc;
	obj.move = function(){ // New closure scope is created
		obj.loc++;
	};
	return obj;
};

//run.js
var amy = carlike({}, 1);
amy.move;
var ben = carlike({}, 9);
ben.move;
```

Abstract out code because
1. Less code
2. Faster refactoring
