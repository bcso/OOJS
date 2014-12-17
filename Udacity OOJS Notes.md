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







