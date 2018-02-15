---
layout: post
title: Web server as dynamic html provider Vs web server as api (json/jsonp) provider
date: 2018-02-15
categories: full-stack
---

In this post we'll discuss about two prevailed methodologies for creating dynamic pages that interact with web server.

Here we are concentrating only in dynamic web pages, i.e. html pages that are not really existed as real html files (aka static files), but created dynamically on-demand by the web server program in one methodology or by client-side [SPA](https://en.wikipedia.org/wiki/Single-page_application) program in the other methodology that typically also employs server (if at all) as messages provider (typically in json/jsonp format).

## Web server as dynamic html provider
The traditional role of web server in web applications was to create dynamic web pages base on some logic defined in some business logic coded in the server-side program language. In this methodology we have typically one or some few real (static) html file, intended for initial load and typically called index.html or index.php (html files does not have to be with 'html' extension, and the can also include not only client-side script but also server-side script!), this html file is the one that loads in first request (the home default page), sometimes it redirects to other static file that may needed for display in initial load (aka 'login.html' or 'login.php' and this login file may include a link to other static 'register' file),- the other pages that loads after this initial load are typically not static but dynamically created pages [that not always the case,– some page(s) does not need to include dynamic server-side content, but only static (unchanged) content, and thus are better to be served as real static page(s), – 'terms of use/license' page or pdf manual assets can be good examples for that].

That is the traditional (old but not obsolete) way for web programming, and it is worth mentioning that in this approach the web server is the one who responsible for creating the dynamic html pages in response to client requests base on some business logic located in its side (sometime resorting also for client inputs to be sent along with the client request, using http form or ajax request).  

That traditional approach also entails:
*	Server-side routing (if routing needed) – since we are dealing here with server-side dynamic pages.
*	In some cases the use of server-side view template software (aka view templating) also might by handy [like twig or smarty in php and pug (formally jade) or ejs in node.js ,– surely it existed also in all other server-side languages].
*	In some cases needs for code structured modular design, like the MVC popular design pattern, to help arrange server side code (thus the emerge of the server-side mvc frameworks genre, that typically also includes server-side routing and server-side view templating).

## Web server as api (json/jsonp) provider
Nowadays more and more web developers are drifting toward the SPA & web server as API methodology (it seems that is not the case, for some reason, in web projects that needs to have backend admin interface, and that includes CMS, EBook, Ecommerce and all others that employs backend admin). 

SPA stands for Single Page Application and when we use this term we intend to client-side software that employs <b>dynamic web pages on the client side</b>, i.e. in this methodology we have typically one static html file, intended for the first load and that file typically called index.html, and it includes script for SPA javascript framework that gives as the tools for creating dynamic web pages (in this methodology they are called 'views' and it has built-in templating support), of course along with the app javascript that utilizes those tools base on the app business logic. 

That also means that all of the application code is loaded in the first load (we do not request pages further on),- pages (now also called views) are switched base on client-side interaction logic defined in the app javascript (and not by the client->server model in the form of client-request->server-response). 

That also means that the web server does not responsible for creating dynamic web pages nor for the routing,- all of this is done in the client side by utilizing the SPA,- the server is typically only sends (on first client request) the first page needed to be loaded (the index.html), and after that, if needed, it responses to ajax/form calls as an api provider (typically by sending response in json/jsonp format).

The pros of this methodology are:
*	Better user experience – the application performance is match better than it is in the traditional other methodology,- that not includes the first load that is match slower since the all application is loaded in that first load,- but after the first load everything works match faster and smoothly without any flickering in page (view) changing,- flickering that are characteristics of the traditional other methodology, since there any page change entitles sending request to the server and wait for it to response, and here that is not the case.
*	Reducing burden from the server:
    *	Reduces the client requests to one initial request only.
    *	All the business logic is executed in the client side.
    *	The server can act mainly as an api and thus reduce payload to messages (json) only that can be sent asynchronically and also by not resorting to the http protocol response time can be match faster.
