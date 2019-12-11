---
layout: post
title: Comprehensive Guide to Javavscript Modules
date: 2019-12-11
categories: javascript
---

In this guide I'm going to point out all the aspect details of modules in javascript starting with why we need modules at all? (what is the purpose they are actually fill?) – and from this point I'll describe extensively the details of ES6 modules.
Here is the list of the titles of topics:
* Why we need modules at all? (What is the purpose they are actually fill?)
* Definition of modules
* The ES6 Modules
 * Digging more on ES6 Modules
 * Using Default Exports
 * Import a module for its side effects only
 * Conventional ES6 modules (those discussed till now) are static by nature – i.e. the browser downloads the script files at compile time (upon loading the page),- as opposed to function calls, or any code inside block (like if statements for instance),  that are executed at run time.
 * Dynamic Import in ES6 Modules
 * Lazy Loading in action (code example)
 * Implementing the popular MVC design pattern with ES6 Modules
 * Using Aggregation Module
  *	Aggregating to object
 * Using external libraries (third-party es6 module-based packages)
 * ES6 module pattern is different from the CommonJS module pattern regarding to relative path resolution and other fundamental characteristics
   * Relative path resolution that conforms both to the ES6 module pattern (client-side js in general or html – in short: any client-side) and the CommonJS module pattern
   * Client-side (and it includes of course the case of the ES6 module pattern) relative path resolution
   * CommonJS module pattern relative path resolution
   * Module Bundlers realizes only the CommonJS module pattern relative path resolution
* Module Loaders – What are they and do we still need them?
* Module Bundlers
 * Using module bundler solely for the purpose of creating bundled file to be deployed in the production environment (the web server)
 * Using module bundler in development stage
 * Tree Shaking
 * Lazy loading with module bundlers

## Why we need modules at all? (What is the purpose they are actually fill?)
To understand the need for modules, let us do some coding. Create some html file (I called this file "index.without-modules.html"), and put this content in it:
```html
<!DOCTYPE html>
<html>
  <head>
	  <meta charset="UTF-8">
   <meta name="viewport" content="width=device-width, initial-scale=1">
	  <title>Example of ES6 Classes</title>
  </head>	
  <body>
    <div id="animal-container"></div>
  
    <script src="src/without-modules/owner.js"></script>
    <script src="src/without-modules/animal.js"></script>
    <script src="src/without-modules/snake.js"></script>
    <script src="src/without-modules/horse.js"></script>
    <script src="src/without-modules/main.js"></script>
  </body>
</html>
```
Now create the files mentioned in the script tags add to each of the some console.log messages and run the html file to verify their loads.

Now that everything loads properly make the contents of the script tags to be as follows:

**src/without-modules/owner.js**
```javascript
class Owner {
    constructor(name, address) {
        this.name = name;
        this.address = address;
    }
}
```

**src/without-modules/animal.js**
```javascript
class Animal {
    constructor(theName, domElement, ownerName, ownerAddress) {
        this.name = theName;
        this.totalDistance = 0;
        this.parentElement = domElement;
        this.owner = new Owner(ownerName, ownerAddress);
    }
    move(distanceInMeter = 0) {
        console.log(`${this.name} moved ${distanceInMeter}m.`);
        this.totalDistance += distanceInMeter;
    }
    display() {
        let containerDiv = document.getElementById(this.parentElement);
        let animalDiv = document.createElement("div");
        containerDiv.appendChild(animalDiv);
        animalDiv.innerText = "'" + this.name + "' total moves distance was " + this.totalDistance + "m.";
        animalDiv.innerText += "\nThe registreted owner is '" + this.owner.name + "' and he lives in " + this.owner.address + ".";
        animalDiv.innerHTML += "<hr>";
    }
}
```

**src/without-modules/snake.js**
```javascript
class Snake extends Animal {
    constructor(...args) {
        super(...args);
        console.log("Slithering...!");
    }
}
src/without-modules/horse.js
class Horse extends Animal {
    constructor(...args) {
        super(...args);
        console.log("Galopping...!");
    }
}
```

**src/without-modules/main.js**
```javascript
let sam = new Horse("Sammy the Pony", "animal-container", "Koko", "New York");
sam.move(47);
sam.move(15);
sam.display();

let ben = new Snake("Benny the Python", "animal-container", "Moko", "Boston");
ben.move(0.4);
ben.move(0.3);
ben.display();
```

Now run the html file to see the details are presenting properly in the page and also open the console to see that everything presented properly also there.

Although this works fine there are some downsides with this approach:
* __The need to include all those html script tags in the html file__ - in our example we have five script tags, though it is severe enough, you can guess the nightmare in structured project that split to hundreds of script files that obviously all of them needs to be load via script tags – it's really nightmare!
* __The need to include all those html script tags in the html file *in the right order*__ – order matters! In our example we must first include the owner file and only after this we can include the animal file that refers in its code to the owner, and only after this we can include the horse and snake files that refers in their code to the animal file (they are classes that extends the animal base class), and only at the end we can include the main file that responsible in our example for the end result display. Any change mistakably made to this order will cause everything to collapse!
* __Encapsulation is not supported__ – Here in our example we have the Owner (in the owner.js file) that is meant to be used only inside the Animal (in the animal.js file) i.e  Animal instance owns Owner instance (see in the Animal constructor code that it creates new Owner instance). Nonetheless here the end user of our 'Animal' application must to be informed also about the Owner to include it also in script tag (before the script tag for the animal), and also the end user have direct access to the Owner,- though the end user needs to be familiar with the Animal only and should not access the Owner directly (our intention, again and as you can see in the code, is to have Animal instances that each of them builds Owner instance that it owns). So here though we need encapsulation, i.e. to encapsulate the Owner inside the Animal instance, thus exposing only the Animal for the end-user to bring up, here this is not achieved,- add new line in the main.js file: ```console.log(Owner);``` - you'll see of course in the console the Owner class details,- class that intended to be hidden (internal class) but it is not,- more than this: the end-user must to bring up himself the Owner and he cannot settle with bringing up the Animal only.

To mitigate the downsides mentioned for this approach, the common practice is to use some task runner (grunt or gulp or some other task runner), that will be in charge to concatenate (and typically also minify) the files to single file (we need to verify that it does the concatenation in the right order, because order matters!). Third-party javascript packages (by the way: some uses the term 'libraries' instead of 'packages'), are practically interested with this practice as they can offer to the end-user one file to download. (Others, like jquery, does not make a use of multiple files rather they use one file all along. In jquery for instance it is one file with one object that wraps in it all the methods exposed to the end user). Nonetheless this approach is not the elegant way, nor complete solution, for dealing with the downsides mentioned, __so really it dictates us to come up with idea to download script files via javascript code rather than loading script via html script tag – and that what the modules is all about.__
## Definition of modules
> __*Modules are script files downloaded via javascript code (rather than the old tradition script loading methodology: loading via html script tag).*__

Script files downloaded via javascript maintain their own global scope (rather than sharing the same global scope, as it is when we download script files with the old fashion script tag). That means that accessing the scope identifiers (variables, methods) of the script files entails special mechanism of exporting them explicitly (we'll explain latter how we do this). 
Bootstrap this downloading methodology entails loading one initial script file, called __entry file__, with the old fashion script tag, nonetheless in the same time, we need to find a way to tell the compiler to keep track for script file(s) - 'instructions to download' - (we'll explain latter how we tell the compiler this information). Keeping the track for script file(s) download instructions via javascript code is done recursively, i.e. tracking the code in the initial script file (the entry file) which instructs what file(s) needs to be downloaded, download them, keep track also on those files for code which instructs what file(s) needs to be downloaded, and so forth.
## The ES6 Modules
Let's take our example to use ES6 modules.

First of all we'll need web server - you can't run ES6 modules via a file://<URL> because it will not run and you'll get error - try it and open your browser console to see the error message (this errors known as CORS type errors – the same type of error you'll get when you try to run code with AJAX via a file://<URL> - or alternatively even via web server, when trying to do AJAX call to file located not in your domain). 

The html file content needs now to be with this content in it:
```html
<!DOCTYPE html>
<html>
  <head>
     <meta charset="UTF-8">
     <meta name="viewport" content="width=device-width, initial-scale=1">
     <title>Example of ES6 Modules</title>
  </head>	
  <body>
    <div id="animal-container"></div>
    
    <script src="src/with-modules/main.js" type="module"></script>
  </body>
</html>
```

**src/with-modules/owner.js**
```javascript
export class Owner {
    constructor(name, address) {
        this.name = name;
        this.address = address;
    }
}
```

**src/with-modules/animal.js**
```javascript
import { Owner } from "./owner.js";
export class Animal {
    constructor(theName, domElement, ownerName, ownerAddress) {
        this.name = theName;
        this.totalDistance = 0;
        this.parentElement = domElement;
        this.owner = new Owner(ownerName, ownerAddress);
    }
    move(distanceInMeter = 0) {
        console.log(`${this.name} moved ${distanceInMeter}m.`);
        this.totalDistance += distanceInMeter;
    }
    display() {
        let containerDiv = document.getElementById(this.parentElement);
        let animalDiv = document.createElement("div");
        containerDiv.appendChild(animalDiv);
        animalDiv.innerText = "'" + this.name + "' total moves distance was " + this.totalDistance + "m.";
        animalDiv.innerText += "\nThe registreted owner is '" + this.owner.name + "' and he lives in " + this.owner.address + ".";
        animalDiv.innerHTML += "<hr>";
    }
}
```

**src/with-modules/snake.js**
```javascript
import { Animal } from "./animal.js";
export class Snake extends Animal {
   constructor(...args) {
        super(...args);
        console.log("Slithering...!");
   }
}
```

**src/with-modules/horse.js**
```javascript
import { Animal } from "./animal.js";
export class Horse extends Animal {
   constructor(...args) {
        super(...args);
        console.log("Galopping...!");
   }
}
```

**src/with-modules/main.js**
```javascript
import { Snake } from "./snake.js";
import { Horse } from "./horse.js";

let sam = new Horse("Sammy the Pony", "animal-container", "Koko", "New York");
sam.move(47);
sam.move(15);
sam.display();

let ben = new Snake("Benny the Python", "animal-container", "Moko", "Boston");
ben.move(0.4);
ben.move(0.3);
ben.display();
```

Again don't forget to run this code from web host (using http web server) – otherwise you'll get error.

To sum up the changes we have here:
* In the html file now we include only one script tag – script tag for our entry file (main.js): ```<script src="src/with-modules/main.js" type="module"></script>``` - Pay notice also to the attribute type="module" – it won't work without this because we need to tell the browser compiler to track for script file(s) download instructions via javascript code.
* The import code lines that instruct the browser compiler to download the script file, for example this line: ```import { Snake } from "./snake.js";``` - Pay notice to { Snake } - In the same time we instruct to download the script file we are also bringing the identifier exported in snake.js to our scope,- only then we can use the Snake class identifier in our global scope. Of course the Snake class identifier had to be exported in snake.js: ```export class Snake …```
* The scoping we mentioned in the earlier point has another positive aspect we need to appreciate – I call it encapsulation (though it is not the exact terminology to use) – You can see that our main.js imports to it scope only the Snake and Horse and nothing else, so the only decedents of main.js (our entry file) are the Snake and Horse that are in the bottom level of our application tree,- that is almost always the practice because naturally the entry file needs access to those members in the bottom level whereas those higher level in the application tree,– in our example it is the Animal and the Owner,– are considered to be internals intended to be used in some member of some lower level.
## Digging more on ES6 Modules
Here are some more details concerning ES6 Modules:
* We can bring to our scope multiple identifiers exported in the same file with one command, separating them with comma. In our example we can do the follows:
  * In the horse.js file add this line: ```export let numberOfHorseBreeds = 600;``` and in the main.js file change the line ```import { Horse } from "./horse.js";``` to: ```import { Horse, numberOfHorseBreeds } from "./horse.js";``` - Now you have access to numberOfHorseBreeds (log it to the console to verify).
* Continuing with the previous example added just before – add now this line (to the main.js file): ```numberOfHorseBreeds = 602;``` - You'll get an error (see this in the browser console). ES6 modules exports are frozen, i.e: they are read-only, also worth to note that when you use modules (script tags with type="module") you are implicity using strict mode,- thus the following line causing the produce of 'TypeError'.
* Though ES6 module exports are read-only the inner properties are NOT read-only (it is 'shallow freeze'),- thus the follow lines will work: ```sam.name = "Fred the Mustang"; sam.totalDistance = 701; sam.display();```
* We can even add new properties. i.e: this will also work ```sam.age = 8; console.log(sam);``` sam will include also the new 'age' added
* Import statements are hoisted. Add this line to the file horse.js ```export let classification = { trinomialName: "Equus ferus caballus", lifeExpectancy: "25-30" };``` - Now add in main.js this: ```console.log(classification); import { classification } from "./horse.js";``` - Examine this and you'll see that import statement are hoisted,- thus we can call console log to classification even though the import of classification had made later in the code. So placing import statements in the top of the file are nothing more than readability conventions!
* You cannot import to the same identifier scope twice – so in our example the following line will produce error because numberOfHorseBreeds already imported in previous line we made: ```import { numberOfHorseBreeds } from "./horse.js";```
* Add those lines to horse.js – we'll use those lines later on: ```export let insight = "Mule is not horse!"; export let popularBreeds = ["Arab", "Pony", "Palomino"];```
* I can import the entire module to some namespace. In our example we can do this (in main.js) to import the entire horse.js exports to namespace called 'myModule': ```import * as myModule from "./horse.js"; console.log(myModule.insight);```
* ... and import the same again to some other namespace (here againMyModule): ```import * as againMyModule from "./horse.js"; console.log(againMyModule.insight);```
* Again: though ES6 module exports are read-only the inner properties are NOT read-only (it is 'shallow freeze'),- thus we can assign new value to classification.lifeExpectancy. Add those lines to verify this: ```classification.lifeExpectancy = "15-20"; console.log(classification.lifeExpectancy);```
* Again: ES6 modules exports are frozen so we cannot assign new values to import identifiers. The follows will produce an error: ```classification = "new value";```
* We cannot delete import identifier. The follows will produce an error: ```delete classification;```
* Arrays are object, so same as in inner properties in objects, the elements in array are NOT read-only. ```javascript import { popularBreeds } from "./horse.js"; popularBreeds.push("Mustang"); popularBreeds[0] = "Arabic"; popularBreeds.splice(2, 1); console.log(popularBreeds); ```
* The follows will cause an error that will crash the application since we haven't created file called 'hurse.js'. In Firefox we'll get this error message: ```Loading module from “http://<PATH>/hurse.js” was blocked because of a disallowed MIME type (“text/html”).``` In chrome we'll get this error message: ```Failed to load resource: the server responded with a status of 404 (Not Found).``` - Both browsers will not give us indication about the code place (file line code number) causing this error! ```javascript import * as myHurse from "./hurse.js"; ```
* You can also import the entry file (or any other javascript file) inside the script tag (aka inline), like this: ```html <script type="module">import "/src/with-modules/main.js";</script>```
* There are much more detail points to discuss concerning to ES6 Modules – for each one of them I am going to dedicate topic of its own – so keep on reading!

