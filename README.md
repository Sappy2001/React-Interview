# React Hook: useState

## 1. Introduction

`useState` is a React Hook that allows you to add state to functional components. With `useState`, you can manage state in a simpler, more modular way within function components.

**State** is a data store that is local to a component. It is mutable (can be changed over time) and is used to keep track of component-specific data that influences rendering.

## 2. Syntax

```jsx
const [state, setState] = useState(initialValue);
```

- When a component is first rendered, `useState(initialValue)` initializes state with the provided value.
- When you call `setState(newValue)`, React does not immediately update the state. Instead, it schedules an update and then, during the next render cycle, React computes and updates the new state value and re-renders the component (for performance optimization).

## 3. Example: Counter Component

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  const handleClick = () => {
    console.log('Before update, count:', count); // logs current count-0
    setCount(count + 1);
    console.log('After update, count:', count);  // still logs the old count-0
  };
}
```

## 4. State Updates Are Asynchronous

React may batch multiple state updates together for performance reasons. This means that if you call multiple state update functions in a row, React might process them all in one go before re-rendering.

```jsx
const handleMultipleUpdates = () => {
  setCount(count + 1); // count=0
  setCount(count + 1); // since count doesn't immediately update, here also is 0
};
```

**Result:** Instead of giving `count=2`, it gives `count=1`.

### 4.1 Solution: Using Functional Update

```jsx
const handleMultipleUpdates = () => {
  setCount(prevCount => prevCount + 1); // count=0 sets it to 1
  setCount(prevCount => prevCount + 1); // count=1 sets it to 2
};
```

## 5. Asynchronous Callback in useState

### 5.1 Problem Example: Stale Closure

```jsx
function IncorrectCounter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const intervalId = setInterval(() => {
      setCount(count + 1); // Always sets count to 0, because 'count' here is old value (stale)
    }, 1000);

    return () => clearInterval(intervalId);
  }, []); // Runs only on mount
}
```

### 5.2 Explanation

- When the component first renders, `useState(0)` sets `count` to 0.
- The callback function inside `setInterval` is created in this first render.
- JavaScript closures capture the variables from the lexical scope in which they were created.
- The function captures `count` as `0` and keeps using that value.
- Every second, the interval callback executes but still "remembers" `count` as `0`.
- The state is updated to `1`, but subsequent executions use the old value (`0`), leading to the state being set to `1` over and over.

### 5.3 Solution: Using Functional Update

```jsx
useEffect(() => {
  const intervalId = setInterval(() => {
    setCount((prevCount) => prevCount + 1); // Always work with the most recent state
  }, 1000);

  return () => clearInterval(intervalId);
}, []);
```

## 6. Functional Update in useState

When updating state with React’s `useState` hook, you can pass a function that receives the current state and returns the updated state. This is called a **functional update**.

## 7. Updating a State Without Causing a Re-render

By default, React state (via `useState` or `useReducer`) is designed to trigger a re-render when updated because the rendered UI output depends on that state. However, a **ref object** has a `.current` property that you can update at will. Changes to `.current` do not cause the component to re-render.

### 7.1 Example: Using `useRef` to Store State Without Re-rendering

```jsx
const countRef = useRef(0);

useEffect(() => {
  const intervalId = setInterval(() => {
    countRef.current += 1; // Updates the ref value but does not re-render
    console.log("Count (ref):", countRef.current); // Updated count is logged in the console
  }, 1000);

  return () => clearInterval(intervalId);
}, []);
```

### 7.2 Key Points

- **Persistence:** The ref value persists across renders.
- **No Re-render:** Updating `countRef.current` does not cause the component to re-render.
- **Use Case:** Ideal for storing timers, caching values, or tracking previous state values that do not need to be displayed.

## 8. Stale Closure

A **stale closure** happens when a function “captures” variables (such as state) from an earlier render or closure and then continues to use those outdated values in subsequent calls. This can lead to bugs where the function operates on outdated state, causing inconsistent or unexpected behavior.

---

This document provides a deep understanding of the `useState` hook, its asynchronous behavior, functional updates, and `useRef` for managing state without re-renders.

