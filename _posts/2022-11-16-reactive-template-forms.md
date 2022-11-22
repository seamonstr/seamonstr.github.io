---
title: Reactive and Template Driven Forms
---

## Misc

* Package names that begin with a `@somevalue` are in a namespace. A namespace is controlled by a single organisation, so you can be sure items in it are legit.
* The `declare` keyword is used to declare a type, semantically identical to a `.h` file in C. You see it used in `.d.ts` files, which are type declarations for the actual implementation.
* The `keyof` typescript operator gives you back a type which is the union of all types of all keys of the specified object.

### Setting CSS classes on an element programmatically

Angular provides an attribute directive, `[ngClass]`, which enables you to conditionally set classes on an element.

It has three options for use:

Set classes provided in a string:
```html
<input type="text" [ngClass]="error mt-3">
```

Set classes provided in an array of strings:
```html
<input type="text" [ngClass]="['error', 'mt-3']">
```

Set classes provided in an object's attributes. The key of the attribute is the class; the value is some JS that is evaluated. The class is set if the JS evaluates to truthy.

```html
<input name="breed" [formControl]="breed" type="text" [ngClass]="{'error': breed.value=='poodle', 'mt-3': breed.value='alsatian'">
```


## Reactive- vs. Template-driven approach

In the labs covered earlier, we used a template-driven approach to generate the table of values.  This is characterised by for-loop generated rows, injecting values with `{{}}` interpolation operators, getting data into and out of controls and the model with directives.

### Template-driven forms
Relies on the directives in the `FormsModule`: `NgModel`, `NgForm`, `NgModelGroup`. If the solutions uses these, it's template driven.

The UI element has the `NgModel` directive on it. This creates a `ValueAccessor` and `NgModel` instance on that element. The `NgModel` has an `FormControl` instance embedded in it. 

When a user modifies the value in the UI, the `ValueAccessor` recieves the `input` event. It calls:
1. `FormControl.setValue()`, so that subscribers get the event, and 
1. `NgModel.viewToModelUpdate()` which triggers the configured code to update the model value in the component.

When the value is modified in the component:
1. Change detection figures out that a value that is an input to a control. The control's `NgModel` instance's ngOnChanges hook is called.
1. `ngOnChanges` queues a task to update its embedded `FormControl`.
1. The `FormControl` task emits the change through its `valueChanges` observable, which is caught by the `ValueAccessor`
1. The `ValueAccessor` updates the input element.

### Reactive forms
Relies on the directives in `ReactiveFormsModule`.

Reactive forms are directly connected to the underlying model of the component: the UI element is connected directly to a corresponding FormControl instance in the class which contains the data. Validation is likewise created in the class, rather than manually bound with directives and showing/hiding UI elements.

To make this connection:
* In the TS class, create an attribute assigned to a new instance of FormControl.
* In the HTML, declare the UI element. use the `[formControl]` directive, assigning it to a string containing the corresponding attribute name on the TS class.

When a user of the form changes the value in the control, the UI element's [`input`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement#input_events) event is emitted and caught by the `FormControl`'s value accessor. The form control's value is update, and the `FormControl` then emits the new value through its `valueChanges` observable.

For changes going the other way (from code to the UI), when the `FormControl`'s `setValue` method is called, the `FormControl`'s value is updated. `FormControl` emits the value through the `valueChanges` observable. The control's value accessor is a subscriber to that observable, and updates the UI element.

## Working with reactive forms

You can create a template form using the `ng` util:
```bash
npx ng generate component MyForm
```

You can add individual controls to the component as above, but the more common need is for groups of controls making up a form.

For this, you create a FormGroup, passing it an object containing the controls in the group:

```ts
personForm = new FormGroup({
    name = new FormControl(''),
    surname = new FormControl(''),
})
```
In code you can access the individual controls through `personForm.controls.name`, etc. 

In the template, you use a `[FormGroup]` on the form container, and then associate each control with its TypeScript control with the `formControlName` attribute on the input element:
```html
<form class="card-body" [FormGroup]="personForm" (ngSubmit)="onSubmit()">
    <label for="name">Name: </label>
    <input id="name" formControlName="name" type="text">
    <label for="surname">Surname: </label>
    <input id="surname" formControlName="surname" type="text">
    <button type="submit">Submit</button>
</form>
```

To logically group items, you can add `FormGroup`s as items in the root `FormGroup`:
```ts
personForm = new FormGroup({
    name = new FormControl(''),
    surname = new FormControl(''),
    address = new FormGroup({
        street = new FormControl(''),
        town = new FormControl(''),
        postcode = new FormControl(''),
    })
})
```

The sub-`FormGroup` is tagged with `formGroupName` (ie. not its own `[FormGroup]` directive), and needs to be nested inside the element tagged with `[FormGroup]`:
```html
<!-- Previous div goes here -->
<div formGroupName="address">
    <!-- Controls -->
</div>
```

### FormBuilder

There is a shortcut service for generating forms called FormBuilder.

You get a reference to it by specifying it in the constructor: ```constructor(private fb: FormBuilder) {}`

You use the `fb` to generate a tree of controls with a simple object as input, just in the class declaration:
```ts
  profileForm = this.fb.group({
    firstName: ['', Validators.required],
    lastName: ['', [Validators.required, Validators,minLength(3)]],
    address: this.fb.group({
      street: '',
      city: '',
      state: '',
      zip: ''
    }),
  });
```
The constructor for a FormControl takes
* A string `value`, or a control state object
* A validator or array of validators
* An async validator or array async validators

The `fb` can take a string (for the value), or a vector of args.


You still have to build your HTML, though.

### Submitting the form

The first option is to place a button outside the `form` element, with a handler that does whatever processing you want.

Second, to use the submit button of the form (which means you get the "enter to submit" behaviour), you need to ensure that:
1. The button is inside the form
2. The `form` element or one of its parents is both tagged with `[FormGroup]` and has a `(ngSubmit)="onSubmit()"` event binding.

Best practise for processing is to use `EventEmitter` to throw an event from the `onSubmit` handler; this keeps the form loosely coupled.

### Validation

Unlike in template-drive forms, you specify the validators you want on the `FormControl` objects directly by passing them into the `FormControl` constructor.  

The `Validators` class has a number of methods on it that return you a closure that is your configured validator: `min`, `required`, etc. A validator is just a function pointer, so you can supply your own too.

The aggregate validation statuses of the individual `FormControl`s bubble up to the `FormGroup` containing them.

You then add some code to the template to specify how to handle the validation status. Eg. I added a utility function to calculate the `css` styles needed in TypeScript:
```ts
    getControlClass(control: FormControl): string {
        if (control.pristine || control == this.focussed)
            return '';

        return control.dirty && control.invalid ? 'is-invalid' : 'is-valid';
    }
```

...and then called that from the template with an `[ngClass]` directive:
```html
    <input class="form-control" formControlName="password" 
        (focus)="onFocus($event)"
        [ngClass]="getControlClass(loginForm.controls.password)"
        type="text" name="password">
```
