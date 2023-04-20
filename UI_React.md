# UI and React

## Basic Redux and UI Integration

Using Redux with any UI layer requires a few consistent steps:

1. Create a Redux store
2. Subscribe to updates
3. Inside the subscription callback:
    i. Gete the current store
    ii. Extract the data needed by this piece of UI
    iii. Update the UI with the data
4. if necessary, render the UI with initial state
5. Respond to UI inputs by dispatching Redux actions

### What is Subscription callbacks?

Subscription callbacks are called so because they are registered with a Redux store using the subscribe method, which effectively subscribes the callback function to the store. Once subscribed, the callback function is called by the store every time the state of the store changes.

The concept of subscription is used because the subscription callback function is essentially "subscribed" to the store, which means it will receive updates whenever the store changes. The callback function is not called directly by the application code, but rather is invoked by the store in response to state changes.

The term "callback" is used because the function is called back by the store when the subscribed event occurs, i.e., when the state changes. In other words, the callback function is called back by the store in response to a particular event, which is the change in the store state.

Overall, the term "subscription callback" is used to describe a function that is subscribed to a store and is called back by the store whenever the state of the store changes.

Let's go back to the the counter app example we saw in Part 1 and see how it follows those steps:

```js
// 1) Create a new Redux store with the `createStore` function
const store = Redux.createStore(counterReducer)

// 2) Subscribe to redraw whenever the data changes in the future
store.subscribe(render)

// Our "user interface" is some text in a single HTML element
const valueEl = document.getElementById('value')

// 3) When the subscription callback runs:
function render() {
  // 3.1) Get the current store state
  const state = store.getState()
  // 3.2) Extract the data you want
  const newValue = state.value.toString()

  // 3.3) Update the UI with the new value
  valueEl.innerHTML = newValue
}

// 4) Display the UI with the initial store state
render()

// 5) Dispatch actions based on UI inputs
document.getElementById('increment').addEventListener('click', function () {
  store.dispatch({ type: 'counter/incremented' })
})
```

No matter what UI layer you're using, Redux works this same way with every UI. The actual implementations are typically a bit more complicated to help optimize performance, but it's the same steps each time.

## useSelecttor

useSelector lets your React components read data from the Redux store. It accepts a single function, wich we call a selector function. A selector is a function that takes the entire Redux store state as its argumets, reads some value from the state, and returns that result.

```js
const selectTodos = state => state.todos
```

As we alredy know when we dispatch an ation the redux state will be updated by the reducer, but our component needs to know that something has changed so that it can re-render.

We know that we can call store.subscribe() to listen for changes to the store, we could try writting the code to subscribe to the store in every component. But, that would quickly get very repetitive and hard to handle.

Fortunately, useSelector automatically subscribes to the Redux store for us! That way, any time an action is dispatched, it will call its selector function again right away. If the value returned by the selector changes from the last time it ran, useSelector will force our component to re-render with the new data. All we have to do is call useSelector() once in our component, and it does the rest of the work for us.

However, there's a very important thing to remember here:

"useSelector compares its results using strict === reference comparisons, so the component will re-render any time the selector result is a new reference! This means that if you create a new reference in your selector and return it, your component could re-render every time an action has been dispatched, even if the data really isn't different."

## Dispatching Actions with useDispatch

The React-Redux useDispatch hook gives us the store's dispatch method as its result. (In fact, the implementation of the hook really is return store.dispatch.)

```js
import React, { useState } from 'react'
import { useDispatch } from 'react-redux'

const Header = () => {
  const [text, setText] = useState('')
  const dispatch = useDispatch()
  //...rest of code
}
```

## Passing the Store with Provider

we have to specifically tell React-Redux what store we want to use in our components. We do this by rendering a <Provider> component around our entire <App>, and passing the Redux store as a prop to <Provider>. After we do this once, every component in the application will be able to access the Redux store if it needs to.

```js
import React from 'react'
import ReactDOM from 'react-dom'
import { Provider } from 'react-redux'

import App from './App'
import store from './store'

ReactDOM.render(
  // Render a `<Provider>` around the entire `<App>`,
  // and pass the Redux store to as a prop
  <React.StrictMode>
    <Provider store={store}>
      <App />
    </Provider>
  </React.StrictMode>,
  document.getElementById('root')
)
```

That covers the key parts of using React-Redux with React:

- Call the useSelector hook to read data in React components
- Call the useDispatch hook to dispatch actions in React components
- Put <Provider store={store}> around your entire <App> component so that other components can talk to the store