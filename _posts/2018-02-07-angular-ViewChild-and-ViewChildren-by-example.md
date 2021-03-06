---
layout: post
title: Angular ViewChild and ViewChildren by example
date: 2018-02-07
categories: angular
---

In this post I'll explain the use of Angular ViewChild and ViewChildren all accompanied with code to demonstrate the usage of them. Plunker link to the full code we'll use are listed in the bottom of this post.

ViewChild and ViewChildren enable us to communicate with child components and thus enabling to encapsulate some business logic along with its related view (template) to component of its own, so that it can be reused more than once,- not as singleton (one shared occurrence) like service, that also is intended for business logic only and doesn't have template,- but here we use component, meaning that each use has its own occurrence and also accompanied with template as it is always in components (that the main difference between components and services). <b>All of that in the reality that any component that will use it will need to communicate with it</b> - that is what ViewChild and ViewChildren come to solve!

That open for us the way to modular our project without any limitations – <b>any</b> part that has the potential to be used more than once (in one component or across different components)   can, and should be, moved to component if its own,- we can take it even further than that and organize those general-purpose components under some folder called 'shared' or 'common' or whatever,- we can take it even further and use [share module](https://angular.io/guide/sharing-ngmodules),- but here we'll focus in demonstrating the use case of ViewChild and ViewChildren solely and not in other angular modularity aspects.

## ViewChild and ViewChildren
To demonstrate the usage of ViewChild and ViewChildren we'll do a demo using the ng-bootstrap datepicker & timepicker.
Install the [ng-bootstrap](https://ng-bootstrap.github.io/#/getting-started) by running <code>npm install --save @ng-bootstrap/ng-bootstrap</code>

> If you are using SystemJS, you should also adjust your configuration to point to the UMD bundle. 
> In your systemjs config file, map needs to tell the System loader where to look for ng-bootstrap: 
>
> map: {
>
>   '@ng-bootstrap/ng-bootstrap': 'node_modules/@ng-bootstrap/ng-bootstrap/bundles/ng-bootstrap.js',
>
> }

Add the bootstrap css and the font awesome css in the index.html :
```
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/4.7.0/css/font-awesome.min.css">

<link rel="stylesheet" href="https://unpkg.com/bootstrap@4.0.0-beta.2/dist/css/bootstrap.min.css">
```

Edit the app module (your root module) to look like that:
```javascript
import { NgModule }      from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';

import { FormsModule } from '@angular/forms';
import { NgbDatepickerModule, NgbTimepickerModule } from '@ng-bootstrap/ng-bootstrap';

import { AppComponent }  from './app.component';
import { NgbdDatepickerPopup } from './datepickers/datepicker-popup';
import { NgbdTimepicker } from './timepickers/timepicker.component';

@NgModule({
  imports: [ 
    NgbDatepickerModule.forRoot(), 
    NgbTimepickerModule.forRoot(), 
    BrowserModule, 
    FormsModule 
  ],
  declarations: [ AppComponent, NgbdDatepickerPopup, NgbdTimepicker ],
  bootstrap:    [ AppComponent ]
})
export class AppModule { }
```
> Pay notice to the line: <code>import { NgbDatepickerModule, NgbTimepickerModule } from '@ng-bootstrap/ng-bootstrap';</code>
>
> In the ng-bootstrap documentation they guide us to import the main module like this: <code>import {NgbModule} from '@ng-bootstrap/ng-bootstrap';</code>
>
> But here we are interesting only on the datepicker and timepicker so we imported only those modules specifically.

> The next lines to pay notice about are:
>
> import { NgbdDatepickerPopup } from './datepickers/datepicker-popup';
>
> import { NgbdTimepicker } from './timepickers/timepicker.component';
>
> Those import statements are for components we are going to create very soon. The purpose for those components is to encapsulate the ng-bootstrap datepicker & timepicker into components with configuration settings so that we can reuse those pre-configured components without the need to define the settings again in each call to datepicker/timepicker.

## Creating the timepicker component
Create folder called timepickers and in it create new file called timepicker.component.ts and paste this in it:
```javascript
import {Component, Input} from '@angular/core';

@Component({
  selector: 'ngbd-timepicker',
  templateUrl: './timepicker.html'
})
export class NgbdTimepicker {
  @Input() spinners: Boolean = false;
  model: Object;
}
```
Now create new file timepicker.html in the same folder and paste this in it:
```
<ngb-timepicker [(ngModel)]="model" [seconds]="true" [spinners]="spinners"></ngb-timepicker>
```
For details on the available timepicker options see [here](https://ng-bootstrap.github.io/#/components/timepicker/examples) in the ng-bootstrap documentation.

## Creating the datepicker component
Create folder called datepickers and in it create new file called datepicker-popup.ts and paste this in it:
```javascript
import { Component } from '@angular/core';

@Component({
  selector: 'ngbd-datepicker-popup',
  templateUrl: './datepicker-popup.html'
})
export class NgbdDatepickerPopup {
    model: Object;
    date: Date = new Date();
    minimumDate = { year: this.date.getFullYear() - 3, month: 1, day: 1 };
    maximumDate = { year: this.date.getFullYear() + 3, month: 12, day: 31 };
}
```
Now create new file datepicker-popup.html in the same folder and paste this in it:
```
<div class="input-group">
  <input class="form-control" placeholder="yyyy-mm-dd" readonly [minDate]="minimumDate" [maxDate]="maximumDate" [outsideDays]="'collapsed'"
    name="dp" [(ngModel)]="model" ngbDatepicker #d="ngbDatepicker">
  <button class="input-group-addon" (click)="d.toggle()" type="button">
       <span class="fa fa-calendar"></span>
  </button>
</div>
```
For details on the available datepicker options see [here](https://ng-bootstrap.github.io/#/components/datepicker/examples) in the ng-bootstrap documentation.

## Let's use datepicker and timepicker in some component
Edit the root component to look like that:
```javascript
import { Component } from '@angular/core';
@Component({
  selector: 'my-app',
  templateUrl: './app.component.html'
})
export class AppComponent { }
```
And add new file app.component.html and paste this in it:
```
<div class="container-fluid">
    <h4>Datepicker in a popup + Timepicker</h4>
    <div class="row">
        <div class="col-xl-2 col-sm-4">
            <ngbd-datepicker-popup></ngbd-datepicker-popup>    
        </div>
        <div class="col-xl-10 col-sm-8">
            <ngbd-timepicker></ngbd-timepicker>
        </div>
    </div>
</div>
```
After saving all of the code and running it you'll be able to see page with datepicker and timepicker in each you can enter values of time/date.

It's all seems to be OK but there is one big problem – say I click on the datepicker and choose date on it, the date I chose saved in the model object of the datepicker component I made (the same is with the timepicker the time is saved in the model object of the timepicker component I made),- how can I retrieve this value here in the app component, in general speaking: how can I get value of child component in the parent component?

Let's change the app.component.html to include also div section that displays the date that had been chosen (for now we don't know how to fill the values):
```html
<div class="container-fluid">
    <h4>Datepicker in a popup + Timepicker</h4>
    <div class="row">
        …
    </div>
   <div class="row">
        <div class="col-xl-2 col-sm-4">
            {% raw %}
            <pre>Selected date: {{ WHAT DO I NEED HERE? }}</pre>
            {% endraw %}
        </div>
        <div class="col-xl-10 col-sm-8">
            {% raw %}
            <pre>Selected time: {{ WHAT DO I NEED HERE? }}</pre>
            {% endraw %}
        </div>
    </div>
</div>
```
So how can we get access to the selected date/time values?

## Introducing ViewChild to get access to child component
With the use of ViewChild we can reference child component to variable.

Change the app component to look like this:
```javascript
import { Component, ViewChild } from '@angular/core';
import { NgbdDatepickerPopup } from './datepickers/datepicker-popup';
import { NgbdTimepicker } from './timepickers/timepicker.component';
@Component({ … })
export class AppComponent {
  @ViewChild(NgbdDatepickerPopup) private datePicker: NgbdDatepickerPopup;
  @ViewChild(NgbdTimepicker) private timePicker: NgbdTimepicker;
}
```
Now we can use this in the template file like this:
```html
{% raw %}
<pre>Selected date: {{ datePicker?.model | json }}</pre>
<pre>Selected time: {{ timePicker?.model | json }}</pre>
{% endraw %}
```

This is fine solution to get values of single occurrence of child component – but what if we have multiple occurrences of the same child component?

To demonstrate this scenario let's add one more timepicker to our template file (this time, just for fun, I did it with spinners):

<code><ngbd-timepicker [spinners]="true"></ngbd-timepicker></code>

How do I get now the separate values of the two occurrences of the timepicker child component? Can I get array reference of child component occurrences?

## Before introducing ViewChildren to get array reference of child component occurrences
We will add second timepicker to our template and we want both timepickers to display their selected values, so our template file, app.component.html, needs to look like this (for now we don't know how to fill the values for the timepickers):
```html
<div class="container-fluid">
    <h4>Datepicker in a popup + Timepicker</h4>
    <div class="row">
        <div class="col-xl-2 col-sm-4">
            <ngbd-datepicker-popup></ngbd-datepicker-popup>    
        </div>
        <div class="col-xl-10 col-sm-8">
            <ngbd-timepicker></ngbd-timepicker>
        </div>
    </div>
    <div class="row">
        <div class="col-xl-2 col-sm-4">
            {% raw %}
            <pre>Selected date: {{ datePicker?.model | json }}</pre>
            {% endraw %}
        </div>
        <div class="col-xl-10 col-sm-8">
            {% raw %}
            <pre>Selected time: {{ WHAT DO I NEED HERE? }}</pre>
            {% endraw %}
        </div>
    </div>
</div>

<hr>
<h4>Timepicker with spinners</h4>
<ngbd-timepicker [spinners]="true"></ngbd-timepicker>
<div>
    {% raw %}
    <pre>Selected time: {{ WHAT DO I NEED HERE? }}</pre>
    {% endraw %}
</div>
```
So again how can we get access to the selected time values?

## Now introducing ViewChildren to get array reference of child component occurrences
With the use of ViewChildren we can reference child component occurrences to array variable,- to QueryList object if to be precise,- but it can be easily to array using its toArray method.

Open the app component file and change the first import statement to include also the ViewChildren and the QueryList:
```javascript
import { Component, ViewChild, ViewChildren, QueryList } from '@angular/core';
```
In the app component class initialize two variables to for each timepicker:
```
time1: Object;
time2: Object;
```  
Use the ViewChildren to get QueryList of timepickers and try to assign them to the two variables in the constructor:
```javascript
@ViewChildren(NgbdTimepicker) timePickers: QueryList<NgbdTimepicker>;
constructor() {
    this.time1 = this.timePickers.toArray()[0];
    this.time2 = this.timePickers.toArray()[1];
}
```
And in the template file we'll use it:
```html
{% raw %}
<pre>Selected time: {{ time1?.model | json }}</pre>
<pre>Selected time: {{ time2?.model | json }}</pre>
{% endraw %}
```
If you'll do all of the above and run it, the application will not run and you'll get error message in the browser console: this.timePickers is udefined – that error is because the ViewChildren (and ViewChild) instances are not available in the constructor life cycle phase and therefore you need to postpone those assigns to later phase in the life cycle: AfterViewChecked

So change the first import to include the AfterViewChecked and the class declaration to implement this phase and finally move those assigns to the ngAfterViewChecked method:
```javascript
import { Component, ViewChild, ViewChildren, QueryList, AfterViewChecked } from '@angular/core';
…
export class AppComponent implements AfterViewChecked {
  time1: Object;
  time2: Object; 
  @ViewChild(NgbdDatepickerPopup) private datePicker: NgbdDatepickerPopup;
  @ViewChildren(NgbdTimepicker) timePickers: QueryList<NgbdTimepicker>;

  ngAfterViewChecked() {
    this.time1 = this.timePickers.toArray()[0];
    this.time2 = this.timePickers.toArray()[1];
  }
}
```
Now everything seems to work fine (test it!), – but if you open the browser console you'll see error message of: Expression has changed after it was checked. Previous value: 'null'. Current value: 'undefined'.

Although the exception appear in development mode only at the moment the value is checked and it is harmless (the application works), it is obviously desired to get rid of it.

For details about this exception see this stack overflow: https://stackoverflow.com/questions/39787038/how-to-manage-angular2-expression-has-changed-after-it-was-checked-exception-w ,and following the recommendation there you'll need to use the ChangeDetectorRef – so add it to the import statement:
```javascript
import { Component, ViewChild, ViewChildren, QueryList, AfterViewChecked, ChangeDetectorRef } from '@angular/core';
```
And inject it in the constructor:
```javascript
constructor(private cdRef:ChangeDetectorRef) {}
```
Finally use it in the ngAfterViewChecked method:
```javascript
ngAfterViewChecked() {
    this.time1 = this.timePickers.toArray()[0];
    this.time2 = this.timePickers.toArray()[1];
    this.cdRef.detectChanges();
  }
```
Now everything will work fine without any exceptions.

Plunker link to the full code available [here](https://plnkr.co/edit/9pP6V1dPXUVIcEYqs29V?p=preview).

## Drawback of our usage of ViewChildren: the use of array by index
The only drawback of our usage of ViewChildren is that is uses array by index – i.e: this.timePickers.toArray()[0] – this implies drawback because if I will change in the template file the position of the first timepicker to be the second (switching the order), then its index will also changed accordingly and I'll need to return to the component code and change it from this.timePickers.toArray()[0] to this.timePickers.toArray()[1] – this may be forgotten and any way it would be useful not to rely upon index,- but to get array element by name rather than by index.

Let's try to achieve this goal.

## Bonus: Assign to array by name with ViewChildren
To start with there's a thing I need reveal here: the first timepicker is for work time and the second timepicker is for fun time,- well, I know that I should tell you this earlier, but better late than never.

So let's start by changing the variable names in our root component from time1 and time2 to meaning names:
```javascript
workTime: Object;
funTime: Object;
ngAfterViewChecked() {
   this.workTime = …
   this.funTime = …
   …
}
```
Also in our root template file, app.component.html:
```html
{% raw %}
<pre>Selected time: {{ workTime?.model | json }}</pre>
<pre>Selected time: {{ funTime?.model | json }}</pre>
{% endraw %}
```
Next in the timepicker.component.ts file add @Input called name (with type string) like this:
```javascript
export class NgbdTimepicker {
  @Input() name: string = '';
  …
 }
```
Next return again to app.component.html to define those inputs in the timepickers components:
```html
<ngbd-timepicker [name]="'work'"></ngbd-timepicker>
<ngbd-timepicker [spinners]="true" [name]="'fun'"></ngbd-timepicker>
```
<b><u>Important:</u></b> Pay notice to the single quotation marks between the double quotation marks, i.e. "<b>'</b>...<b>'</b>" – without them it will not work, since it will not recognize as string, as it intended to be, but as a variable name and this is not our intention here. 

Finally return again to the root component and change the ngAfterViewChecked method to look like this:
```javascript
ngAfterViewChecked() {
    let workIndex = this.timePickers.toArray().map(function(e) { return e.name; }).indexOf('work');
    let funIndex = this.timePickers.toArray().map(function(e) { return e.name; }).indexOf('fun');
    this.workTime = this.timePickers.toArray()[workIndex];
    this.funTime = this.timePickers.toArray()[funIndex];
    this.cdRef.detectChanges();
}
```
And that’s all. Plunker link to the full code of this bonus section is available [here](https://plnkr.co/edit/g1RODUcTXKmpsC5RGMsn?p=preview) (the full plunker code we did earlier before the bonus section is available [here](https://plnkr.co/edit/9pP6V1dPXUVIcEYqs29V?p=preview)).

Final words: happy modular coding!
