# Redux Fundamentals, Concepts and Data Flow

## Background Concepts

Before we dive into some actual code, let's talk about some of the terms and concepts you'll need to know to use Redux.

### Sate Management

This is a small example of "one-way data flow":

- State describes the condition of the app at a specific point in time
- The UI is rendered based on that state
- when something happens (such as a user clicking a button), the state is updated based on what occurred
- The UI re-renders based on the new state

![Alt text](./assets/images/one-way-data-flow.png)

However, the simplicity can break down when we have multiple components that need to share and use the same state, especially if those components are located in different parts of the application. Sometimes this can be solved by "lifting state up" to parent components, but that doesn't always help.

One way to solve this is to extract the shared state from the components, and put it into a centralized location outside the component tree. With this, our component tree becomes a big "view", and any component can access the state or trigger actions, no matter where they are in the tree!

By defining and separating the concepts involved in state management and enforcing rules that maintain independence between views and states, we give our code more structure and maintainability.

This is the basic idea behind Redux: a single centralized place to contain the global state in your application, and specific patterns to follow when updating that state to make the code predictable.

### Immutability

"Mutable" means "changeable". If something is "immutable", it can never be changed.

JavaScript objects and arrays are all mutable by default. If I create an object, I can change the contents of its fields. If I create an array, I can change the contents as well:

```js
const obj = { a: 1, b: 2 };
// still the same object outside, but the contents have changed
obj.b = 3;

const arr = ["a", "b"];
// In the same way, we can change the contents of this array
arr.push("c");
arr[1] = "d";
```

This is called mutating the object or array. It's the same object or array reference in memory, but now the contents inside the object have changed.

**In order to update values immutably, your code must make copies of existing objects/arrays, and then modify the copies.**

We can do this by hand using JavaScript's array / object spread operators, as well as array methods that return new copies of the array instead of mutating the original array:

```js
const obj = {
  a: {
    // To safely update obj.a.c, we have to copy each piece
    c: 3,
  },
  b: 2,
};

const obj2 = {
  // copy obj
  ...obj,
  // overwrite a
  a: {
    // copy obj.a
    ...obj.a,
    // overwrite c
    c: 42,
  },
};

const arr = ["a", "b"];
// Create a new copy of arr, with "c" appended to the end
const arr2 = arr.concat("c");

// or, we can make a copy of the original array:
const arr3 = arr.slice();
// and mutate the copy:
arr3.push("c");
```

**Redux expects that all state updates are done immutably**. We'll look at where and how this is important a bit later, as well as some easier ways to write immutable update logic.

## Redux Terminology

There's some important Redux terms that you'll need to be familiar with before we continue:

### Actions
