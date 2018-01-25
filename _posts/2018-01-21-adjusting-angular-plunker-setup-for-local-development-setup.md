---
layout: post
title: Adjusting angular plunker setup for local development setup
date: 2018-01-21
categories: angular
---

Download the plunker angular setup by visiting https://plnkr.co/ and clicking the 'Launch the Editor' button, then in the top menu choose New->Angular , after it will finish loading click the download button in the top right corner to download this setup as a zip file, create folder for your project and extract it there , and for converting it to local development setup follow those steps:

## In index.html
1.	Change from &lt;base href=\".\" /&gt; to &lt;base href=\"/\" /&gt;
2.	Add bellow this: &lt;meta charset=\"UTF-8\"&gt;&lt;meta name=\"viewport\" content=\"width=device-width, initial-scale=1\"&gt;
3.	Change from &lt;script src=\"config.js\"&gt;&lt;/script&gt; to &lt;script src=\"systemjs.config.js\"&gt;&lt;/script&gt;<br>Change also the file name !
4.	Change from: System.import(\'app\')<br>to: System.import(\'src/main.js\')

## In systemjs.config.js (you have changed to this name from config.js)
1.	Delete (or comment) the first lines in the System.config method:<br>transpiler: \'typescript\',<br>typescriptOptions: {<br>   emitDecoratorMetadata: true<br>},
2.	If you don\'t intend to use angular animations then delete (or comment) those lines:<br>\'@angular/animations\': \'npm:@angular/animations\' + angularVersion + \'/bundles/animations.umd.js\',<br>\'@angular/platform-browser/animations\': \'npm:@angular/platform-browser\' + angularVersion + \'/bundles/platform-browser-animations.umd.js\',<br>    \'@angular/animations/browser\': \'npm:@angular/animations\' + angularVersion + \'/bundles/animations-browser.umd.js\',
3.	If you don\'t intend to use angular testing packages then delete (or comment) those lines:<br>\'@angular/core/testing\': \'npm:@angular/core\' + angularVersion + \'/bundles/core-testing.umd.js\',<br>\'@angular/common/testing\': \'npm:@angular/common\' + angularVersion + \'/bundles/common-testing.umd.js\',<br>\'@angular/common/http/testing\': \'npm:@angular/common\' + angularVersion + \'/bundles/common-http-testing.umd.js\',<br>\'@angular/compiler/testing\': \'npm:@angular/compiler\' + angularVersion + \'/bundles/compiler-testing.umd.js\',<br>\'@angular/platform-browser/testing\': \'npm:@angular/platform-browser\' + angularVersion + \'/bundles/platform-browser-testing.umd.js\',<br>\'@angular/platform-browser-dynamic/testing\': \'npm:@angular/platform-browser-dynamic\' + angularVersion + \'/bundles/platform-browser-dynamic-testing.umd.js\',<br>    \'@angular/http/testing\': \'npm:@angular/http\' + angularVersion + \'/bundles/http-testing.umd.js\',<br>\'@angular/router/testing\': \'npm:@angular/router\' + angularVersion + \'/bundles/router-testing.umd.js\',
4.	Change from:<br>app: {<br> &nbsp;&nbsp;     main: \'./main.ts\',<br>  &nbsp;&nbsp;    defaultExtension: \'ts\'<br>},<br>to:<br>app: {<br>  &nbsp;&nbsp;    defaultExtension: \'js\',<br>  &nbsp;&nbsp;   meta: {<br>  &nbsp;&nbsp;&nbsp;&nbsp;      \'./*.js\': {<br>   &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;       loader: \'systemjs-angular-loader.js\'<br>  &nbsp;&nbsp;&nbsp;&nbsp;      }<br>  &nbsp;&nbsp;    }<br>},<br>Create the file systemjs-angular-loader.js and copy its content from here:<br><https://github.com/angular/quickstart/blob/master/src/systemjs-angular-loader.js>

## Under src folder
Create the file tsconfig.json and copy its content from here:<br>
<https://github.com/angular/quickstart/blob/master/src/tsconfig.json>

## Back in the root folder
1.	Create new file: server.js and put in it this content:<br>process.env.NODE_ENV = \'production\';<br>var history = require(\'connect-history-api-fallback\');<br>var express = require(\'express\');<br>var app = express();<br>app.use(history({ htmlAcceptHeaders: [\'text/html\', \'application/xhtml+xml\'] }));<br>app.use(express.static(__dirname)); <br>app.listen(3000);<br>console.log(\'App listening on port 3000\');
2.	Create new file: package.json and put in it this content:<br>{<br>  &nbsp;&nbsp;  \"name\": \"Your Name\",<br>  &nbsp;&nbsp;  \"version\": \"1.0.0\",<br>  &nbsp;&nbsp;  \"description\": \"…\",<br>  &nbsp;&nbsp;  \"scripts\": {<br>   &nbsp;&nbsp;&nbsp;&nbsp;    \"build\": \"tsc -p src/\",<br>   &nbsp;&nbsp;&nbsp;&nbsp;    \"build:watch\": \"tsc -p src/ -w\",<br>   &nbsp;&nbsp;&nbsp;&nbsp;    \"start\": \"concurrently \\\"node ./server\\\" \\\"node ./node_modules/browser-sync/dist/bin start \-\-proxy localhost:3000 \-\-files src/\*\*/\*\.\*\\\" \\\"npm run build:watch\\\"\"<br> &nbsp;&nbsp;   },<br>  &nbsp;&nbsp;  \"author\": \"…\",<br>  &nbsp;&nbsp;  \"license\": \"…\",<br>  &nbsp;&nbsp;  \"dependencies\": {<br>     &nbsp;&nbsp;&nbsp;&nbsp;    \"connect-history-api-fallback\": \"^1.5.0\",<br>   &nbsp;&nbsp;&nbsp;&nbsp;      \"express\": \"4.15.2\"<br>  &nbsp;&nbsp;  },<br>  &nbsp;&nbsp;  \"devDependencies\": {<br>   &nbsp;&nbsp;&nbsp;&nbsp;     \"browser-sync\": \"^2.23.5\",<br>   &nbsp;&nbsp;&nbsp;&nbsp;     \"concurrently\": \"^3.2.0\"<br>  &nbsp;&nbsp;  }<br>}<br>Install the dependencies by running npm install and after that run the application by running npm start – if everything is OK your browser will be launched automatically with your starting project page.<br>

## Notes:
1.	The plunker setup uses CDN from <https://unpkg.com/> - of course you can change this to be installed locally  in your computer by adding them as dependencies (some of them needs to be in the dependencies section and some in the devDependencies section) in the package.json file and installing them with npm install, then you also need to change line in the systemjs.config.js file, from: paths: { \'npm:\': \'https://unpkg.com/\' }, to: paths: { \'npm:\': \' node_modules/\' },
2.	Since the CDN packages defined in the systemjs.config.js file are not available in compilation time, you\'ll get compile errors (in the command line output after running npm start and in typescript supported editors \'problems\' tab like existed in VS Code, <u>you will not get any error in the browser console</u>) like this error message: Cannot find module \'@angular/…\' – ignore them they are harmless!
