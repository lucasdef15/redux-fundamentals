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

```js
import { createStore } from "redux";
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
