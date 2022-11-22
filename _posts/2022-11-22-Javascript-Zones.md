---
title: Zones and change detection in Angular
---

## Intro

Because JS is single-threaded but asynchronous, you tend to kick off a lot of your functionality as tasks to be scheduled in future ticks. When you have a complicated app with lots of different areas kicking off tasks in ticks, there is sometimes a need to be able to store state associated with a given set of tasks that are all contributing to the same semantic outcome.

Javascript Zones are a means of storing that state; you create zones, or contexts, to store the state in.

## Zone.js

The library you want is `Zone.js`. It's available on cdn at `https://cdnjs.cloudflare.com/ajax/libs/zone.js/0.5.14/zone.min.js`, but it's included in Angular. Angular uses it to trigger change detection and update the HTML elements with modified values from Angular.

### Angular change detection

Zone provides a series of lifecycle hooks (provided as attributes of the zonespec when you create a new zone) that let you know when anything runs inside your zone of interest. Angular creates its own Zone called 'angular', and anything that updates values that need reflecting in the UI must run inside this zone. Angular hooks the lifecycle events to know when something has run, and calls change detection to do the updates.

This mostly happens automatically, because Angular kicks off most of the tasks itself and they're thus inside the correct zone. However, if you write your own async code that is invoked from somewhere outside of Angular then it won't have the right zone; you need to add some code that _is_ in the Angular zone to trigger the update:
```ts
export class AppComponent implements OnInit {
  constructor(private ngZone: NgZone) {}
  ngOnInit() {
    // New async API is not handled by Zone (ie. called by zone.run()),
    // so you need to use ngZone.run() to make the asynchronous 
    // operation callback in the Angular zone and
    // trigger change detection automatically.
    someNewAsyncAPI(() => {
      this.ngZone.run(() => {
        // update the data of the component
      });
    });
  }
}
```

Sometimes you don't want to trigger change detection, so you run your code outside of the Angular zone:

```ts
export class AppComponent implements OnInit {
  constructor(private ngZone: NgZone) {}
  ngOnInit() {
    // You know no data will be updated,
    // so you don't want to trigger change detection in this
    // specified operation. Instead, call ngZone.runOutsideAngular()
    this.ngZone.runOutsideAngular(() => {
      setTimeout(() => {
        // update component data
        // but don't trigger change detection.
      });
    });
  }
}
```

### Optimisation

By default, almost every event will trigger change detection. YOu can optimise your app by excluding some browser events, if you know that they won't cause any relevant changes.  Details [are here](https://angular.io/guide/zone).

## For testing - loading a script from console

```js
var _loadScript = function(path) {
    var script = document.createElement('script'); 
    script.type= 'text/javascript'; 
    script.src= path; 
    document.head.appendChild(script); 
};
_loadScript('underscorejs.org/underscore-min.js');
```

Something similar in node: 

```js
import('./node_modules/zone.js/dist/zone.min.js').then((module) => {this.zone = module;})
```