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

### Accessing component data with interpolation

Members of the component class can be referenced in the HTML with `{{ attrib }}` syntax.

### Event binding
Code in the component can be invoked with event binding:

```html
<!-- Will call the component's save() method -->
<p><input type="button" value="Save" (click)="save()"></p>
```

After an event is triggered, Angular change detection runs to update the UI for any changes in the model.

### Property binding

You can bind a property of an HTML element to a calculated value on the component:

```html
<!-- Value could also be "firstName" property -->
<input [value]="getFirstName()"/>
```

The advantage is that this is type aware. You can assign a boolean value and it'll work. If you assign an interpolated value straight to the attribute it calculates a string representation which always evaluates to `false`.


## Deployment

Build the app with `npx ng build -c production`; gives you a `dist` folder with all required artefacts.