# Async Logic and Data Fetching

## Redux Middleware and Side Effects

By itself , Redux store doesn't know anything about async logic. It only knows how to synchronously dispatch actions, update the state by calling the root reducer function, and notify the UI that something has changed. any asynchronicity has to happen outside the store.

Redux middleware were designed to enable writing logic that has side effects.

## Using Middleware to Enable Async Logic

Let's look at a couple examples of how middleware can enable us to write some kind of async logic that interacts with the Redux store.

One possibility is writing a middleware that looks for specific action types, and runs async logic when it sees those actions, like these examples:

```js
import { client } from "../api/client";

const delayedActionMiddleware = (storeAPI) => (next) => (action) => {
  if (action.type === "todos/todoAdded") {
    setTimeout(() => {
      // Delay this action by one second
      next(action);
    }, 1000);
    return;
  }

  return next(action);
};

const fetchTodosMiddleware = (storeAPI) => (next) => (action) => {
  if (action.type === "todos/fetchTodos") {
    // Make an API call to fetch todos from the server
    client.get("todos").then((todos) => {
      // Dispatch an action with the todos we received
      storeAPI.dispatch({ type: "todos/todosLoaded", payload: todos });
    });
  }

  return next(action);
};
```

## Redux Async Data Flow

So how do middleware and async logic affect the overall data flow of a Redux app?

Just like with a normal action, we first need to handle a user event in the application, such as a click on a button. Then, we call dispatch(), and pass in something, whether it be a plain action object, a function, or some other value that a middleware can look for.

Once that dispatched value reaches a middleware, it can make an async call, and then dispatch a real action object when the async call completes.

Earlier, we saw a diagram that represents the normal synchronous Redux data flow. When we add async logic to a Redux app, we add an extra step where middleware can run logic like AJAX requests, then dispatch actions. That makes the async data flow look like this:

![Alt text](./assets/images/ReduxAsyncDataFlowDiagram.gif)

## Using the Redux Thunk Middleware

As it turns out, Redux already has an official version of that "async function middleware", called the Redux "Thunk" middleware. The thunk middleware allows us to write functions that get dispatch and getState as arguments. The thunk functions can have any async logic we want inside, and that logic can dispatch actions and read the store state as needed.

Writing async logic as thunk functions allows us to reuse that logic without knowing what Redux store we're using ahead of time.

The word "thunk" is a programming term that means "a piece of code that does some delayed work".
