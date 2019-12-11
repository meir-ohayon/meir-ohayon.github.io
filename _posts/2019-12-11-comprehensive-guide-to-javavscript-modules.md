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

### Digging more on ES6 Modules
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
* Arrays are object, so same as in inner properties in objects, the elements in array are NOT read-only. ```import { popularBreeds } from "./horse.js"; popularBreeds.push("Mustang"); popularBreeds[0] = "Arabic"; popularBreeds.splice(2, 1); console.log(popularBreeds); ```
* The follows will cause an error that will crash the application since we haven't created file called 'hurse.js'. In Firefox we'll get this error message: ```Loading module from “http://<PATH>/hurse.js” was blocked because of a disallowed MIME type (“text/html”).``` In chrome we'll get this error message: ```Failed to load resource: the server responded with a status of 404 (Not Found).``` - Both browsers will not give us indication about the code place (file line code number) causing this error! ```javascript import * as myHurse from "./hurse.js"; ```
* You can also import the entry file (or any other javascript file) inside the script tag (aka inline), like this: ```<script type="module">import "/src/with-modules/main.js";</script>```
* There are much more detail points to discuss concerning to ES6 Modules – for each one of them I am going to dedicate topic of its own – so keep on reading!

### Using Default Exports
There are two different types of export, "named" and "default". Until now we use in our tutorial only the "named" type of export (that is more commonly used by myself), nonetheless sometimes developers prefers the make a use of the "default" export type. Instead of explaining in words what is the default export type and how it differs from the named type, I think it can be best learned by exploring example code, so let's jump ahead directly to code. Create new file called some-module-to-test-with.js and put in it this code:
```javascript
let someVar = "kkk";
export { someVar as default };
```
Now add those lines in our entry file (main.js):
```javascript
import someVar from "./some-module-to-test-with.js";
console.log(someVar);
```
Here are more things needs to be pointed:
* This code line is equal alternative to line export { someVar as default } we've used earlier: ```export default someVar;```
* Only one default export can be in a module so this will produce syntax error: ```let someAnotherVar = "yyy"; export { someAnotherVar as default };```
* Default export can be imported with any name - Do this to import the same someVar that default exported in the module file with the name someVar, now to import it with the name sVar ! ```import sVar from "./some-module-to-test-with.js"; console.log(sVar);```
* We can even export default the value itself, like this: ```export default "kkkk";```
* We can export default named/unamed function and invoke the function in the file that imports. For example use this line for the export: ```export default () => "fffff";``` And this for the import: ```import sVar from "./some-module-to-test-with.js"; console.log(sVar());```
* Default export can be also used in named/unamed class (try it).
* Default export can be also used in named/unamed generator function (aka function*).

### Import a module for its side effects only
We can import a module for its side effects only, without importing anything, i.e: only import module file for loading it to run its execution code.

To demonstrate this create new file called rounded-frame.js and copy to it this code:
```javascript
/*
Pay attention that there is no export in the next two lines of code - those lines, like any other code placed in this module file that does not need ANY kind of interaction from the file that imports it, will be executed by importing this file for its side effects only, i.e: import "/<PATH-FOR-THIS-FILE>/rounded-frame.js";
*/
let animalContainer = document.getElementById("animal-container");
animalContainer.style.cssText = "border: 2px solid red; border-radius: 33px; padding: 45px; width: 450px;";
```
As you can see the intention of this code lines is to add rounded red border around our whole animal display.

Please read also the important comment lines in the head explaining why we are not doing any export over here.

Now let's do the actual import. Add this line of code in our main.js file to execute the global code in the rounded-frame.js file (of course save and refresh the page to see the lovely border it adds): 
```javascript
import "./rounded-frame.js";
```
__Note:__ If you need some kind of interaction with this module, you cannot use the above approach we called 'side effects only import', rather you need to use the regular export techniques, and that includes ANY kind of interaction needed from the file that imports it: access to identifiers (to get them or set them), execute function code, sending options (parameters).

So if we'll continue the above example by adding the ability to set the border color from the file that imports this module file,- we do that using export of wrapper function that gets the color as parameter. Here is the export code (in rounded-frame.js) needed for this:
```javascript
export default function(borderColor) {
    animalContainer.style.cssText = "border: 2px solid " + borderColor + "; border-radius: 33px; padding: 45px; width: 450px;";
}
```
And here the complementary import code (in main.js):
```javascript
import m from "./rounded-frame.js";
m("blue");
```

### Conventional ES6 modules (those discussed till now) are static by nature – i.e. the browser downloads the script files at compile time (upon loading the page),- as opposed to function calls, or any code inside block (like if statements for instance),  that are executed at run time.
I guess that you notice, as a seasoned javascript developer (and if not, be noticed now about this), that function code are not interpreted upon the loading of the page, thus no complains about compile errors being made will be displayed,- that until you execute the function (by calling it), the same goes for any code inside block,- try for instance some syntax error inside function not being called in your code, and you'll see that it will pass silently, the same goes if you move this code inside block code not going to be called (like inside if block section that never called, e.g. like inside if (1 == 0) ). So we can say that code inside block statements are said to be called "dynamic" (and that include functions, if statements, while statements, for statements, etc. – really any code inside block code that is not intended to be run from the global top level scope), while other code that is executed at the global top level scope is said to be called "static".

ES6 modules we've done so far in this tutorial are "static" by nature and thus can be executed only at the global top level scope only, the follows demonstrates practically the meanings of this fact:
* The follow lines will produce  syntaxError because the import is inside function: ```function displayInsight() {  import { insight } from "./horse.js";  console.log(insight); } displayInsight();```
* Also we cannot load module conditionally. ```let needsInsights = true; if (needsInsights) {  import { insight } from "./horse.js";  console.log(insight); }```
* Also we cannot import in a try/catch block, so there is no error recovery for import errors. ```try {  import { insight } from "./horse.js";  console.log(insight);} catch {  console.log('failed'); }```

### Dynamic Import in ES6 Modules
There is a new way to import module dynamically using asynchronous function syntax. 

This import function returns a promise and it is known as "dynamic import". 

"Dynamic import" is a stage 3 proposal and not implemented yet in IE at any version (even not the Edge). 

Dynamic import imports the entire module file asynchronously, and at least for now does not enables us to import specific export identifier,- i.e. here for example I cannot import only 'insight',- only the entire horse.js module file can be imported dynamically,- thus the scope is equivalent to the follows static form: import * as myModule from "./horse.js" (here we'll use though the dynamic import not the regular static syntax we done thus far).
```javascript
import("./horse.js").then(m => console.log(m.insight));
```
Of course dynamic import enables us to import module inside function or to conditionally import module. Here we demonstrate both: import module inside function that also conditionally imports the module.
```javascript
var needsInsights = true;
function displayInsight() {
  if (needsInsights) {
    import("./horse.js").then(mf => console.log(mf.insight));
  }
}
displayInsight();
```
One of the advantages of dynamic import is that it enables us error recovery using catch block to catch any kind of module loading error. So here for example we have misspelled the file name (hurse.js instead of horse.js), and the catch block will run and will log to the console the error message.
```javascript
import("./hurse.js").then(me => console.log(me.insight)).catch(err => console.log(err.message));
```
The most important advantage of dynamic import is that it enables us doing 'lazy loading' of modules - i.e. loading modules at runtime based on aroused needs determined in runtime - so for example if we have in our application some modules that are not commonly design to be in any need but only in some special somewhat rare circumstances, we can now, curtsy to the dynamic import, import those modules at runtime only on the arise of one of one those circumstances - this thing is called "dynamic import" and we are going also to demonstrate it later with specific example (in brand new dedicated code example in a brand new file we'll create later on – right in next!).

### Lazy Loading in action (code example)
Make html file with this content in it:
```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Example of ES6 Modules Lazy Loading</title>
  </head>	
  <body>
    <div id="animal-container"></div>
    
    <script src="src/with-modules/using-lazy-loading/test-lazy-loading.js" type="module"></script>
  </body>
</html>
```
**src/with-modules/using-lazy-loading/test-lazy-loading.js**
```javascript
/*
Pay attention that we've chosen to make this file the entry file rather than making the main.js directly the entry file, just for the reason that we can…
*/ 
import "./main.js";
```

**src/with-modules/using-lazy-loading/top-nav.js**
```javascript
let navElement = document.createElement("nav");
let navAnchorStyle = "text-decoration: none; background-color: #c6bcc6; padding: 5px 10px; border-radius: 4px; margin-right: 10px;";
navElement.innerHTML = "<a href='snake.html' style='" + navAnchorStyle + "'" + "data-animal-module='snake'>Snakes</a>";
navElement.innerHTML += "<a href='horse.html' style='" + navAnchorStyle + "'" + "data-animal-module='horse'>Horses</a>";

let body = document.querySelector("body");
navElement.style.cssText = "margin-bottom: 20px; display: inline-block;";
body.prepend(navElement);
```

**src/with-modules/using-lazy-loading/main.js**
```javascript
document.getElementById("animal-container").innerText = "Click on the top menu buttons to choose animal type.";
import "./top-nav.js";
var selectedType = "";
for (const link of document.querySelectorAll("nav > a")) {
    link.addEventListener("click", e => {
        e.preventDefault();
        document.getElementById("animal-container").innerText = "";
        selectedType = link.dataset.animalModule;
        document.querySelectorAll("nav > a").forEach(el => el.style.fontWeight = "normal");
        e.target.style.fontWeight = "bold";
        import(`../${selectedType}.js`).then(m => {
            import("../rounded-frame.js");
            switch (selectedType) {
                case "horse":
                    let sam = new m.Horse("Sammy the Pony", "animal-container", "Koko", "New York");
                    sam.move(647);
                    sam.move(315);
                    sam.display();
                    let fred = new m.Horse("Fred the Mule", "animal-container", "Moko", "Boston");
                    fred.move(500);
                    fred.display();
                    
                    break;
                case "snake":
                    let ben = new m.Snake("Benny the Python", "animal-container", "Moko", "Boston");
                    ben.move(12);
                    ben.move(5.3);
                    ben.display();
                    let ron = new m.Snake("Ron the Cobra", "animal-container", "Koko", "New York");
                    ron.move(10.1);
                    ron.display();
  
                    break;
                default:
                    document.getElementById("animal-container").innerText = "Not available animal type";
            }
        }).catch(err => document.getElementById("animal-container").textContent = err.message);
    });
}
```

**Explanations for this code:**
* We import the top-nav.js file that makes navigation bar on the top of the page with two menu items, one is Snakes and the other is Horses. When you'll first load the html in the browser you'll see this top navigation bar and bellow it a message "Click on the top menu buttons to choose animal type." Open the browser console, switch to the "Network" tab and refresh the page to verify that the horse.js and snake.js haven't been loaded yet!
* The next lines adds event listener to the menu items click event to make them, loads their respective js file. 
With the "Network" tab active in the browser console click on the "Snake" menu item to see that it causes the load of the snake.js file and changes the message bellow to a rounded frame containing our snakes ("Benny the Python" and "Ron the Cobra"). Now continue to click the "Horse" menu item to see that it causes the load of the horse.js file (and changes the rounded frame content respectively).
* We can improve more our code to make it more prettier/structured – we'll do those improvement suggestions right on next.

#### src/with-modules/using-lazy-loading/main.js : Improvement 1
The demo we've done implements the initialization and display of the horses/snakes, directly inside the click event (inside the switch statement that determines what have been clicked). We can do our code more prettier/structured by moving this code to outside functions and passing them the module object as parameter,- this is how we do it:
```javascript
…
   switch (selectedType) {
                case "horse":
                  displayHorses(m);
                  break;
                case "snake":
                  displaySnakes(m);
                  break;
               …
   }
…
function displayHorses(horseModulePromise) {
    let sam = new horseModulePromise.Horse("Sammy the Pony", "animal-container", "Koko", "New York");
    sam.move(647);
    sam.move(315);
    sam.display();
    let fred = new horseModulePromise.Horse("Fred the Mule", "animal-container", "Moko", "Boston");
    fred.move(500);
    fred.display();
}

function displaySnakes(snakeModulePromise) {
    let ben = new snakeModulePromise.Snake("Benny the Python", "animal-container", "Moko", "Boston");
    ben.move(12);
    ben.move(5.3);
    ben.display();
    let ron = new snakeModulePromise.Snake("Ron the Cobra", "animal-container", "Koko", "New York");
    ron.move(10.1);
    ron.display();
}
```

#### src/with-modules/using-lazy-loading/main.js : Improvement 2
We can also do our code more prettier/structured by moving this code instead to outside modules passing the module object as parameter to their methods (i.e. we'll get dynamic module that imports another module dynamically) ,- this is how we do it:
```javascript
…
   switch (selectedType) {
                case "horse":
                  import("./partials/horses.js").then(pm => pm.viewHorses(m));
                  break;
                case "snake":
                  import("./partials/snakes.js").then(pm => pm.viewSnakes(m));
                  break;
               …
   }
…
```
**src/with-modules/using-lazy-loading/partials/horses.js**
```javascript
export function viewHorses(horseModulePromise) {
    let sam = new horseModulePromise.Horse("Sammy the Pony", "animal-container", "Koko", "New York");
    sam.move(647);
    sam.move(315);
    sam.display();
    let fred = new horseModulePromise.Horse("Fred the Mule", "animal-container", "Moko", "Boston");
    fred.move(500);
    fred.display();
}
```
**src/with-modules/using-lazy-loading/partials/snakes.js**
```javascript
export function viewSnakes(snakeModulePromise) {
    let ben = new snakeModulePromise.Snake("Benny the Python", "animal-container", "Moko", "Boston");
    ben.move(12);
    ben.move(5.3);
    ben.display();
    let ron = new snakeModulePromise.Snake("Ron the Cobra", "animal-container", "Koko", "New York");
    ron.move(10.1);
    ron.display();
}
```

### Implementing the popular MVC design pattern with ES6 Modules
MVC is very popular design pattern. For those not familiar about, MVC it is the acronyms of Model View Controller,- and as the name implies it urge us to structure our application to three layers: the module layer that suppose to consist the data and its internals, the view layer that suppose to consist the display that the end user will see (only the presentation and not any logic connected to them,- and that includes also direct logic that connected to the presentation like event handlers), and the controller layer that suppose to act as a mediate between the data layer and the view layer and thus consist all the logic code (like nerve center that controls all). Here is how we implement this design using ES6 modules:

First we need html file and point our entry file to be our main controller:
```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Using ES6 Modules for developing in MVC pattern</title>
  </head>	
  <body>
    <div id="animal-container"></div>
    <script src="src/with-modules/using-mvc-pattern/controllers/main.js" type="module"></script>
  </body>
</html>
```

#### The Model Layer
**src/with-modules/using-mvc-pattern/models/owner.js**
```javascript
export class Owner {
    constructor(name, address) {
        this.name = name;
        this.address = address;
    }
}
```

**src/with-modules/using-mvc-pattern/models/animal.js**
```javascript
import { Owner } from "./owner.js";
export class Animal {
    constructor(theName, ownerName, ownerAddress) {
        this.name = theName;
        this.totalDistance = 0;
        this.owner = new Owner(ownerName, ownerAddress);
    }
    move(distanceInMeter = 0) {
        this.totalDistance += distanceInMeter;
    }
}
```

**src/with-modules/using-mvc-pattern/models/horse.js**
```javascript
import { Animal } from "./animal.js";
export class Horse extends Animal {
   constructor(...args) {
        super(...args);
        console.log("Galopping...!");
   }
}
```

**src/with-modules/using-mvc-pattern/models/snake.js**
```javascript
import { Animal } from "./animal.js";
export class Snake extends Animal {
   constructor(...args) {
        super(...args);
        console.log("Slithering...!");
   }
}
```

#### The View Layer
**src/with-modules/using-mvc-pattern/views/top-nav.js**
```javascript
let navElement = document.createElement("nav");
let navAnchorStyle = "text-decoration: none; background-color: #c6bcc6; padding: 5px 10px; border-radius: 4px; margin-right: 10px;";
navElement.innerHTML = "<a href='snake.html' style='" + navAnchorStyle + "'" + "data-animal-module='snake'>Snakes</a>";
navElement.innerHTML += "<a href='horse.html' style='" + navAnchorStyle + "'" + "data-animal-module='horse'>Horses</a>";

let body = document.querySelector("body");
navElement.style.cssText = "margin-bottom: 20px; display: inline-block;";
body.prepend(navElement);

document.getElementById("animal-container").innerText = "Click on the top menu buttons to choose animal type.";
```

**src/with-modules/using-mvc-pattern/views/helpers/animals.js**
```javascript
export function displayAnimal(animal, containerDiv) {
    let animalDiv = document.createElement("div");
    containerDiv.appendChild(animalDiv);
    animalDiv.innerText = "'" + animal.name + "' total moves distance was " + animal.totalDistance + "m.";
    animalDiv.innerText += "\nThe registreted owner is '" + animal.owner.name + "' and he lives in " + animal.owner.address + ".";
    animalDiv.innerHTML += "<hr>";
}
```

**src/with-modules/using-mvc-pattern/views/horses.js**
```javascript
import { Horse } from "../models/horse.js";
import { displayAnimal } from "../views/helpers/animals.js";

export function viewHorses() {
    let animalContainer = document.getElementById("animal-container");
    animalContainer.innerText = "";
    animalContainer.style.cssText = "border: 2px solid red; border-radius: 33px; padding: 45px; width: 450px;";

    let sam = new Horse("Sammy the Pony", "Koko", "New York");
    sam.move(647);
    sam.move(315);
    displayAnimal(sam, animalContainer);
    let fred = new Horse("Fred the Mule", "Moko", "Boston");
    fred.move(500);
    displayAnimal(fred, animalContainer);
}
```

**src/with-modules/using-mvc-pattern/views/snakes.js**
```javascript
import { Snake } from "../models/snake.js";
import { displayAnimal } from "../views/helpers/animals.js";

export function viewSnakes() {
    let animalContainer = document.getElementById("animal-container");
    animalContainer.innerText = "";
    animalContainer.style.cssText = "border: 2px solid red; border-radius: 33px; padding: 45px; width: 450px;";

    let ben = new Snake("Benny the Python", "Moko", "Boston");
    ben.move(12);
    ben.move(5.3);
    displayAnimal(ben, animalContainer);
    let ron = new Snake("Ron the Cobra", "Koko", "New York");
    ron.move(10.1);
    displayAnimal(ron, animalContainer);
}
```

#### The Controller Layer
**src/with-modules/using-mvc-pattern/controllers/main.js**
```javascript
import "../views/top-nav.js";
import { viewHorses } from "../views/horses.js";
import { viewSnakes } from "../views/snakes.js";

var selectedType = "";
for (const link of document.querySelectorAll("nav > a")) {
    link.addEventListener("click", e => {
        e.preventDefault();
        selectedType = link.dataset.animalModule;
        document.querySelectorAll("nav > a").forEach(el => el.style.fontWeight = "normal");
        e.target.style.fontWeight = "bold";
        switch (selectedType) {
            case "horse":
                viewHorses();
                break;
            case "snake":
                viewSnakes();
                break;
            default:
                document.getElementById("animal-container").innerText = "Not available animal type";
        }
    });
}
```

### Using Aggregation Module
We can create module file that intended for collecting scattering modules (i.e. importing them), and then echoing them back by exporting them. This kind of "echo" module is known by the name Aggregation Module. Here we'll continue with our example now using aggregate module.

First change the script tag in the html file to point to a new entry file that we are going to create just next:
```html
<script src="src/with-modules/using-aggregation-module/test.js" type="module"></script>
```
Now create this test.js file and put this content in it:
```javascript
import { Horse, Snake } from "./main.js";
let sam = new Horse("Sammy the Pony", "animal-container", "Koko", "New York");
sam.move(7);
sam.move(3);
sam.display();
let ben = new Snake("Benny the Python", "animal-container", "Moko", "Boston");
ben.move(1.4);
ben.move(0.1);
ben.display();
```
Pay notice that we are importing both Horse and Snake from the file main.js that is our aggregation module. Create this main.js and put this content in it:
```javascript
import { Horse } from "../horse.js";
import { Snake } from "../snake.js";
export { Horse, Snake };
```
That will work fine but there is shorthand syntax available for us that enable to export directly from file, so these two lines are equivalent to the three lines we used before:
```javascript
export { Horse } from "../horse.js";
export { Snake } from "../snake.js";
```

#### Aggregating to object
We can also aggregate modules to object and export this object. Change the content of main.js to be this content:
```javascript
import { Horse } from "../horse.js";
import { Snake } from "../snake.js";
let Animals = {};
Animals.Horse = Horse;
Animals.Snake = Snake;

export { Animals };
// You can of course do here also export default,- by that letting the end user the 
// ability to choose himself the object package name. Use this: 
// export default { Animals };
// OR this equivalent syntax:
// export { Animals as default };
```
Now the test.js file needs accordingly to be change to this:
```javascript
import { Animals } from "./main.js";
let sam = new Animals.Horse("Sammy the Pony", "animal-container", "Koko", "New York");
sam.move(77);
sam.move(533);
sam.display();
let ben = new Animals.Snake("Benny the Python", "animal-container", "Moko", "Boston");
ben.move(11.4);
ben.move(12.4);
ben.display();
```

### Using external libraries (third-party es6 module-based packages)
The emergence of modules in javascript, and not less important, the emergence of package loaders for javascript, like npm, yarn or bower,- all of those evolvements made it possible to make javascript packages that are structured projects consisted of multiple module-based files,- as opposed to the classic form library that was consisted one minified file that the end user downloaded to its file system via direct link (or download from CDN – aka Content Delivery Network). Included it after in html script tag, jQuery is a good example for this classic approach of one minified file. The emergence of modules in javascript made it possible for library developers to use multiple module-based files in their packages (that now can also be a big projects – for instance we now have quite a lot javascript module-based frameworks). Those kind of multiple file are "installed-based" – i.e. naturally we cannot instruct the end user to download the files one by one (it doesn't make sense practically) but we instruct the end users to install the all project with one command line (in your operating system Command Line Interface), that had been made possible with the installment of some __package loader__ like npm, yarn or bower. After this the end user can now use them directly with import commands (of course here we do not need to add any html script tag, as the end user project has its own entry file included with script tag and that’s more than enough).

One more advantage of module-based third party libraries that needed to be pointed out, is the ability to have dependency on other third party libraries, so we can end up in some cases with "dependency chain",- where the package we need is relied on other packages to be installed, and some of them relies on other packages, and so forth. In the classic library form, before the emerge of module packages, this scenario was harsh since the end user had to be then aware to all of the dependencies to include them in script tags in the right order (and that's really really really harsh!).  More than that when you use package loaders to install packages it informs you about the dependency it had on other packages, urging you to install them, or even optionally if you choose it, automatically install all the dependencies down through the dependency chain. More than that when you use package loader you have the ability to choose the version of the package and of course it identifies you also about the dependencies version demands of the dependent packages,- so really package loaders are great tools that are vital in our world of javascript module-based packages.

Another alternative to the use of package loaders is of course by using CDN (Content Delivery Network), here is example that make use of the lodash-es package (lodash version that uses es6 module format, as opposed to the regular lodash that uses the commonjs module format and thus suits more to node.js), loading this package from CDN:
```javascript
import _join from "https://unpkg.com/lodash-es@4.17.15/join.js";
const mainContentDiv = document.getElementById("main-content");
mainContentDiv.innerText = _join(["ES6", "Modules", "Rules!"], " ~~ ");
```
And for importing this same package being locally installed in node_modules folder (that is the folder name for third-party packages installed by the npm or yarn, that are package loaders for javascript/node.js packages):
```javascript
import _join from "/node_modules/lodash-es/join.js";
```
__Important:__ This line code will work in the regular ES6 loader, but will not work if we bundle it with webpack, nor in any other module bundler (module bundlers compiles javascript module files into one bundle file – I am going to dedicate topic for module bundlers later on this article), and that is because module bundlers are desktop compilers written with node.js that has the node.js relative path resolution that is different from the client-side javascript relative path resolution (that is identical to the html relative path resolution). I'm going to elaborate about relative path resolution just in the next topic.

Another thing worth to be mentioned here is the structure of the lodash-es package, that split each of it module methods to file of its own, by that allowing us to import one-by-one only the modules we need (here we import only the join.js file that as it names implies includes the join lodash method), lodash-es has also aggregation module to aggregate all the methods in single file called lodash.js, thus make it possible to import the all package strait away – like this:
```javascript
import * as _  from "/node_modules/lodash-es/lodash.js";
const mainContentDiv = document.getElementById("main-content");
mainContentDiv.innerText = _.join(["ES6", "Modules", "Rules!"], " ~~ ");
```
That of course will result with large amount of scripts imports payload, not to mention the large amount of code (we are loading here the entire package). In that case (e.g. importing from lodash.js), even if you bring to scope only the join method, like this:
```javascript
import _join from "/node_modules/lodash-es/lodash.js";
```
Now it is still loading the entire package, - so unless you intend to make heavy use with wide variety of the lodash-es methods, stick with importing from the dedicated files like we did earlier:
```javascript
import _join from "/node_modules/lodash-es/join.js";
```
Here it is worth mentioning that module bundlers offers feature called "tree shaking" (only relevant to static imports) to reduce from the resulted bundled the file modules that have not brought in to any scope (aka "stray modules"), so in this use case the previous mentioned line of code: ```import _join from "/node_modules/lodash-es/lodash.js";``` will not result with the payload of the entire package.

### ES6 module pattern is different from the CommonJS module pattern regarding to relative path resolution and other fundamental characteristics
ES6 module pattern and CommonJS module pattern intended from the first place to work in different execution environments. The ES6 module pattern is intended for use in client-side javascript been executed with web browsers, whereas the CommonJS module pattern is intended for use in server-side javascript been executed with Node.js – and that fact causes some character differences that grows from the different requirements being stems from the different environments. Maybe the most important difference is on relative path resolution and I'm just going to elaborate on this – but there are also other fundamental characteristics differences the stems from the different environments.

Node.js had no default object for the global scope, like the window object in client-side javascript language - this fact means:
* We cannot import the scope to our global scope because we haven't got one – so the syntax with the curly braces that imports scope is not suitable to Node.js  
* In the Node.js we need module pattern that imports dynamically (similar to the ES6 dynamic import) since we don't have here top-level global scope. 
* In the Node.js we need module pattern that aggregates the exports to function object, so that it can be grabbed within the file that imports as a returned value. *Explanation:* Node.js is a function based language (typically it uses callbacks) – I mentioned before the difference between static code, that is code that runs in the top level global scope, and the dynamic code, that is any other code kept inside block that is not intended to be run from the global top level scope – In node.js there is no static code only dynamic code exists! 

So the CommonJS as a module pattern has to be different to meet those Node.js requirements. Again other than those differences, the most important difference seems to be regarding to relative path resolution.

#### Relative path resolution that conforms both to the ES6 module pattern (client-side js in general or html – in short: any client-side) and the CommonJS module pattern
* ./ that denotes the same path of the script (or the html)
* ../ that denotes one level up (the parent path) of the script path (or the html path)
  * We can concatenate this denotes so that ../../ means two levels up, ../../../ means three levels up, and so forth.

#### Client-side (and it includes of course the case of the ES6 module pattern) relative path resolution 
* The back slash (/) denotes the web root. __Important:__ Client side, whether it is javascript or html, does not have access to folders out from the web root (i.e. located not under the web root folder) – the reason for this limitation connected to web security.

#### CommonJS module pattern relative path resolution
* The back slash (/) denotes the file system root.
* Paths that starts directly with folder name implies folder name (aka directory)
that needs to be located in node_modules folder - node_modules is the conventional name of the folder that the Node.js package loaders uses for package installments (whether npm or yarn) – and it can be under the web root, if the package had been installed locally, or, in the case of global install packages, in some other specific folder designated for those global package installments.

#### Module Bundlers realizes only the CommonJS module pattern relative path resolution
Module Bundlers are desktop tools written with Node.js and they intend to bundle module files into one single bundled file, and as such they realize only the CommonJS relative path resolution (by the way they are actually supports also the CommonJS module pattern as well as the ES6 module pattern), so if you intend to use module bundler for production, you need to change ahead before bundling all the javascript paths that appears on the ES6 module import statements to be conformed with the CommonJS relative path resolution (don't worry here about web security – the bundler removes all of those paths in the outputted bundled file). 

My suggestion over here is to use from the start paths that conforms to both module patterns, or as an alternative, to use the module bundler straight ahead in development stage – that is because though module bundlers aims for the purpose of creating bundled file for the production, their usefulness needs to be considered also in development as they are typically functioning also as task runners. Also worth to mention that in the past, and that was not so far from now, browsers compilers didn't have support for the capability to load scripts via the module es6 syntax (nor of course in other module syntax), so using module bundler in development stage was reasonable way to overcome this (as an alternative to module loader). I'll write more on using module bundlers in development stage later in this article when I'll come to discuss about module bundlers in general.

## Module Loaders – What are they and do we still need them?
Some history background: In the past – and that was not so far from now – browsers compilers didn't have support for the capability to load scripts via the module es6 syntax (nor of course in other module syntax). They had barely the support for the es6 modules in syntax terms but nothing more than that,- browsers didn't recognize the attribute type=module in the script tag, so no recognized way was to define the entry file,- more than that they didn't have the ability for bootstrapping the process of scanning module files tree up the road from the entry file, that must be done for determine which javascript files needs to be downloaded. So really at that time nothing was prepared, from the prospective of any browser, to support modules in practice. The bottom line of all this means that you could write modules in ES6 module format, but at that time the browsers didn't have the capability to make this actually work, the browsers at that time "refused" to load script file not through the html traditional script tag. 

That bring us to "Module Loaders" – those kind of javascript packages, as their name implies, brought us the support for module loading that the browsers until recent lack. The most popular module is SystemJS that also supports the ES6 module pattern. 

For loading our familiar module demo using the SystemJS you'll need first to include it with script tag:
```html
<script src="https://unpkg.com/systemjs@0.19.31/dist/system.js"></script>
```
Then bellow this, change the ```<script src="src/with-modules/main.js" type="module"></script>``` that worked for us great, just because we are working with some modern browser (all modern browsers now supports module loading), nonetheless now change it to load the entry file via SystemJS:
```html
<script>
    System.import('src/with-modules/main.js')
      .catch(console.error.bind(console));
</script>
```
Here we are coming to the question: Do we still need the SystemJS module loader, now that we have full support for module loading in all of the modern browsers?

The answer for this is that in general we don't need it anymore, unless:
* You need for some reason to support old browsers.
* You need support for dynamic import for IE browsers - "Dynamic import" is a stage 3 proposal and not implemented yet in IE at any version (even not the Edge).
* You need, again for some reason, support also for the module CommonJS pattern (the CommonJS is the pattern that prevail in Node.js). SystemJS support both the ES6 modules and the CommonJS module.

## Module Bundlers
The module paradigm has one big drawback aspect: performance - that is a major issue in any substantial project that involves more than few files. The prevailed custom in those projects was (and I'm not talking here necessarily about module files) that when it comes to move those kinds of projects to the production environment, i.e. uploading the project to the web server, we did those steps in advance (typically with the help of some task runner tool like Grunt or Gulp):
* Concatenating all the files to one big file.
* Minifying this file.
How we can concatenate files that use import statements that obviously specify the path for the file they intend to import?

Here Module Bundlers comes to help. Module Bundlers compiling javascript module files into one compiled file (typically also minifying it) that does not includes any imports/exports (all import and export statements had been removed from the outputted compiled file), - but still this single outputted compiled file must to faithfully reflect, by practical mean, its corresponded input module files (that’s the challenge must be met by Module Bundlers and obviously they succeed doing so). 

Module bundlers refuse to bundle modules that contains import from CDN (Content Delivery Network) or from any http url address whatsoever.

As I mentioned before, module bundlers realize only the CommonJS relative path resolution.

Module bundlers support also the CommonJS module pattern as well as the ES6 module pattern.

Examples of Module Bundlers are: Webpack, Parcel, Rollup, FuseBox

### Using module bundler solely for the purpose of creating bundled file to be deployed in the production environment (the web server)
As explained, module bundlers aims for the purpose of creating bundled file for the production, thus it make sense to consider module bundler uses to be only for that purpose, i.e. to be relevant only when it comes to deployment, but not to be involved at all during the development – I must say here that this is my personal preference of the way to use module bundlers (when I have the possibility to choose – some javascript development frameworks that are packed with some module bundler, for some reason, dictate us to use module bundlers also in development).

Although it is my personal preference it is not the prevailed preference main because of the historical fact that until recently the browsers lucks the support for loading modules so making use of module bundler (like webpack) gave us decent solution for this. I'll refer to the use of module bundlers and its other advantages in the next topic – but for those that prefer to not use module bundler until it comes to deployment – here are some vital things you need to do before bundling:
* Replace CDN (Content Delivery Network) imports with local installed equivalents.
* As mentioned earlier you need the paths in the imports to be conformed with the CommonJS relative path resolution – again my recommendation here is to use from the start paths that conforms to both module patterns.

### Using module bundler in development stage
Module bundlers typically offer the possibility to work with the bundled file during the development nonetheless to "feel" same as you are still working with your source modules in the sense that any console log (whether it is an error message or any other debug message log) will be pointed to the referred line in the originated source file, not the line in the bundle file (though that is the real file we are working with), and that is done with the help of [source maps](https://developer.mozilla.org/en-US/docs/Tools/Debugger/How_to/Use_a_source_map) that the bundler creates in the bundled file. More than this the module bundlers, again typically, offers (as an option if you choose to use it) a "watch" background task that tracking saved changes made in the source module files and in the event of any change made to them it will rebuild the bundled file.

Using module bundler in development stage has those advantages:
* As mentioned until recently the browsers lucks the support for loading modules so making use of module bundler (like webpack) gave us decent solution for this.
* You need, again for some reason, support also for the module CommonJS pattern.
  * Some find the CommonJS relative path resolution more appealing and concise – for instance, if we'll continue with our previous example of importing from lodash-es that had installed locally in the node_modules, - they prefer this (that conforms only to the CommonJS relative path resolution CommonJS relative path resolution): ```import _join from "lodash-es/join.js";``` over this: ```import _join from "./node_modules/lodash-es/join.js";```
* Module bundlers typically can function also as a task runner  – providing all/most of the task runner typical functionalities and by that saving as the need for a task runner (like grunt or gulp).

### Tree Shaking
As I mentioned before in this article, module bundlers offers feature called "tree shaking" to reduce from the resulted bundled the file modules that have not brought in to any scope (aka "stray modules"). Of course it is relevant only to imports that use the ES6 __static import__ syntax (as it is examining modules that have brought to the global scope and this information located inside the curly braces that only the static syntax have).  Historically Rollup.js was the first to offer tree shaking (and also coin the term "tree shaking"), thus exemplify the rest of module bundlers that now all seem to offer tree shaking.

### Lazy loading with module bundlers
I mentioned earlier in this article the most important advantage of dynamic import that is enables us doing 'lazy loading' of modules - i.e. loading modules at runtime based on aroused needs determined in runtime – also I brought in code to demonstrate this in action. At the concept level lazy loading collide with the concept of bundling not to mention that the bundling requires not only merging all the code to one file but also removing the modules (although as mentioned faithfully reflect the source code that had the modules).

However module bundlers do offer solution for lazy loading and that solution, if you have choose to use it, split aside all the modules imported dynamically to separated files (i.e. separated from the bundled file), terming it "chunk files", and enabling the 'lazy loading' of those chunks with ajax calls.
