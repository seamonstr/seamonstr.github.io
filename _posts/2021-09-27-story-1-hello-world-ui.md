---
layout: post
title: React Hello World
---

# First step: A simple React app building and checked into GitHub

My first task for this project is to get a small, static React app running. Because I'm coming to this only lightly informed at best, I'm first going to need to learn a lot about React, SPAs and how to host them (especially in AWS, given that's my target).

For this post, I'm going to be working on the getting a "Hello, World!" running on my local machine, and checked in to GitHub in a way that it can built elsewhere.

Tne next steps will be getting it hosted and getting CI/CD working; first things first, though!

## Getting started with React

So first I did a google for "react project structure", and that returned a lot of results, [like this one](https://www.taniarascia.com/react-architecture-directory-structure/).  That seemed okay, but it said a whole lot of words that didn't make sense; words like "redux as features slices" and "routing" and things; also, there didn't seem to be an `index.html`, but there did seem to be an `index.js`which leaves me somewhat at sea.

So, I had more of a look for a [beginners guide to react](https://www.freecodecamp.org/news/react-beginner-handbook/) so that I could make some headway.

### First - what is a react app?

React is a javascript library designed to make UI dev easier; it's supposed to make it easier to manage a complicated UI and its current state.  Like most UI frameworks, it does this by dividing the overall UI into components and defining ways of communicating between them.

### Creating react apps

While react is a browser-side library (yes, I know react-native exists, but I'm focussing on web here), creating a react app involves an elaborate build pipeline and a complicated set of source directories on the dev workstation (and build machine!). The contents of the source tree are read by the pipeline, which creates a set of HTML, styles and JS that can be used by the browser.

So first, I use a tool called `create-react-app` to create a source tree with a default app in it.  This is a tool that runs in nodeJS. NodeJS is a locally installable javascript runtime which executes any javascript code you like.  People use it to create lightweight servers, utilities, all sorts of things.

NodeJS includes a utility called `npx` which enables the running of scripts from the NodeJS package repository without installing them: `npx` downloads the script and its dependencies, runs it and then deletes it again.  

So, from a logical location (ie. in my `code` dir which is my root for all coding work), I run
```
npx create-react-app todo-web
``` 

...which then does a **whole** bunch of stuff.  At the end I have a directory named `todo-web` full of files that is clearly designed to be a git repo - it even has a README.md in it.  It contains a huge amount of code: in excess of 40,000 JS files.

Incidentally, this answered my question about the standard structure for a React app - the root of my repo looks like this:
- **build:** build destination for my app
- **node_modules:** JS library and utility code; provided by create-react-app
- **src:**  duh
- **public:** static assets for the result website (HTML, CSS, images, ...)

### Running my new react app locally

Running `npm start` in my new app's directory builds the app and starts up a nodeJS webserver to serve it; it even flips to a browser and loads the local URL of the server.

### Building the react app

All well and good to run the app locally, but how do I build a deployable version of it for pushing to a test or live environment?

If I run `npm build` in that directory, it creates a `build` directory with a runnable version of the app in it.  The compiled version of the app is "minified", which means that large parts of the code have been gzipped inside their JS files, and a loader is generated which knows how to unzip the right bits at the right time to make the whole thing work again.

This is the build process I'll need to trigger from GitHub later.

## Learning some React

Okay, now that I have a locally running React app, I'd like to learn a bit about how the framework works.

Looking at my new app shows that it's got a file called `index.js` in the `src` directory. This renders a component of type "App" into the root of the page's DOM; in the same directory is a file called `App.js`, which declared a component of type "App".  

The crucial parts of React that I need to learn are components, JSX, State and Props.

### Components

A component in modern React is a javascript function that returns a JSX object (ie. one of the HTML-ish structures in React code - see the bit on JSX below).  You can then instantiate the component inside other components with some JSX markup using the function name as the HTML tag.

For example, this snippet shows an "App" component that embeds an "EmbeddedStuff" component:
```
function App() {
  return <p>Welcome! <EmbeddedStuff/> </p>
}

function EmbeddedStuff() {
  return <p>An embedded thingy.</p>
}
``` 

Looking at the above, you can see that we're kinda creating classes without classes, but with functions. It turns out that React components used to be written with classes (and still are!), but the authors decided that they could actually get on better with just functions - hence 'functional react components' like the above.

The issues are how do we do instance variables, and how do we add behaviour (especially polymorphic behaviour) to the component.  At this point I did some side-reading and found [this post on react functional components](https://www.robinwieruch.de/react-function-component) which I used in addition to the above tutorial.

### JSX - defining components 

JSX is a hybrid language that React apps are written in. It's javascript, but you can natively embed HTML-esque code into the functional components, like this:
```
function WelcomeMessage() {
  return <p>Welcome!</p>
}
```

I started off thinking that this was just some syntactical sugar to avoid having to manage template text in either separate files or as elaborately string-interpolated constants.  As I've read more, though, I've realised that it's a actually a really elegant way to join the template and behavioural aspects of the component into one thing.  Styling is still managed separately (in CSS or other methods).

This method acknowledges that the structure of a component and the behaviour of a component are tightly coupled, and this is a single syntax that enables managing them in the same place.  It makes understanding and reasoning about them much easier.

Just as a point - when I first saw the above I thought the functional component definition was actually a kind of constructor, especially when you see the [State] section below.  It kinda is that, in that you initialise a bunch of state in it, but actually the component's function is the `render` function of the component.

### State

"State", in this context, refers to how you store the internal state of your UI: the clicked status of a button, the visibility status of a sidebar, etc. It seems when React was young the way to capture this state was how we do it in most other languages: you write a class for the component, and you add instance properties that store the state for that instance.

It seems JS classes are pretty peculiar and hard to understand, so they moved to the function syntax I mentioned above in the [Components] section. However, this makes storing state really awkward - no convenient properties on objects to put stuff in! 

So, React introduced the concept of "hooks", which are functions that you can call from inside your function component in order to access React state and features.  It means that you don't have to convert your functional component into a class component in order to have state.

The `state` hook is the one that we're interested in here. From within a functional component you call `useState`, passing it the initial value of the state variable you want to create. The two values returned are hooked into React's internals to allow you to manipulate the state. 

Note the creation of a closure for the onClick handler; that closure will contain the appropriate values of `myString` and `setMyString` in its scope. See the bit below on [Closures in Javascript].

```
const MyComponent = () => {
  const [myString, setMyString] = useState("First string!");
  
  return (
    <div>
      <p>The value here is {myString}.</p>
      <button onClick={() => setMyString(myString + " Another bit!")}>
        Click, plz
      </button>
    </div>
  );
} 
```

#### Closures in Javascript

This is when I hit an element of Javascript (and several other languages that do function pointers) that I didn't know before. Like all structured languages, Javascript has a lexical environment for every scope in a bit of code; if you declare a variable inside a block, it's not available once that block has ended.  So far, so normal.

However, if you declare a function inside a block, and return a reference to the function from the block (eg. with JS's arrow syntax), then that function forever more has a pointer to the lexical environment (ie. the scope) where it was created, and variables created in that scope live on while the function reference is still alive.

For example:
```javascript

function getCounter() {
  let counter = 0; // This variable is in the parent scope of the two closures created below

  // Create and return two closures. Both of them contain a 
  // reference to this scope in its current state, including the "counter" variable.
  // On subsequent calls to these two fuctions, they'll continue to increment the 
  // 'counter' variable initialised in the execution that created the functions.
  return [() => counter ++, () => counter ++];
}

[a, b] = getCounter();

console.log(a()); // Prints "0"
console.log(b()); // Prints "1"
console.log(b()); // Prints "2"
console.log(a()); // Prints "3"

[c, d] = getCounter();

console.log(c()); // Prints "0"
console.log(d()); // Prints "1"

```

React uses closures like this to store state in functional components.

### Props

Props are a way of passing data into the render of a react component. Simply put, a `props` object is passed as the first argument to a component's function, and you can grab data out of it. The props object is populated from the attributes you put into the JSX when you instantiate the component.

```
const MyComponent = (props) => {
  return <p>My text is here: {props.theValueThatImExpecting}</p>;
}

const ParentComponent = () =>
  return <MyComponent theValueThatImExpecting="have some data!"/>;
```

You can use [javascript object destructuring](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment#object_destructuring) and the [super-lean function definitions syntax in JS6](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions) to make this syntactically more explicit:
```
MySecondComponent = ({otherValueImExpecting}) => <p>Some More: otherValueImExpecting</p>;
```

## Conclusion

I covered a lot of ground quite quickly in this part of my project.  I was originally intending to put the hosting and CI/CD bits in here too, but I've split that into the next task so I can keep some momentum going!

I have created my basic react app (including figuring out how to get a NodeJS dev environment running) and tested it locally.

I did a lot of reading about react, learning how components work, what JSX is how how it's used, how component state is managed in the React framework (using React's state storage and JS closures) and how data is passed between components using props.

All this is now checked in to my GitHub repo.

My next step is to decide how I'm going to host the app, test that, and get some CI-CD running to update it as the code changes.