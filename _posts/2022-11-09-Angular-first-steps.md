---
title: First steps with Angular
---

## Overall process for Angular apps

1. Set up the project: package.json, download and install all the Angular stuff
  * Typescript
  * Angular framwork
  * Number of other libs: RxJs (reactive extension lib), zone.js (execution context), tslib (TS helper utils)
  * Configure webpack for the build process
  * Set up the dev web server, plug into the production deployment 
2. Startup code - write components, etc. A component is some HTML and some JS.
3. Bundle, build, run, test.


There is a set of scripts to automate the above, obv:
1. `npm install @angular/cli` - gives you a utility, `ng`.
2. `ng new myapp` - interactive config builder, deps installer, ...
3. `ng serve --open` - `open` opens the browser to the right URL.


*NOTE:* A prefix on an npm package like "@angular/" is a namespace. The namespace has an organisational owner, so you can be sure that anything in that namespace is "official". Also helps to eliminate duplication of package names.

## Angular apps

Angular apps are made up of modules, and modules are made up of components.  A component is an HTML tag, and comprises some TS, an HTML template and (optionally) some CSS for styling.

One of the modules is the root module of the app, and it declares a root component. The root component must be referenced in index.html, or the Angular startup will fail; it expects to find that component in the index file.

The root module is specified in `<project>/src/main.ts`. The main TS and the index HTML are specified in the project's angular config, at `<project>/angular.json`, in `projects -> <app_name> -> architect -> build -> options`.

```ts
// AppModule is the root module.
platformBrowserDynamic().bootstrapModule(AppModule)
  .catch(err => console.error(err));
``` 

In the declaration of that module, it will declare its contained components, and its root components. In the default app, the moduel is declared in `<project>/src/app/app.module.ts`:

```ts
@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule
  ],
  providers: [],
  // Declaration of root components
  bootstrap: [AppComponent]
})
export class AppModule { }

```

## Declaring and rendering components 

The root component in the default app is at at `<project>/src/app/app.component.*`: html, ts and css.

The code is as follows:

```ts

// Decorator to turn the class into a component
@Component({
	// Html tag
  selector: 'app-root',
  // HTML code
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent {
  title = 'myapp';
}

```

TS classes that are components are called "component classes" or "template classes". There are industry conventions around them:
* Put the files for a component in a directory of their own in `app` named after the component.
* Each file is named `<component-name>.component.<ext>`
* The template HTML goes into a separate file if it's complicated, referred to with `templateURL`. If it's simple it can go into `template` inline.

Each new component needs adding to the `declarations` list in the `NgModule` decorator on the module class. After that, just use it inside the root component and bob's your uncle.

### Angular directives

HTML can contain the following elements from Angular:
* *Component directives:* usage of tags for components
* *Attribute directives:* Attributes attached to Angular components to change their default behaviour
* *Structural directives:* Also attributes, but they cause the component's rendered structure to change.
    An example of this is `ngFor`: `<my-component *ngFor="let item of myItemList"> {{item.name}} </my-component>` 

Directives such as the above are made available to the components' HTML by importing them into the module's TS.

## Exchanging data with components

Data binding between TS classes and the HTML elements (ie. the DOM tree) can be one-way or two-way.

One way binding is achieved with the following syntaxes:

### Interpolation
Embed a typescript expression inside quotes: `{{expression}}` 

The expression will be evaluated, and the DOM tree updated whenever the resolved value of the expression changes. If the expression is a function call, it needs to be idempotent and have no side effects.

### TS-to-DOM binding - one way

Also known as "property binding".

Assigning an expression to a DOM attribute in square brackets causes the DOM to be udpated when the value of the expression changes. Like interpolation, if the expression is a function call, it needs to be idempotent and have no side effects.

```html
<input [value]="getFirstName()"/>
```

This binding is a binding to the DOM attribute, NOT to the HTML attribute. HTML elements sometimes have attributes that are not available in the DOM, and vice versa. In all cases where the rendered DOM and the HTML have attributes of the same name, the value supplied to the HTML is only used to _initialise_ the attribute; altering it after the fact (for example by assigning it to an interpolated value) will have no effect on the UI.

### DOM-to-TS - one way

Also known as "event binding". Binding some Typescript to an event. After an event is triggered, Angular change detection runs to update the UI for any changes in the model.

For event binding, you can specify arguments to pass to the bound handler. The code below is turned into a function by Angular, and executed in the context that includes `$event`. 

```html
<input [value]="lastName" type="text" name="LName" 
        (change)="setLastName($any($event.target).value)"></p>
```

Note how `$event.target` has to be cast to an `any` to be able to use `value`. This is because `$event.target` is declared as type `EventTarget` in the browser API, and the `value` attrib isn't present on that interface. 

In actuality, the target of the event that gets passed to an `input` element's `change` event is always going to be of type `HTMLInputElement`, which does have a `target` attribute. A more type-safe way of achieving the above (recommended in Mozilla docs) is to have a method on the component class that knows how to correctly cast the event target:

```ts
  getValue(event: Event): string {
    console.log(event)
    return (event.target as HTMLInputElement).value;
  }
```

Then call that from the HTML template:

```html
    <p>Last name: 
        <input [value]="lastName" type="text" name="LName" 
        (change)="setLastName(getValue($event))"></p>
```

## Two-way binding

In two-way binding, the zen is that we're binding a named attribute on an element to a value provided externally to that element. For this to work, Angular requires that if the element has an attribute named (for example) `val`, then it will also have an event named `valChange`. [Docs on how to write attributes to fit this pattern are here.](https://angular.io/guide/two-way-binding) Spoiler: you use `Input` and `Output` decorators to tag values and emitters in the class. 

If the element follows this pattern, then you can do the following to have MyCustomElement continuously updated as the value changes, and have the value update as MyCustomElement changes it. 

```html
<MyCustomElement [(val)]="someValue">
``` 

This is shorthand for the following (assuming that `MyCustomElement`'s output emitter is just emitting the changed value):

```html
<MyCustomElement [val]="someValue" (changeVal)="someValue = $event">
```

However, none of the standard HTML elements follow this pattern, and so Angular provides a way to hook those elements' attributes up to your model with an attribute directive called `ngModel`.

### Using ngModel

It's included in the Angular Forms module, and you add it into your module file with 

```ts
import { FormsModule } from '@angular/forms';

```

You can bind input elements to component model data with the `ngModel` directive: 

```html
 <input [(ngModel)]="this.firstName" type="text" name="FName">
```

If you need to receive the changes as events, you can split the property and event binding:

```html
 <input [ngModel]="this.firstName" (ngModelChange)="setFirstName($event)" 
    type="text" name="FName">
```

This syntax will create an instance of `NgModel` and connect it to your component (one instance for every instance of the component). 

`ngModel` manages the magic of getting values into and out of HTML elements by having an implemented `ControlValueAccessor` for every type of standard element, which knows how to access the value for each element type. 

### Validation

HTML5 included the ability to validate the values in HTML `input` elements.  Two types were introduced (in addition to all the others!) that include validation: `email` and `URL`.

Additionally, a number of validation attributes were included: 
* `pattern` - provides a regex
* `min`, `max` - for number types
* `required`
* `step`
* `minlength`, `maxlength` - for string types

The validation functionality on these controls is triggered by calling `checkValidity` on the element or on its form, or by submitting the form. 

One of the things that `NGModel` directive does is it starts managing some classes on the form's input elements, and on the form itself:
* `ng-valid` / `ng-invalid`: set according to if the element's value matches its validation criteria.
* `ng-touched` / `ng-untouched`: set according to if the element has ever had focus
* `ng-pristine` / `ng-dirty`: set according to if the element's value has ever been modified. 

The form will take on the high-watermark of these classes across all of its input elements; ie. if one element has been touched, the form will be marked as touched.

These values are also available through the `ngModel` directive, which is accessible by naming a reference to it on the element:

```html
<input [(ngModel)]="this.firstName" type="text" name="FName" #f="ngModel">
  <span [hidden]="!(f.dirty && f.errors['required'])">Enter the first name</span> 
```
Note how there's no need to check if `f.errors` is non-null before subscripting it; the Angular authors saw that the most common case in the expression syntax was that you wanted to check chained attributes for nullness, and short-circuit a return of `undefined` if necessary. They implemented the syntax so that this is how it works - essentially there an implicit `?.` in every chaining.

You can add a style to the module's css (`src/styles.css`) that will apply for controls that have the invalid, touched and dirty classes set, but do not have an `ngForm` atttribute:

```css
.ng-invalid.ng-touched.ng-dirty:not([ngForm]) {
    border: 1px solid red
}
```

## CSS Basics Refresh

* Matching by class: `.class1.class2.class3 {styles}` does an "and"
* Matching by attribute: `div[bob="boo"] {}`
  * Regex style matching operators: `div[bob^="http"] {}`, `dev[bob$=".txt"] {}`
  * Case sensitive: `div[bob="boo" s] {}`, insensitive: `div[bob="boo" i] {}`
  * Multiple-matchers means `and`: `div[bob="boo" s] div[bob="boo" i] {}`
* ID matchers: `#myID {}`
* You can achieve `or` behaviour with multiple, comma-delimited selectors for a single style.
* the `:not()` contains multiple selectors to trim the resulting set of elements.

## `@ViewChild` decorator

Used to inject a reference to a DOM element into a TS class. The references get populated as the UI initialises, and are only available when AfterViewInit lifecycle hook fires; initialisation stuff should be done in there.

The `ViewChild` selectors can only see components inside the current components - not grandchildren or parents.
* Select by class: `@ViewChild(MySubComponent) mySub: MySubComponent`
* Select by reference from HTML: `<h2 #title>STuff</h2>`, then `@ViewChild('title') title: ElementRef`; also works for both `ngForm` and `ngModel`.

[More docs here.](https://blog.angular-university.io/angular-viewchild/)
## Deployment

Build the app with `npx ng build -c production`; gives you a `dist` folder with all required artefacts.