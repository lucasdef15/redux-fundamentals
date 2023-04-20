# Redux Store

The Redux store brings together the state, actions, and reducers that make up your app. The store has several responsibilities:

- Holds the current application state inside
- Allows access to the current state via store.getState();
- Allows state to be updated via store.dispatch(action);
- Registers listener callbacks via store.subscribe(listener);
- Handles unregistering of listeners via the unsubscribe function returned by store.subscribe(listener).

It's important to note that **you'll only have a single store in a Redux application.** When you want to split your data handling logic, you'll use reducer composition and create multiple reducers that can be combined together, instead of creating separate stores.

## Creating a Store

Every Redux store has a single root reducer function.

The Redux core library has a **createStore API** that will create the store. Add a new file called **store.js**, and import **createStore** and the root reducer. Then, call **createStore** and pass in the root reducer:
//this.create

```js
import { createStore } from "redux";
this;
import rootReducer from "./reducer";

const store = createStore(rootReducer);

export default store;
```

## Loading Intial State

**createStore** can also accept a **preloadedState** value as its second argument. You could use this to add initial data when the store is created, such as values that were included in an HTML page sent from the server, or persisted in **localStorage** and read back when the user visits the page again, like this:

```js
import { createStore } from "redux";
import rootReducer from "./reducer";

let preloadedState;
const persistedTodosString = localStorage.getItem("todos");

if (persistedTodosString) {
  preloadedState = {
    todos: JSON.parse(persistedTodosString),
  };
}

const store = createStore(rootReducer, preloadedState);
```

# Dispatching Actions

Remember, every time we call store.dispatch(action):

- The store calls rootReducer(state, action)
- That root reducer may call other slice reducers inside of itself, like todosReducer(state.todos, action)
- The store saves the new state value inside
- The store calls all the listener subscription callbacks
- If a listener has access to the store, it can now call store.getState() to read the latest state value

# Inside a Redux Store

It might be helpful to take a peek inside a Redux store to see how it works. Here's a miniature example of a working Redux store, in about 25 lines of code:

```js
function createStore(reducer, preloadedState) {
  let state = preloadedState;
  const listeners = [];

  function getState() {
    return state;
  }

  function subscribe(listener) {
    listeners.push(listener);
    return function unsubscribe() {
      const index = listeners.indexOf(listener);
      listeners.splice(index, 1);
    };
  }

  function dispatch(action) {
    state = reducer(state, action);
    listeners.forEach((listener) => listener());
  }

  dispatch({ type: "@@redux/INIT" });

  return { dispatch, subscribe, getState };
}
```

This small version of a Redux store works well enough that you could use it to replace the actual Redux createStore function you've been using in your app so far. (Try it and see for yourself!) The actual Redux store implementation is longer and a bit more complicated, but most of that is comments, warning messages, and handling some edge cases.

As you can see, the actual logic here is fairly short:

- The store has the current state value and reducer function inside of itself
- getState returns the current state value
- subscribe keeps an array of listener callbacks and returns a function to remove the new callback
- dispatch calls the reducer, saves the state, and runs the listeners
- The store dispatches one action on startup to initialize the reducers with their state
- The store API is an object with {dispatch, subscribe, getState} inside

To emphasize one of those in particular: notice that getState just returns whatever the current state value is. That means that by default, nothing prevents you from accidentally mutating the current state value! This code will run without any errors, but it's incorrect:

```js
const state = store.getState();
// ❌ Don't do this - it mutates the current state!
state.filters.status = "Active";
```

In other words:

- The Redux store doesn't make an extra copy of the state value when you call getState(). It's exactly the same reference that was returned from the root reducer function
- The Redux store doesn't do anything else to prevent accidental mutations. It is possible to mutate the state, either inside a reducer or outside the store, and you must always be careful to avoid mutations.

One common cause of accidental mutations is sorting arrays. Calling array.sort() actually mutates the existing array. If we called const sortedTodos = state.todos.sort(), we'd end up mutating the real store state unintentionally.

# Configuring the Store

We've already seen that we can pass rootReducer and preloadedState arguments to createStore. However, createStore can also take one more argument, which is used to customize the store's abilities and give it new powers.

Redux stores are customized using something called a store enhancer. A store enhancer is like a special version of createStore that adds another layer wrapping around the original Redux store. An enhanced store can then change how the store behaves, by supplying its own versions of the store's dispatch, getState, and subscribe functions instead of the originals.

For this tutorial, we won't go into details about how store enhancers actually work - we'll focus on how to use them.

# Creating a Store with Enhancers

two small example store enhancers:

```js
export const sayHiOnDispatch = (createStore) => {
  return (rootReducer, preloadedState, enhancers) => {
    const store = createStore(rootReducer, preloadedState, enhancers);

    function newDispatch(action) {
      const result = store.dispatch(action);
      console.log("Hi!");
      return result;
    }

    return { ...store, dispatch: newDispatch };
  };
};

export const includeMeaningOfLife = (createStore) => {
  return (rootReducer, preloadedState, enhancers) => {
    const store = createStore(rootReducer, preloadedState, enhancers);

    function newGetState() {
      return {
        ...store.getState(),
        meaningOfLife: 42,
      };
    }

    return { ...store, getState: newGetState };
  };
};
```

- **sayHiOnDispatch**: an enhancer that always logs 'Hi'! to the console every time an action is dispatched
- **includeMeaningOfLife**: an enhancer that always adds the field meaningOfLife: 42 to the value returned from getState()

we can use one of those enhancers, we'll import it, and pass it to createStore:

//src/store.js

```js
import { createStore } from "redux";
import rootReducer from "./reducer";
import { sayHiOnDispatch } from "./exampleAddons/enhancers";

const store = createStore(rootReducer, undefined, sayHiOnDispatch);

export default store;
```

We don't have a preloadedState value here, so we'll pass undefined as the second argument instead.

Next, let's try dispatching an action:

//src/index.js

```js
import store from "./store";

console.log("Dispatching action");
store.dispatch({ type: "todos/todoAdded", payload: "Learn about actions" });
console.log("Dispatch complete");
```

The **sayHiOnDispatch** enhancer wrapped the original **store.dispatch** function with its own specialized version of **dispatch**. When we called **store.dispatch()**, we were actually calling the wrapper function from **sayHiOnDispatch**, which called the original and then printed 'Hi'.

Now, let's try adding a second enhancer. We can import **includeMeaningOfLife** from that same file... but we have a problem. **createStore** only accepts one enhancer as its third argument! How can we pass two enhancers at the same time?

What we really need is some way to merge both the **sayHiOnDispatch** enhancer and the **includeMeaningOfLife** enhancer into a single combined enhancer, and then pass that instead.

Fortunately, **the Redux core includes a compose function that can be used to merge multiple enhancers together.** Let's use that here:

//src/store.js

```js
import { createStore, compose } from "redux";
import rootReducer from "./reducer";
import {
  sayHiOnDispatch,
  includeMeaningOfLife,
} from "./exampleAddons/enhancers";

const composedEnhancer = compose(sayHiOnDispatch, includeMeaningOfLife);

const store = createStore(rootReducer, undefined, composedEnhancer);

export default store;
```

Now we can see what happens if we use the store:

//src/index.js

```js
import store from "./store";

store.dispatch({ type: "todos/todoAdded", payload: "Learn about actions" });
// log: 'Hi!'

console.log("State after dispatch: ", store.getState());
// log: {todos: [...], filters: {status, colors}, meaningOfLife: 42}
```

So, we can see that both enhancers are modifying the behavior of the store at the same time. **sayHiOnDispatch** has changed how dispatch works, and **includeMeaningOfLife** has changed how **getState** works.

Store enhancers are a very powerful way to modify the store, and almost all Redux apps will include at least one enhancer when setting up the store.

If you don't have any preloadedState to pass in, you can pass the enhancer as the second argument instead:

```js
const store = createStore(rootReducer, storeEnhancer);
```

# Middleware

Redux uses a special kind of addon called **middleware** to let us customize the dispatch function.

**Redux middleware provides a third-party extension point between dispatching an action, and the moment it reaches the reducer.** People use Redux middleware for logging, crash reporting, talking to an asynchronous API, routing, and more.

First, we'll look at how to add middleware to the store, then we'll show how you can write your own.

## Using Middleware

We already saw that you can customize a Redux store using store enhancers. Redux middleware are actually implemented on top of a very special store enhancer that comes built in with Redux, called **applyMiddleware**.

Since we already know how to add enhancers to our store, we should be able to do that now. We'll start with **applyMiddleware** by itself, and we'll add three example middleware that have been included in this project.

//src/store.js

```js
import { createStore, applyMiddleware } from "redux";
import rootReducer from "./reducer";
import { print1, print2, print3 } from "./exampleAddons/middleware";

const middlewareEnhancer = applyMiddleware(print1, print2, print3);

// Pass enhancer as the second arg, since there's no preloadedState
const store = createStore(rootReducer, middlewareEnhancer);

export default store;
```

As their names say, each of these middleware will print a number when an action is dispatched.

What happens if we dispatch now?

//src/index.js

```js
import store from "./store";

store.dispatch({ type: "todos/todoAdded", payload: "Learn about actions" });
// log: '1'
// log: '2'
// log: '3'
```

So how does that work?

**Middleware form a pipeline around the store's dispatch method.** When we call **store.dispatch(action)**, we're actually calling the first middleware in the pipeline. That middleware can then do anything it wants when it sees the action. Typically, a middleware will check to see if the action is a specific type that it cares about, much like a reducer would. If it's the right type, the middleware might run some custom logic. Otherwise, it passes the action to the next middleware in the pipeline.

_Unlike_ a reducer, **middleware can have side effects inside**, including timeouts and other async logic.

In this case, the action is passed through:

1. The **print1** middleware (which we see as **store.dispatch**)
2. The **print2** middleware
3. The **print3** middleware
4. The original **store.dispatch**
5. The root reducer inside **store**

And since these are all function calls, they all return from that call stack. So, the print1 middleware is the first to run, and the last to finish.

## Writing Custom Middleware

We can also write our own middleware. You might not need to do this all the time, but custom middleware are a great way to add specific behaviors to a Redux application.

**Redux middleware are written as a series of three nested functions.** Let's see what that pattern looks like. We'll start by trying to write this middleware using the **function** keyword, so that it's more clear what's happening:

```js
// Middleware written as ES5 functions

// Outer function:
function exampleMiddleware(storeAPI) {
  return function wrapDispatch(next) {
    return function handleAction(action) {
      // Do anything here: pass the action onwards with next(action),
      // or restart the pipeline with storeAPI.dispatch(action)
      // Can also use storeAPI.getState() here

      return next(action);
    };
  };
}
```

Let's break down what these three functions do and what their arguments are.

- exampleMiddleware: The outer function is actually the "middleware" itself. It will be called by applyMiddleware, and receives a storeAPI object containing the store's {dispatch, getState} functions. These are the same dispatch and getState functions that are actually part of the store. If you call this dispatch function, it will send the action to the start of the middleware pipeline. This is only called once.
- wrapDispatch: The middle function receives a function called next as its argument. This function is actually the next middleware in the pipeline. If this middleware is the last one in the sequence, then next is actually the original store.dispatch function instead. Calling next(action) passes the action to the next middleware in the pipeline. This is also only called once
- handleAction: Finally, the inner function receives the current action as its argument, and will be called every time an action is dispatched.

Any middleware can return any value, and the return value from the first middleware in the pipeline is actually returned when you call **store.dispatch().** For example:

```js
const alwaysReturnHelloMiddleware = (storeAPI) => (next) => (action) => {
  const originalResult = next(action);
  // Ignore the original result, return something else
  return "Hello!";
};

const middlewareEnhancer = applyMiddleware(alwaysReturnHelloMiddleware);
const store = createStore(rootReducer, middlewareEnhancer);

const dispatchResult = store.dispatch({ type: "some/action" });
console.log(dispatchResult);
// log: 'Hello!'
```

Let's try one more example. Middleware often look for a specific action, and then do something when that action is dispatched. Middleware also have the ability to run async logic inside. We can write a middleware that prints something on a delay when it sees a certain action:

```js
const delayedMessageMiddleware = (storeAPI) => (next) => (action) => {
  if (action.type === "todos/todoAdded") {
    setTimeout(() => {
      console.log("Added a new todo: ", action.payload);
    }, 1000);
  }

  return next(action);
};
```

This middleware will look for "todo added" actions. Every time it sees one, it sets a 1-second timer, and then prints the action's payload to the console.

## Middleware Use Cases

So, what can we do with middleware? Lots of things!

A middleware can do anything it wants when it sees a dispatched action:

- Log something to the console
- Set timeouts
- Make asynchronous API calls
- Modify the action
- Pause the action or even stop it entirely
- and anything else you can think of.

In particular,** middleware are intended to contain logic with side effects.** In addition, **middleware can modify dispatch to accept things that are not plain action objects.** We'll talk more about both of these in Part 6: Async Logic.

# Redux DevTools
