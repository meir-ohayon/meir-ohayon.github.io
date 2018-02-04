---
layout: post
title: Building Command Line Application with Node.js. Explains how to build package that act (also or just) as command line tool
date: 2018-01-21
categories: angular
---

We all know how to run node or js scripts in command line tool (cli) using the command node <i><b>path/to/script/script_file_name</i></b> , - but what if we want to give clients of our package the ability to run our scripts in <b>their</b> cli using the node command mentioned or alternatively (to save repeatedly typing of long commands) to alias those scripts in <b>their</b> package.json scripts section and then run the desired script with the command <b><i>npm run alias_name</b></i> ?

Of course we can instruct any client to run scripts in our package using the same commands, but he/she would had to specify the all path to our package, say that the script located in folder called "bin" in our package root folder, then the command should be something like this: <i><b>node ./node_modules/package_folder_name/bin/script_file_name</b></i> , not only that, usually when dealing with cli packages the desire is to add those command line scripts globally – i.e. you guiding clients to install the package <u>globally</u> so that they will be available to any future projects (few examples for that, from many lot others, are the grunt-cli, gulp and angular-cli packages),- for those, and <b>here we are discussing only in this common scenario</b>, the full path is OS dependent, so we need to instruct the client to get the path the global installation in she's OS themselves and from that constructing the full path to our scripts, and that is cumbersome! In this case our package is available globally because it was installed globally, how can we do those scripts available globally too, in other words: how can I add those script files into the clients PATH so that she can run them anywhere without specifying the full path?

Here are two points of definitions that important for the rest of this article:
*	Scripts that not intended to be only required (called) by our package, but intended to be executed by the package clients, are called "binaries scripts" and are commonly located in folder named bin. Do not confuse those binary script files with binary (exe) files.
*	Each binary script file must start with the line:<br>#! /usr/bin/env node<br>This line is needed in all OS, including non-linux OS like Windows or Mac,- if you omit this line the solution I'll give just bellow will not enable you to run the script by name directly without prefixing the command name with node (e.g. this way you can run the script from any directory simply by calling it by name and nothing more).

## The package.json bin section
To solve the problem described here, the package.json bin section had created. In the bin section we map command name to binary script file. <b>On global install of the package</b>, npm will add all the scripts mentioned in the bin section to the PATH, thus enabling you run them from any directory, and also enable running them simply by calling their names and nothing more!

Let's demonstrate that with simple example:

Create folder called cli-package, in this folder create package.json file and bin folder. In the package.json file paste this:

```javascript
{
  "name": "cli-package",
  "version": "1.0.0",
  "description": "Demonstrate command line tool package",
  "bin": {
	"mycmd": "./bin/my-cmd.js"
  }  
}
```

Now in the bin folder create my-cmd.js and paste this:
```javascript
#! /usr/bin/env node
console.log("I am my-cmd.js");
```

Again pay notice to the first line: #! /usr/bin/env node – each binary script file must start with this as explained earlier.

Now we'll take advantage of the npm install <folder> command that installs the folder and at the same time symlinks it. In global mode it also adds the binaries specified in the package.json bin section to the PATH and enable running them simply by calling their name and nothing more.

Open the command line in the parent folder of the cli-package folder, and type:
    
    npm install ./cli-package –-global

Now you can test this by opening command line in any directory (no matter where), and type: 
    
    mycmd

You should see in response:
    
    I am my-cmd.js

The install command we did earlier also symlink the global install cli-package folder to our local source cli-package folder,- to see the effect of this open the my-cmd.js file and change the message in the console.log and save it. Now run again mycmd and you should see the new message (no need for reinstall the package). However any addition to the bin section, i.e. adding new binary file will force us to run again npm install ./cli-package –-global (otherwise the new ones would not be recognized).

## Bonus: Adding Parameters Support
To accept parameters from the user you need to use <i>process.argv</i> : 

Open the my-cmd.js file we created earlier and change its content to this:

```javascript
#! /usr/bin/env node
console.log("I am my-cmd.js");
console.log("My arguments are:\n" + process.argv);
console.log("My parameters, sliced from the arguments, are:\n" + process.argv.slice(2));
console.log("My second parameter is:\n" + process.argv.slice(2)[1]);
console.log("Also my second parameter is (this time without slicing):\n" + process.argv[3]);
```

Save this and run

    mycmd param-a param-b
    
You should see response similar to this:

```
I am my-cmd.js
My arguments are:
C:\Program Files\nodejs\node.exe,C:\Users\meir\AppData\Roaming\npm\node_modules\cli-package\bin\my-cmd.js,param-a,param-b
My parameters, sliced from the arguments, are:
param-a,param-b
My second parameter is:
param-b
Also my second parameter is (this time without slicing):
param-b
```

Notice that process.argv returns array with the first two elements are the path for the node executable and the path to the binary script,- and only from the third element and on it starts listing our parameters.

Also worth to mention that the array structure is not always the convenient structure for dealing with parameters,- sometimes we want the parameters to be in the form of key=value ,i.e. something like: mycmd a=param-a b=param-b ,for those kind of parameters a key-value object is more suitable and convenient. For this scenario kind, and/or to support flags in convenient way consider using the [minimist](https://www.npmjs.com/package/minimist) package.
