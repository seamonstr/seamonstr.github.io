---
title: Angular routing
---

## Intro

Routing in Angular is conceptually identical to routing in a web server.

With a single page app, you don't want the page to reload every time the user goes somewhere new, but you _do_ want the URL to update meaningfully, and you do want to enable deep-linking to content in the app.

Angular solves this with routing: the routing module provides a listener that hooks any attempt to navigate to a new URL, and instead routes that URL to a componenent internally in the app.

In other words, when building the app you associate URLs with components. When a user clicks a link in the app, your URL -> component configuration is used to figure out which component should be used, and that component is loaded and rendered.

There is a [great set of articles on the topic here](https://indepth.dev/posts/1023/the-three-pillars-of-angular-routing-angular-router-series-introduction).

## Getting started with routing

The steps to add routing to your app are:

1. Add the routing module to your app:
    1. Javascript import the module into `app.component.ts`: `import { RouterModule, Routes } from '@angular/router';
    1. Import and configure the router module into the configuration of the app component. The route module's `forRoot` function takes a URL -> Component mapping.  **Note** how each of the URLs specified are URL *segments*, and can have multiple names with slashes, not just one.

    ```ts

    const routes: Routes = [
      {path: 'first', component: FirstComponent},
      {path: 'second', component: SecondComponent},
      {path: 'third/stuff/things', component: ThirdComponent}
    ];

    /* ... */
      // In the NgModule decorator:
      imports: [
        // ...
        RouterModule.forRoot(routes)
      ],

    ```
1. Use the `routerLink='<internal app URL>` attribute directive in any element to make that element into a routing link. Simple examples do this in the app component, using that as the always-present root and display host.
1. Add a `<router-outlet></router-outlet>` tag to the app component where the result of the routing is to be rendered. The rendered content will be added to the DOM as a _sibling_ to the `router-outlet` tag.

## Child URLs == child components

Each of the URL segments provided to `forRoot` is handled however you specify in the config: directed to a component, redirected to a different URL.  The handler is called a "routable state", and the combination of URL and state for it is a `Route`.

Each `Route` takes an array `children` of further `Route`s. This creates a tree of `Route`s; to build the URL to reach any given routable state, you concatenate the URLs together from the root down, separated with `/`.

The trick here is that the router **does not** take the URL, work out which ultimate `Route` it corresponds to and then render just that as a sibling to the `router-outlet`; rather, it renders every state's component on the way down the tree, and each component is responsible for providing a `router-outlet` in its template for its child to be rendered into.

So, if you have the following router configuration:

```ts
     const routes: Routes = [
      {path: 'first', component: FirstComponent, children:[
        {path: 'second', component: SecondComponent, children: [
          {path: 'third', component: ThirdComponent}
        ]}
      ]}
    ];
```

with a `app.component.html` that looks like:

```html
<h1>My Page</h1>
<p>Child goes here: </p>
<div>
<router-outlet></router-outlet>
</div>
```

...and you go to `https://myservice/first/second/third`, then semantically you'd _expect_ that to mean "please render ThirdComponent at the point marked by `router-outlet`.

Actually, what's it's saying is "Render `FirstComponent` at `router-outlet`; tell `FirstComponent` to render `SecondComponent`, and `SecondComponent` to render `ThirdComponent`". 

Each of the components that expect to render a child provide a `router-outlet` to mark where to render it:

```html
<h2>First Component</h2>
<div>
  <router-outlet></router-outlet>
  <!-- SecondComponent will go here -->
</div>
```

```html
<h3>Second Component</h2>
<div>
  <router-outlet></router-outlet>
  <!-- ThirdComponent will go here -->
</div>
```

```html
<h4>Third Component</h2>
<!-- No need for a router-outlet; ThirdComponent renders no children. -->
```

## pathMatch='full' and redirects

Before a URL will match a component, the whole of the URL needs to be consumed by the matched branch of `Route`s. For redirects, however, the redirect will be processed as soon as you have a successful partial match.

Eg. consider a URL of `https://myservice/one/two/three`, and the following config@

```ts
ROUTES = [
  {path: 'one', redirectTo: 'bananas'},
  {path: 'one/two/three', component: 'HurrahMatch'}
]
```

The second path will never match anything, because the `'one'` will partially match and immediately redirect. The search process will begin again with a path of `bananas`.

Enter `pathMatch`. When specified on a `Route` in a routing config, `pathMatch='full'` means that that route only matches if the whole of that as-yet unmatched URL matches. In other words, the `Route` needs to consume the whole URL, and any children of that route would be ignored.

In our above example, `one/two/three` would _not_ match `one`, because there would still be `two/three` unconsumed. Thus, `pathmatch='full'` is pretty much only useful to disable the partial-matching behaviour of redirects.