---
layout: default
title: Interjú felkészülés
---

# Interjú felkészülés

## React interjúkérdések (medior szintig)

Az alábbi kérdés-válasz gyűjtemény a [GreatFrontEnd 100+ React interjúkérdés cikkéből](https://www.greatfrontend.com/blog/100-react-interview-questions-straight-from-ex-interviewers) származik, a **Freshers** és **Experienced** (kezdő és medior, kb. 0–5 év tapasztalat) szekciókkal, kihagyva a Senior szintet. A promóciós linkeket (kvízre/gyakorlásra mutató belső linkek) eltávolítottam, a hasznos külső referenciákat (react.dev, MDN, stb.) és a kódpéldákat megtartottam.

---

## Freshers

Freshers with 0–2 years of experience: practice explaining component behavior, writing simple interactions, and testing the visible result.

### Components and JSX

#### 1. What is React, and what are its main features?

React is a JavaScript library developed by Facebook for creating user interfaces, particularly in single-page applications. It enables the use of reusable components that <mark>manage their own state</mark>. Key advantages include a component-driven architecture, optimized updates through the virtual DOM, a declarative approach for better readability, and robust community backing.

#### 2. What is JSX and how does it work?

JSX, short for JavaScript XML, is a syntax extension for JavaScript that allows you to write HTML-like code within JavaScript. It makes building React components easier. JSX <mark>gets converted into JavaScript function calls</mark>, often by Babel. For instance, `<div>Hello, world!</div>` is transformed into `React.createElement('div', null, 'Hello, world!')`.

#### 3. Explain the concept of the Virtual DOM in React.

The virtual DOM is a simplified version of the actual DOM used by React. It allows for efficient UI updates by comparing the virtual DOM to the real DOM and making only the necessary changes through a process <mark>known as reconciliation</mark>.

#### 4. What is the difference between Shadow DOM and Virtual DOM?

The Shadow DOM is a web standard that <mark>encapsulates a part of the DOM</mark>, isolating it from the rest of the document. It's used for creating reusable, self-contained components without affecting the global styles or scripts.

The Virtual DOM is an in-memory representation of the actual DOM used to optimize rendering. It compares the current and previous states of the UI, updating only the necessary parts of the DOM, which improves performance.

#### 5. What is the difference Between React Node, React Element, and React Component?

A React Node refers to any unit that can be rendered in React, such as an element, string, number, or `null`. A React Element is an immutable object that defines what should be rendered, typically created using JSX or `React.createElement`. A React Component is either a function or class that returns React Elements, enabling the creation of reusable UI components.

#### 6. What are React Fragments used for?

React Fragments allow you to group multiple elements without <mark>adding extra nodes to the DOM</mark>. They are particularly useful when you need to return multiple elements from a component but don't want to wrap them in a container element. You can utilize shorthand syntax `<>...</>` or `React.Fragment`.

```jsx
return (
  <>
    <ChildComponent1 />
    <ChildComponent2 />
  </>
);
```

#### 7. What are props in React? How are they different from state?

Props (short for properties) are inputs to React components that allow you to pass data from a parent component to a child component. They are immutable and are used to configure a component. In contrast, <mark>state is internal to a component</mark> and can change over time, typically due to user interactions or other events.

#### 8. What is the difference between React's class components and functional components?

Class components are ES6 classes that extend `React.Component` and rely on lifecycle methods `componentDidMount`, `componentDidUpdate`, etc.) and `this.state`. Function components are plain functions that take props as input and return JSX, and use hooks (`useState`, `useEffect`, `useRef`, etc.) for state and side effects. Since hooks landed in React 16.8, function components are <mark>the default for new code</mark>; class components are kept for backward compatibility and are no longer the recommended pattern.

#### 9. What are stateless components?

Stateless components in React are components that <mark>do not manage or hold any internal state</mark>. They simply receive data via props and render UI based on that data. These components are often functional components and are used for presentational purposes.

##### Key points:

- Do not use `this.state`
- Render UI based on `props`
- Focused on displaying information, not managing behavior

```jsx
function StatelessComponent({ message }) {
  return <div>{message}</div>;
}
```

Stateless components are simpler, easier to test, and often more reusable. With the introduction of hooks, React components are mostly written using functions and can contain state via the `useState` hook.

#### 10. What are stateful components?

Stateful components in React are components that manage and hold their own internal state. They can modify their state in response to user interactions or other events and re-render themselves when the state changes.

##### Key points:

- Use `this.state` (in class components) or `useState` (in functional components)
- Can update state using event handlers or lifecycle methods
- Handle logic and data management

```jsx
function StatefulComponent() {
  const [count, setCount] = React.useState(0);

  return (
    <div>
      <p>{count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
```

Stateful components are essential for handling dynamic and interactive UIs.

#### 11. What is the role of `PropTypes` in React?

PropTypes was React's runtime prop type-checker. You declared expected types, and React would warn in the console when a mismatch occurred in development.

```jsx
import PropTypes from 'prop-types';

function MyComponent({ name, age }) {
  return (
    <div>
      {name} is {age} years old
    </div>
  );
}

MyComponent.propTypes = {
  name: PropTypes.string.isRequired,
  age: PropTypes.number.isRequired,
};
```

`PropTypes` is <mark>deprecated as of React 19</mark> and no longer ships from the `react` package. Use TypeScript instead; it catches the same mismatches at compile time and integrates with editor tooling.

#### 12. What are the recommended ways for type checking of React component props?

Use TypeScript. It catches prop mismatches at compile time, integrates with editor tooling (autocomplete, refactors, jump-to-definition), and is the default in most React project templates.

```tsx
type MyComponentProps = {
  name: string;
  age: number;
};

function MyComponent({ name, age }: MyComponentProps) {
  return (
    <div>
      {name} is {age} years old
    </div>
  );
}
```

The older alternative was `PropTypes`, a runtime checker that warned in dev mode when prop types didn't match. It is deprecated as of React 19 and no longer ships from the `react` package. If you're maintaining a codebase that still uses `prop-types`, migrate to TypeScript.

#### 13. Why can `count && <Badge />` unexpectedly render a zero?

The `&&` expression returns its left operand when that operand is falsy. With `count` equal to `0`, the expression returns `0`, and <mark>React renders that number</mark>.

Use a boolean condition, such as `count > 0 && <Badge />`, or an explicit ternary that returns `null`. This matters when a badge disappears but an unexplained zero remains in the UI. See [conditional rendering](https://react.dev/learn/conditional-rendering).

#### 14. What is the difference between `onClick={save}` and `onClick={save()}`?

`onClick={save}` passes a function for React to call when the event happens. `onClick={save()}` calls it during render and passes its return value as the handler, which is <mark>usually a bug</mark> and can cause repeated updates.

To pass an argument, use `onClick={() => save(record.id)}`. Rendering should calculate UI, while event handlers perform work caused by that interaction. See [responding to events](https://react.dev/learn/responding-to-events).

### State, lists, and events

#### 15. What is the purpose of the `key` prop in React?

In React, the `key` prop is used to <mark>uniquely identify elements in a list</mark>, allowing React to optimize rendering by updating and reordering items more efficiently. Without unique keys, React might re-render elements unnecessarily, causing performance problems and potential bugs.

```jsx
{
  items.map((item) => <ListItem key={item.id} value={item.value} />);
}
```

#### 16. What is the consequence of using array indices as keys in React?

Using array indices as `key`s can lead to <mark>performance issues and unexpected behavior</mark>, especially when reordering or deleting items. React relies on keys to identify elements uniquely, and using indices can cause components to be re-rendered unnecessarily or display incorrect data.

#### 17. What is the difference between Controlled and Uncontrolled React components?

In controlled components, form data is managed through the component's state, making it <mark>the definitive source of truth</mark>. Input value changes are handled by event handlers. In uncontrolled components, the form state is managed internally and accessed via refs. Controlled components provide more control and are easier to test, while uncontrolled components are simpler for basic use cases.

Example of a controlled component:

```jsx
function ControlledInput() {
  const [value, setValue] = React.useState('');
  return (
    <input
      type="text"
      value={value}
      onChange={(e) => setValue(e.target.value)}
    />
  );
}
```

Example of an uncontrolled component:

```jsx
function UncontrolledInput() {
  const inputRef = React.useRef();
  return <input type="text" ref={inputRef} />;
}
```

#### 18. How would you lift the state up in a React application, and why is it necessary?

Lifting state up in React involves moving the state from child components to their <mark>nearest common ancestor</mark>. This pattern is used to share state between components that don't have a direct parent-child relationship. By lifting state up, you can avoid prop drilling and simplify the management of shared data.

```jsx
// Lifting state up
const Parent = () => {
  const [counter, setCounter] = useState(0);

  return (
    <div>
      <Child1 counter={counter} />
      <Child2 setCounter={setCounter} />
    </div>
  );
};

const Child1 = ({ counter }) => <h1>{counter}</h1>;
const Child2 = ({ setCounter }) => (
  <button onClick={() => setCounter((prev) => prev + 1)}>Increment</button>
);
```

In this example, the state is managed in the `Parent` component, and both child components access it via props.

#### 19. Why does React recommend against mutating state?

React advises against mutating state as it can lead to unexpected behaviors and bugs. State immutability helps efficiently determine when components need re-rendering; direct mutations may <mark>prevent React from detecting changes</mark>.

#### 20. What does re-rendering mean in React?

In React, re-rendering refers to the process of updating the user interface (UI) in response to changes in the component's state or props. When the state or props of a component change, React re-renders the component to reflect the updated data in the UI.

This involves:

1. Recalculating the JSX returned by the component
2. Comparing the new JSX with the previous one (using the Virtual DOM)
3. Updating the real DOM with only the differences (efficient rendering)
4. Re-rendering ensures that the UI stays in sync with the component's state and props

#### 21. Explain one-way data flow of React

In React, one-way data flow means <mark>data moves from parent to child</mark> components through props.

- **Parent to child**: The parent passes data to the child
- **State updates**: To change data, the child calls a function passed down by the parent

Example:

```jsx
function Parent() {
  const [count, setCount] = React.useState(0);
  return <Child count={count} increment={() => setCount(count + 1)} />;
}

function Child({ count, increment }) {
  return <button onClick={increment}>Count: {count}</button>;
}
```

This ensures data flows in one direction, making the app more predictable.

#### 22. Explain what happens when `setState` is called in React?

When `setState` is called in React:

1. **State update**: It updates the component's state, triggering a re-render of the component
2. **Batching**: React may batch multiple setState calls into a single update for performance optimization
3. **Re-render**: React re-renders the component (and its child components if needed) with the new state
4. **Asynchronous**: State updates may be asynchronous, meaning React doesn't immediately apply the state change; it schedules it for later to optimize performance

Example:

```jsx
function Counter() {
  const [count, setCount] = React.useState(0);

  const increment = () => {
    setCount(count + 1); // Calls setState to update state
  };

  return <button onClick={increment}>Count: {count}</button>;
}
```

In this example, calling `setState` (via `setCount`) triggers a re-render with the updated `count`.

#### 23. Explain prop drilling

Prop drilling is when you pass data from a parent component to a deeply nested child component through props, even if intermediate components don't use it.

Example:

```jsx
function Grandparent() {
  const data = 'Hello from Grandparent';
  return <Parent data={data} />;
}

function Parent({ data }) {
  return <Child data={data} />;
}

function Child({ data }) {
  return <p>{data}</p>;
}
```

In this example, `data` is passed through multiple components, even though only the `Child` component uses it. Prop drilling is <mark>acceptable for small applications</mark> where the component hierarchy is shallow. When global state is needed to be accessed in deeper levels of the app, using context and/or external state managers might be better.

#### 24. Discuss synthetic events in React

Synthetic events in React are a wrapper around native DOM events that ensure <mark>consistent behavior across browsers</mark>. They normalize the way events are handled, providing a unified API for React applications.

These events are wrapped in the `SyntheticEvent` object and expose the usual methods like `preventDefault()` and `stopPropagation()`. Since React 17, the root event listener is attached to the React root container (not `document`), which makes nested React trees work correctly together.

Example:

```jsx
function MyComponent() {
  const handleClick = (event) => {
    event.preventDefault();
    console.log('Button clicked');
  };

  return <button onClick={handleClick}>Click me</button>;
}
```

Older sources mention event pooling, where React reused the event object after the handler ran, which made the event unusable in async code. Event pooling was removed in React 17, so you can read or pass the event object asynchronously without calling `event.persist()`.

#### 25. What happens when you call a state setter three times in one click handler?

Starting at zero, three calls to `setCount(count + 1)` all request the value `1` because the handler <mark>reads one render's snapshot</mark>. Three calls to `setCount(c => c + 1)` queue three updater functions and produce `3`.

Do not use `await setCount(...)` to wait for rendering: the setter does not return a completion Promise. See [updater queues](https://react.dev/learn/queueing-a-series-of-state-updates).

#### 26. How would you update one field in a nested state object?

Copy each object along the changed path while retaining unchanged branches:

```jsx
setProfile((previous) => ({
  ...previous,
  address: { ...previous.address, city: nextCity },
}));
```

This assumes `profile.address` exists. Mutating `previous.address.city` after only copying `previous` <mark>still mutates existing state</mark>. For deeply nested models, flatten the structure or use a library that produces immutable updates. See [nested state updates](https://react.dev/learn/updating-objects-in-state#updating-a-nested-object).

#### 27. Why does `useState(props.name)` not follow later prop changes?

The argument initializes state; React <mark>does not overwrite that state</mark> every time the prop changes. This is useful for an editable draft but surprising if the UI is meant to always reflect the parent.

If no independent draft is needed, read the prop directly. If the parent should own edits, make the value controlled. If a different record starts a new editing session, a record key can reset it. See [avoiding mirrored props](https://react.dev/learn/choosing-the-state-structure#dont-mirror-props-in-state).

#### 28. How do you fix an uncontrolled-to-controlled input warning?

An input that starts with `value={undefined}` and later receives a string <mark>switches from uncontrolled to controlled</mark>. Initialize controlled text state to `''`, or use `value={name ?? ''}` when the data may be missing.

For checkboxes, use a boolean `checked` value. Keep an input controlled or uncontrolled for its entire lifetime, and update controlled state synchronously from `onChange`. See [input troubleshooting](https://react.dev/reference/react-dom/components/input#im-getting-an-error-a-component-is-changing-an-uncontrolled-input-to-be-controlled).

### Hooks and composition basics

#### 29. What are the benefits of using hooks in React?

Hooks enable the use of state and other React features in functional components, replacing the need for class components. They streamline code by reducing the reliance on lifecycle methods, enhance readability, and facilitate the reuse of stateful logic across components.

Popular hooks like `useState` and `useEffect` are used for managing state and side effects.

#### 30. What are the rules of React hooks?

React hooks should be called at the <mark>top level of a function</mark>, not inside loops, conditions, or nested functions. They must only be used within React function components or custom hooks. These guidelines ensure proper state management and lifecycle behavior.

#### 31. What does the dependency array of `useEffect` affect?

The dependency array of `useEffect` controls when the effect re-runs:

- If it's empty, the effect runs only once after the initial render.
- If it contains variables, the effect re-runs whenever any of those variables change.
- If omitted, the effect runs after every render.

#### 32. What is the `useRef` hook in React and when should it be used?

The `useRef` hook creates a mutable object that <mark>persists through renders</mark>, allowing direct access to DOM elements, storing mutable values without causing re-renders, and maintaining references to values.

For instance, `useRef` can be utilized to focus on an input element:

```jsx
import React, { useRef, useEffect } from 'react';

function TextInputWithFocusButton() {
  const inputEl = useRef(null);
  useEffect(() => {
    inputEl.current.focus();
  }, []);
  return <input ref={inputEl} type="text" />;
}
```

#### 33. What is the `useId` hook in React and when should it be used?

The `useId` hook generates unique IDs for elements within a component, which is crucial for accessibility by dynamically creating ids that can be used for linking form inputs and labels. It guarantees <mark>unique IDs across the application</mark> even if the component renders multiple times.

```jsx
import { useId } from 'react';

function MyComponent() {
  const id = useId();

  return (
    <div>
      <label htmlFor={id}>Name:</label>
      <input id={id} type="text" />
    </div>
  );
}
```

#### 34. Can you explain how to create and use custom hooks in React?

To create and use custom hooks in React:

1. Create a function that starts with use and uses built-in hooks like `useState` or `useEffect`
2. Return the values or functions you want to share.

Example:

```jsx
function useForm(initialState) {
  const [formData, setFormData] = useState(initialState);
  const handleChange = (e) =>
    setFormData({ ...formData, [e.target.name]: e.target.value });
  return [formData, handleChange];
}
```

Use the Hook:

```jsx
function MyForm() {
  const [formData, handleChange] = useForm({ name: '', email: '' });
  return <input name="name" value={formData.name} onChange={handleChange} />;
}
```

Custom hooks let you reuse logic across components, keeping your code clean.

#### 35. Explain the composition pattern in React.

The composition pattern in React involves building components by combining smaller, reusable ones instead of using inheritance. This encourages creating complex UIs by <mark>passing components as children or props</mark>.

```jsx
function WelcomeDialog() {
  return (
    <Dialog>
      <h1>Welcome</h1>
      <p>Thank you for visiting our spacecraft!</p>
    </Dialog>
  );
}

function Dialog(props) {
  return <div className="dialog">{props.children}</div>;
}
```

#### 36. Do two components calling the same custom Hook share state?

If `useCounter()` owns state through `useState`, calling it in two components <mark>creates independent state for each invocation</mark>. Sharing a function shares logic, not the state stored by React for that call.

To share a value, lift its state to a common owner, distribute it through context, or subscribe both components to an external store. A custom Hook can wrap those mechanisms, but the shared ownership must be explicit. See [custom Hook state isolation](https://react.dev/learn/reusing-logic-with-custom-hooks#custom-hooks-let-you-share-stateful-logic-not-state-itself).

### Routing and testing basics

#### 37. What is a React Router?

React Router is a popular routing library for React applications that enables navigation between different components based on the URL. It provides declarative routing, allowing you to define routes and their corresponding components in a straightforward manner.

#### 38. How does React Router work, and how do you implement dynamic routing?

React Router maps URL paths to components, enabling navigation in single-page apps. Dynamic routing allows you to use URL parameters to render components based on dynamic values.

```jsx
import { BrowserRouter, Routes, Route, useParams } from 'react-router-dom';

function UserPage() {
  const { id } = useParams(); // Access dynamic parameter
  return <h1>User ID: {id}</h1>;
}

export default function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/user/:id" element={<UserPage />} /> {/* Dynamic path */}
      </Routes>
    </BrowserRouter>
  );
}
```

**Key features**:

- Dynamic Segments: `:id` captures dynamic data from the URL.
- `useParams` Hook: Accesses these dynamic values for rendering.

#### 39. How do you navigate programmatically in React Router?

In React Router v6, you can navigate programmatically by using the `useNavigate` hook. First, import `useNavigate` from `react-router-dom` and call it to get the navigate function. Then, you can use `navigate('/new-page')` to navigate to a different route.

For example:

```jsx
import { useNavigate } from 'react-router-dom';

function MyComponent() {
  const navigate = useNavigate();
  const goToPage = () => navigate('/new-page');
  return <button onClick={goToPage}>Go to New Page</button>;
}
```

In React Router v5, the `useHistory` hook provides access to the history object, which you can use to push a new route. For example, `history.push('/new-page')` will navigate to the specified route.

For example:

```jsx
import { useHistory } from 'react-router-dom';

function MyComponent() {
  const history = useHistory();
  const goToPage = () => history.push('/new-page');
  return <button onClick={goToPage}>Go to New Page</button>;
}
```

Both methods allow you to navigate programmatically in React Router.

#### 40. How do you manage the active route state in a multi-page React application?

Use the useLocation hook to get the current route, and conditionally apply styles for the active state.

Example:

```jsx
import { useLocation } from 'react-router-dom';

function NavBar() {
  const location = useLocation();
  return (
    <nav>
      <ul>
        <li className={location.pathname === '/home' ? 'active' : ''}>Home</li>
        <li className={location.pathname === '/about' ? 'active' : ''}>
          About
        </li>
      </ul>
    </nav>
  );
}
```

#### 41. How to get query parameters in React Router?

In React Router v6, you can use the `useSearchParams` hook to access query parameters from the URL.

Example:

```jsx
import { useSearchParams } from 'react-router-dom';

function MyComponent() {
  const [searchParams] = useSearchParams();
  const queryParam = searchParams.get('paramName');
  return <div>Query Param: {queryParam}</div>;
}
```

This hook allows you to retrieve and manipulate query parameters in React Router v6.

#### 42. How do you pass props to a route component in React Router?

In React Router v6, you can pass props to a route component using the `element` prop in the `<Route>` component.

Example:

```jsx
import { Routes, Route } from 'react-router-dom';

function MyComponent({ propValue }) {
  return <div>Prop Value: {propValue}</div>;
}

function App() {
  return (
    <Routes>
      <Route path="/my-route" element={<MyComponent propValue="Hello" />} />
    </Routes>
  );
}
```

In this example, the `propValue` prop is passed to the `MyComponent` component when rendering the `/my-route` route.

#### 43. How do you test React applications?

Testing React applications can be done using Jest and React Testing Library. Jest serves as the testing framework while React Testing Library provides utilities for testing components similarly to user interactions.

#### 44. What is Jest and how is it used for testing React applications?

Jest is a JavaScript testing framework that provides a test runner, assertion library, and mocking support. It's commonly used for testing React applications due to its simplicity and integration with tools like React Testing Library.

#### 45. What is React Testing Library and how is it used for testing React components?

React Testing Library is a testing utility for React that helps test components in a way that resembles how users interact with the application. It provides functions to render components, interact with them, and assert on the rendered output.

#### 46. How do you test React components using React Testing Library?

To test React components using React Testing Library, you can:

1. Render the component using `render`.
2. Interact with the component (e.g., clicking buttons, entering text).
3. Assert on the rendered output using queries like `getByText`, `queryByRole`, etc.

Example:

```jsx
import { render, screen, fireEvent } from '@testing-library/react';
import MyComponent from './MyComponent';

test('renders component', () => {
  render(<MyComponent />);
  const button = screen.getByRole('button');
  fireEvent.click(button);
  expect(screen.getByText('Clicked!')).toBeInTheDocument();
});
```

In this example, the test renders `MyComponent`, clicks a button, and asserts that the text 'Clicked!' is present.

#### 47. How do `getBy`, `queryBy`, and `findBy` differ in component tests?

`getBy` returns an existing match and throws if none is found. `queryBy` <mark>returns `null` when no match exists</mark>, useful for asserting absence. `findBy` returns a Promise that retries until a match appears or times out.

The singular variants require exactly one match: synchronous queries throw on multiple matches, while `findBy` retries and rejects if that condition still fails at the timeout. Use the `AllBy` versions when multiple elements are expected. Choose the query based on whether the UI should exist now, be absent, or appear later. See [query types](https://testing-library.com/docs/queries/about/#types-of-queries).

## Experienced

Experienced developers with 2–5 years of experience: practice diagnosing bugs, coordinating async UI, and choosing patterns for features you can own end to end.

### Effects, refs, and state patterns

#### 48. What is the difference between `useEffect` and `useLayoutEffect` in React?

`useEffect` and `useLayoutEffect` both handle side effects in React functional components but differ in when they run:

- `useEffect` runs asynchronously after the DOM has rendered, making it suitable for tasks like data fetching or subscriptions.
- `useLayoutEffect` runs synchronously after DOM updates but <mark>before the browser paints</mark>, ideal for tasks like measuring DOM elements or aligning the UI with the DOM. Example:

```jsx
import React, { useEffect, useLayoutEffect, useRef } from 'react';

function Example() {
  const ref = useRef();

  useEffect(() => {
    console.log('useEffect: Runs after DOM paint');
  });

  useLayoutEffect(() => {
    console.log('useLayoutEffect: Runs before DOM paint');
    console.log('Element width:', ref.current.offsetWidth);
  });

  return <div ref={ref}>Hello</div>;
}
```

#### 49. What is the purpose of callback function argument format of `setState()` in React class components and when should it be used?

This applies to class components, which are no longer the recommended pattern. The function-component equivalent (the updater form of `useState`) is shown at the end.

The callback (updater) form of `setState()` ensures state updates are based on the most current state and props. This matters when the new state depends on the previous state, because React may batch multiple updates and the `this.state` you'd read directly <mark>could be stale</mark>.

```jsx
this.setState((prevState, props) => ({
  counter: prevState.counter + props.increment,
}));
```

The function-component equivalent uses the updater form of `useState`:

```jsx
const [counter, setCounter] = useState(0);
setCounter((prev) => prev + props.increment);
```

#### 50. Explain the React component lifecycle methods in class components.

Class lifecycle methods only apply to class components, which are no longer the recommended pattern. The function-component equivalents (using `useEffect`) are shown at the end.

React class components have lifecycle methods for different phases:

##### Mounting:

- `constructor`: Initializes state or binds methods
- `componentDidMount`: Runs after the component mounts, useful for API calls or subscriptions

```jsx
componentDidMount() {
  console.log('Component mounted');
}
```

##### Updating:

- `shouldComponentUpdate`: Determines if the component should re-render
- `componentDidUpdate`: Runs after updates, useful for side effects

##### Unmounting:

- `componentWillUnmount`: Cleans up (e.g., removing event listeners).

```jsx
componentWillUnmount() {
  console.log('Component will unmount');
}
```

In function components, all of the above are expressed with `useEffect`:

```jsx
useEffect(
  () => {
    // componentDidMount + componentDidUpdate
    console.log('Mounted or updated');
    return () => {
      // componentWillUnmount
      console.log('Will unmount');
    };
  },
  [
    /* deps */
  ],
);
```

#### 51. What are Pure Components?

Pure Components in React are components that only re-render when their props or state change. They use <mark>shallow comparison</mark> to check if the props or state have changed, preventing unnecessary re-renders and improving performance.

- Class components can extend `React.PureComponent` to become pure
- Functional components can use `React.memo` for the same effect

```jsx
const PureFunctionalExample = React.memo(function ({ value }) {
  return <div>{value}</div>;
});
```

With the React Compiler, manual memoization with `React.memo`, `useMemo`, and `useCallback` is rarely needed; the compiler inserts equivalent memoization automatically.

#### 52. What is the `useCallback` hook in React and when should it be used?

The `useCallback` hook memoizes functions to <mark>prevent their recreation on every render</mark>. This is especially beneficial when passing callbacks to optimized child components that depend on reference equality to avoid unnecessary renders. Use it when a function is passed as a prop to a memoized child component.

```jsx
const memoizedCallback = useCallback(() => {
  doSomething(a, b);
}, [a, b]);
```

With the React Compiler enabled, you rarely need `useCallback` manually; the compiler inserts equivalent memoization automatically.

#### 53. What is the `useMemo` hook in React and when should it be used?

The `useMemo` hook memoizes costly calculations, <mark>recomputing them only when dependencies change</mark>. This enhances performance by avoiding unnecessary recalculations. It should be used for computationally intensive functions that don't need to run on every render.

```jsx
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
```

With the React Compiler enabled, you rarely need `useMemo` manually; the compiler memoizes derived values automatically.

#### 54. What is the `useReducer` hook in React and when should it be used?

The `useReducer` hook manages complex state logic in functional components, serving as an alternative to `useState`. It's ideal when state has multiple fields (and there are constraints around how they should be mutated), or when the next state <mark>relies on the previous one</mark>.

The `useReducer` hook accepts a reducer function + an initial state. The `reducer` function is passed the current `state` and `action` and returns a new state.

```jsx
const [state, dispatch] = useReducer(reducer, initialState);
```

#### 55. What is `forwardRef()` in React used for?

Before React 19, function components didn't accept `ref` as a regular prop, so `forwardRef()` was used to pass a `ref` through to a child DOM element.

```jsx
// Pre-React 19
import React, { forwardRef } from 'react';

const MyComponent = forwardRef((props, ref) => <input ref={ref} {...props} />);
```

In React 19, <mark>`ref` is a regular prop</mark> on function components and `forwardRef` is deprecated. Destructure it from props:

```jsx
function MyComponent({ ref, ...props }) {
  return <input ref={ref} {...props} />;
}
```

#### 56. When should you use a class component over a function component?

Default to function components. Class components are legacy: new APIs like Suspense data fetching, the `use` hook, Actions, Server Components, and the React Compiler are designed for function components only. The one remaining reason to write a class today is <mark>implementing an error boundary</mark>, which still requires `static getDerivedStateFromError` / `componentDidCatch`.

#### 57. What is the difference between `createElement` and `cloneElement`?

The difference between createElement and cloneElement in React is as follows:

##### `createElement`:

- Used to create a new React element.
- It takes the type of the element (e.g., 'div', a React component), props, and children, and returns a new React element.
- Commonly used internally by JSX or when dynamically creating elements. Example:

```jsx
React.createElement('div', { className: 'container' }, 'Hello World');
```

##### `cloneElement`:

- Used to clone an existing React element and optionally modify its props.
- It allows you to clone a React element and pass new props or override the existing ones, keeping the original element's children and state.
- Useful when you want to manipulate an element without recreating it. Example:

```jsx
const element = <button className="btn">Click Me</button>;
const clonedElement = React.cloneElement(element, { className: 'btn-primary' });
```

#### 58. What is React strict mode and what are its benefits?

React Strict Mode is a development feature in React that activates extra checks and warnings to help identify potential issues in your app.

- **Detects unsafe lifecycles**: Warns about deprecated lifecycle methods
- **Identifies side effects**: Highlights components with side effects in render methods
- **Warns about unexpected state changes**: Catches unexpected state mutations
- **Enforces best practices**: Flags potential problems, encouraging modern practices

```jsx
<React.StrictMode>
  <App />
</React.StrictMode>
```

Wrapping components in `<React.StrictMode>` activates these development checks without affecting production builds.

#### 59. What are some React anti-patterns?

React anti-patterns are practices that can lead to inefficient or hard-to-maintain code. Common examples include:

- Directly mutating state instead of using the state setter
- Using `useEffect` to derive state from props (compute it during render instead)
- Putting data into state that you can compute from other state or props
- Not using keys in lists, or using the array index as a key for reorderable lists
- Effects with missing or stale dependencies
- Deeply nested state; prefer flat shapes with `useReducer` or a state library
- Reading or writing refs during render (do it in effects or event handlers)
- Using `useState` for values that don't drive rendering (use `useRef` instead)
- Calling hooks conditionally or inside loops (breaks the Rules of Hooks)

The older class-component anti-patterns (using `componentWillMount` for data fetching or relying on `componentWillReceiveProps`) refer to APIs that were renamed to `UNSAFE_*` and no longer apply to function-component code.

#### 60. What are higher-order components in React?

Higher-order components (HOCs) are functions that take a component and return a new one with added props or behavior, facilitating logic reuse across components.

```jsx
const withExtraProps = (WrappedComponent) => {
  return (props) => <WrappedComponent {...props} extraProp="value" />;
};

const EnhancedComponent = withExtraProps(MyComponent);
```

HOCs were the <mark>dominant pattern for cross-cutting logic</mark> (auth, data fetching, theming) before hooks. Custom hooks cover almost all of those use cases now without the wrapper-component nesting. HOCs still appear in older codebases and some libraries (e.g., `connect` from `react-redux`), but new code should prefer a custom hook.

#### 61. Explain the presentational vs container component pattern in React

The presentational vs container pattern split components into two roles: **presentational** components handled rendering (markup, styling) and received data via props, while **container** components handled state, data fetching, and behavior, then passed data down.

```jsx
// Container: handles state/data
function UserListContainer() {
  const [users, setUsers] = useState([]);
  useEffect(() => {
    fetchUsers().then(setUsers);
  }, []);
  return <UserList users={users} />;
}

// Presentational: pure rendering
function UserList({ users }) {
  return (
    <ul>
      {users.map((u) => (
        <li key={u.id}>{u.name}</li>
      ))}
    </ul>
  );
}
```

This pattern was popular before hooks; its original author (Dan Abramov) has since said it's <mark>no longer worth following</mark> as a hard rule. With hooks, the same separation is usually expressed by extracting a custom hook (e.g., `useUsers()`) rather than a wrapper component. New code typically blends the two roles into a single component plus a custom hook.

#### 62. What are render props in React?

Render props in React allow code sharing between components through a prop that is a function. This function returns a React element, enabling data to be passed to child components.

```jsx
function DataFetcher({ url, render }) {
  const [data, setData] = useState(null);
  useEffect(() => {
    fetch(url)
      .then((res) => res.json())
      .then(setData);
  }, [url]);
  return render(data);
}

// Usage
<DataFetcher
  url="/api/data"
  render={(data) => <div>{data ? data.name : 'Loading...'}</div>}
/>;
```

Like HOCs, render props were a <mark>pre-hooks solution for sharing stateful logic</mark>. Most of those use cases are now solved with a custom hook (`const data = useFetch(url)`), which composes more naturally and avoids the render-prop pyramid. Render props are still useful in narrow cases where the consumer needs to control what to render based on parent-managed state (e.g., headless component libraries).

#### 63. How do you re-render the view when the browser is resized?

To re-render the view on browser resize, use the `useEffect` hook to listen for the resize event and update state.

Example:

```jsx
import React, { useState, useEffect } from 'react';

function ResizeComponent() {
  const [windowWidth, setWindowWidth] = useState(window.innerWidth);

  useEffect(() => {
    const handleResize = () => setWindowWidth(window.innerWidth);
    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);

  return <div>Window width: {windowWidth}px</div>;
}

export default ResizeComponent;
```

This updates the state and re-renders the component whenever the window is resized.

#### 64. How can a key reset a form when the selected record changes?

Render `<Editor key={record.id} record={record} />` when switching records should discard the editor's local draft. A changed key makes React create a new component instance rather than preserve the old state.

Use this deliberately: remounting also <mark>resets focus, refs, and Effects</mark>. If drafts should survive switching records, store them by record ID outside the remounted editor. See [resetting state with a key](https://react.dev/learn/preserving-and-resetting-state#resetting-state-with-a-key).

#### 65. When should you calculate a value during render instead of using an Effect?

If a value is derived entirely from current props or state, compute it during render: `const fullName = firstName + ' ' + lastName`. Setting it from an Effect adds another render and <mark>permits inconsistent intermediate state</mark>.

Use an Effect when synchronizing with something outside React, such as a connection. Memoize a derived calculation only when its cost warrants it. See [removing unnecessary Effects](https://react.dev/learn/you-might-not-need-an-effect#updating-state-based-on-props-or-state).

#### 66. What should happen to a subscription when an Effect dependency changes?

If an Effect connects to `roomId`, changing that ID should disconnect the old room before connecting the new one. Return cleanup from setup and include the reactive inputs the connection uses.

Test the sequence “connect A, disconnect A, connect B, disconnect B on unmount.” <mark>Cleanup must mirror setup</mark>, including when development checks exercise it more than once. See [Effect lifecycle](https://react.dev/learn/lifecycle-of-reactive-effects).

#### 67. How would you prevent an old request from overwriting newer search results?

Give each Effect execution a cleanup guard or request identity and <mark>ignore obsolete responses</mark>. Abort supported requests too, but do not assume abort reverses server work.

```jsx
useEffect(() => {
  let active = true;
  const controller = new AbortController();
  setResult({ status: "loading" });

  async function load() {
    try {
      const response = await fetch(
        `/api/search?q=${encodeURIComponent(query)}`,
        {
          signal: controller.signal,
        },
      );
      if (!response.ok) throw new Error(`HTTP ${response.status}`);
      const data = await response.json();
      if (active) setResult({ status: "success", data });
    } catch (error) {
      if (active) setResult({ status: "error", error });
    }
  }
  load();
  return () => {
    active = false;
    controller.abort();
  };
}, [query]);
```

This excerpt belongs inside a component with `query` and `result` state. See [request cancellation](https://developer.mozilla.org/en-US/docs/Web/API/AbortController).

#### 68. Why does an interval sometimes keep reading the initial state?

The interval callback <mark>closes over values from the render</mark> that created it. An Effect with `[]` will not install a new callback when those values change.

For a counter, use `setCount(c => c + 1)` inside the interval and clear it in cleanup. If the subscription truly depends on another value, include that dependency. Do not suppress the linter to hide stale data. See [removing Effect dependencies](https://react.dev/learn/removing-effect-dependencies).

#### 69. When would you use `useImperativeHandle`?

Use it to <mark>expose a small imperative API</mark> through a ref, such as `focus()` or `scrollToItem()`, instead of exposing every detail of an internal DOM node. Include reactive values used by the handle in its dependencies.

Prefer props for declarative behavior such as whether a panel is open. In React 19, a function component can receive the ref as a prop; older versions require `forwardRef`. See [useImperativeHandle](https://react.dev/reference/react/useImperativeHandle).

#### 70. How can TypeScript prevent impossible component prop combinations?

Use a discriminated union so each mode has its own required props:

```tsx
type FieldProps =
  | { mode: "editable"; value: string; onChange: (value: string) => void }
  | { mode: "readonly"; value: string; onChange?: never };
```

After checking `props.mode`, <mark>TypeScript narrows the allowed properties</mark>. This communicates the component contract better than making every property optional. Runtime data still requires validation. See [TypeScript discriminated unions](https://www.typescriptlang.org/docs/handbook/2/narrowing.html#discriminated-unions).

### Rendering, data loading, and React 19 forms

#### 71. How does virtual DOM in React work? What are its benefits and downsides?

The virtual DOM in React is an in-memory representation of the real DOM. When state or props change, React creates a new virtual DOM tree, compares it to the previous one using a diffing algorithm, and efficiently updates only the parts of the real DOM that changed.

- **Benefits**: It improves performance by reducing costly direct DOM manipulations and makes UI updates declarative and predictable.
- **Downsides**: There's some overhead from diffing and extra memory usage, and in very dynamic UIs, it may not always outperform manual optimizations.

#### 72. What is reconciliation?

Reconciliation is the process by which React updates the DOM to match the virtual DOM efficiently. It involves comparing the new virtual DOM tree with the previous one and determining the minimum number of changes required to update the actual DOM. This process ensures optimal performance by avoiding unnecessary re-renders.

#### 73. What are error boundaries in React for?

Error boundaries catch JavaScript errors in their child components, log them, and display fallback UI instead of crashing the application. They utilize `componentDidCatch` and `static getDerivedStateFromError` methods but <mark>do not catch errors in event handlers</mark> or asynchronous code.

#### 74. What is React Suspense?

React Suspense allows handling asynchronous operations more elegantly within components. It provides fallback content while waiting for resources like data or code to load. You can use it alongside `React.lazy()` for code splitting.

```jsx
const LazyComponent = React.lazy(() => import('./LazyComponent'));

function MyComponent() {
  return (
    <React.Suspense fallback={<div>Loading...</div>}>
      <LazyComponent />
    </React.Suspense>
  );
}
```

#### 75. What is code splitting in a React application?

Code splitting enhances performance by dividing code into smaller chunks loaded on demand, thereby reducing initial load times. This can be achieved through dynamic import() statements or using React's React.lazy and Suspense.

```jsx
// Using React.lazy and Suspense
const LazyComponent = React.lazy(() => import('./LazyComponent'));

function App() {
  return (
    <React.Suspense fallback={<div>Loading...</div>}>
      <LazyComponent />
    </React.Suspense>
  );
}
```

#### 76. Describe lazy loading in React

Lazy loading in React is a technique where components are loaded only when they are needed, rather than at the initial page load. This helps reduce the initial load time and improve performance by splitting the code into smaller chunks.

Example:

```jsx
import React, { Suspense, lazy } from 'react';

const LazyComponent = lazy(() => import('./LazyComponent'));

function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <LazyComponent />
    </Suspense>
  );
}
```

In this example, `LazyComponent` is loaded only when it's rendered, and while loading, a fallback UI (Loading...) is displayed.

#### 77. How do you handle asynchronous data loading in React applications?

Asynchronous data loading uses `useEffect` alongside `useState` hooks; fetching data inside `useEffect` updates state with fetched results ensuring re-renders occur with new data.

```jsx
import React, { useState, useEffect } from 'react';

const FetchAndDisplayData = () => {
  const [info, updateInfo] = useState(null);
  const [isLoading, toggleLoading] = useState(true);

  useEffect(() => {
    const retrieveData = async () => {
      try {
        const res = await fetch('https://api.example.com/data');
        const data = await res.json();
        updateInfo(data);
      } catch (err) {
        console.error('Error fetching data:', err);
      } finally {
        toggleLoading(false);
      }
    };

    retrieveData();
  }, []);

  return (
    <div>
      {isLoading ? (
        <p>Fetching data, please wait...</p>
      ) : (
        <pre>{JSON.stringify(info, null, 2)}</pre>
      )}
    </div>
  );
};

export default FetchAndDisplayData;
```

#### 78. What are some common pitfalls when doing data fetching in React?

Common pitfalls in data fetching with React include failing to handle loading and error states, <mark>neglecting to clean up subscriptions</mark> which can cause memory leaks, and improperly using lifecycle methods or hooks. Always ensure proper handling of these states, clean up after components, and utilize `useEffect` for side effects in functional components.

#### 79. What's new in React 19?

React 19 adds:

- **Actions**: functions that wrap async work and produce pending/error/data state via new hooks.
- **The `use` hook**: reads promises and context during render.
- Stable **React Server Components** and **Server Actions**.
- Native support for `<form action={fn}>`.
- `ref` as a regular prop on function components (no more `forwardRef`).
- Hoisting of `<title>`, `<meta>`, and stylesheets out of JSX.
- The **React Compiler**: an opt-in build-time optimizer that auto-memoizes.

Together, these move data mutations and async UI state into React itself, instead of leaving them as patterns each app reinvents.

#### 80. What are Actions in React 19?

By convention, an Action is an async function passed to a React API that runs it inside a transition: `useActionState`, `startTransition` (from `useTransition`), or a `<form action={...}>` prop. React tracks pending state, surfaces errors, and applies updates inside a transition so the UI stays responsive. This <mark>removes the usual boilerplate</mark> of toggling a loading flag, wrapping in try/catch, and managing error and data state separately.

```jsx
import { useActionState } from 'react';

async function updateName(prevState, formData) {
  const name = formData.get('name');
  const error = await saveName(name);
  if (error) return { error };
  return { name };
}

function NameForm() {
  const [state, dispatchAction, isPending] = useActionState(updateName, {
    name: '',
  });
  return (
    <form action={dispatchAction}>
      <input name="name" defaultValue={state.name} />
      <button disabled={isPending}>Save</button>
      {state.error && <p>{state.error}</p>}
    </form>
  );
}
```

#### 81. What does the `useActionState` hook do?

`useActionState` takes an action function and an initial state, and returns `[state, dispatchAction, isPending]`. Calling `dispatchAction` (usually by passing it to `<form action>`) runs the action, marks `isPending` true, and replaces the state with the action's return value when it resolves. One hook covers what you'd otherwise write as <mark>three separate `useState` calls</mark> for data, loading, and error.

#### 82. What does `useOptimistic` do?

`useOptimistic` renders an optimistic version of state immediately while an action is in flight, then <mark>automatically reverts to the real state</mark> when the action settles. Useful for chat messages, likes, list reordering, or anywhere the network round-trip would feel laggy.

```jsx
import { useOptimistic } from 'react';

function MessageList({ messages, sendMessage }) {
  const [optimisticMessages, addOptimistic] = useOptimistic(
    messages,
    (state, newMessage) => [...state, { text: newMessage, sending: true }],
  );

  async function handleSend(formData) {
    const text = formData.get('text');
    addOptimistic(text);
    await sendMessage(text);
  }

  return (
    <>
      {optimisticMessages.map((m, i) => (
        <p key={i} style={{ opacity: m.sending ? 0.5 : 1 }}>
          {m.text}
        </p>
      ))}
      <form action={handleSend}>
        <input name="text" />
      </form>
    </>
  );
}
```

#### 83. What is the `use` hook and how is it different from `useEffect` + fetch?

`use` reads the value of a Promise or Context during render. When given a Promise, it suspends the component until the promise resolves (handled by the nearest `<Suspense>` boundary) and then returns the resolved value. Unlike `useEffect`, `use` <mark>can be called conditionally and inside loops</mark>, and the resolved data is available synchronously after suspension, so there's no `loading` state to thread through the tree.

```jsx
import { use, Suspense } from 'react';

function Profile({ userPromise }) {
  const user = use(userPromise); // suspends until resolved
  return <h1>{user.name}</h1>;
}

// Server Component: render runs once per request, so the promise is stable.
// In a Client Component, create the promise outside render (or via `cache()`)
// to avoid making a new one on every re-render.
async function Page() {
  const userPromise = fetchUser();
  return (
    <Suspense fallback={<p>Loading...</p>}>
      <Profile userPromise={userPromise} />
    </Suspense>
  );
}
```

#### 84. How does the new form `action` prop work in React 19?

React 19 lets you pass a function directly to `<form action>` (and `<button formAction>`). React calls the function with a `FormData` argument when the form is submitted, runs it inside a transition, and <mark>resets uncontrolled inputs on success</mark>. Combine it with `useActionState` or `useFormStatus` for pending state and error handling without manual `onSubmit` plumbing.

```jsx
import { useFormStatus } from 'react-dom';

function SubmitButton() {
  const { pending } = useFormStatus();
  return <button disabled={pending}>{pending ? 'Saving...' : 'Save'}</button>;
}

function ProfileForm() {
  async function save(formData) {
    await updateProfile(Object.fromEntries(formData));
  }
  return (
    <form action={save}>
      <input name="name" />
      <SubmitButton />
    </form>
  );
}
```

#### 85. Why might `useFormStatus` always report `pending: false`?

The Hook <mark>reads the status of a parent form</mark>. Calling it in the same component that returns the form does not subscribe to that form; move it into a child rendered inside the form.

```jsx
import { useFormStatus } from "react-dom";

function SubmitButton() {
  const { pending } = useFormStatus();
  return (
    <button type="submit" disabled={pending}>
      {pending ? "Saving…" : "Save"}
    </button>
  );
}
```

Render this beneath `<form action={save}>`. See [useFormStatus](https://react.dev/reference/react-dom/hooks/useFormStatus).

#### 86. How do deferred rendering and debouncing differ for search?

A deferred value lets React prioritize urgent UI while preparing results for a newer value. It has no fixed delay and does not by itself reduce the number of network requests.

Debouncing waits for a quiet interval before starting work, which can reduce requests but <mark>adds intentional latency</mark>. Use request cancellation or stale-response protection with either approach; keeping typing responsive does not guarantee that the latest response wins. See [deferred values and requests](https://react.dev/reference/react/useDeferredValue#how-does-deferring-a-value-work-under-the-hood).

### Routing and navigation behavior

#### 87. How do you handle nested routes and route parameters in React Router?

Nested routes allow you to create hierarchies of components, and `useParams` helps access dynamic route parameters.

**Key techniques**:

- `<Outlet>`: Renders child routes within a parent layout
- `useParams`: Retrieves route parameters for dynamic routing

```jsx
import {
  BrowserRouter,
  Routes,
  Route,
  Outlet,
  useParams,
} from 'react-router-dom';

function UserProfile() {
  const { userId } = useParams();
  return <h2>User ID: {userId}</h2>;
}

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="user/:userId" element={<Outlet />}>
          <Route path="profile" element={<UserProfile />} />
        </Route>
      </Routes>
    </BrowserRouter>
  );
}
```

#### 88. What is the difference between BrowserRouter and HashRouter?

- **BrowserRouter**: Uses the HTML5 History API to manage navigation, enabling clean URLs without the hash (`#`). It requires server-side configuration to handle routes correctly, especially for deep linking.
- **HashRouter**: Uses the hash (`#`) portion of the URL to simulate navigation. It doesn't require server-side configuration, as the hash is <mark>never sent to the server</mark>. This makes it suitable for environments where server-side routing isn't possible (e.g., static hosting).

#### 89. How React Router is different from the history library?

React Router is a routing library for React that provides a declarative API for defining routes and handling navigation. It manages components and URLs.

History library is a lower-level utility that only manages browser history (e.g., pushing and popping history entries). It doesn't handle UI rendering or routing, making it more generic and not React-specific.

React Router uses the history library internally but adds additional features like routing and component management.

#### 90. What are the `<Router>` components of React Router v6?

In React Router v6, the key `<Router>` components are:

- `<BrowserRouter>`: Uses the HTML5 history API to keep the UI in sync with the URL. It's commonly used for web applications.
- `<HashRouter>`: Uses URL hash fragments (#) to manage routing, making it suitable for static file hosting or legacy browsers that don't support the HTML5 history API.
- `<MemoryRouter>`: Keeps the URL in memory (no address bar changes), useful for non-browser environments like tests or embedded apps.
- `<StaticRouter>`: Used for server-side rendering (SSR), where routing is handled without a browser, typically in Node.js environments.

Each of these routers serves different use cases but provides the same routing functionality within a React app.

#### 91. What is the purpose of the push and replace methods of history?

The push and replace methods of the history library are used to manage the browser's history stack and control navigation.

##### `push`:

- Adds a new entry to the history stack, which means the user can navigate back to it using the browser's back button.
- Example: `history.push('/new-page')`

##### `replace`:

- Replaces the current entry in the history stack with a new one, meaning the user cannot go back to the previous page using the back button.
- Example: `history.replace('/new-page')`

#### 92. How would you implement route guards or private routes in React?

To implement private routes, create a component that checks if the user is authenticated before rendering the desired route.

Example:

```jsx
import { Navigate } from 'react-router-dom';

function PrivateRoute({ children }) {
  return isAuthenticated ? children : <Navigate to="/login" />;
}
```

- `PrivateRoute`: Checks authentication and either renders the children (protected routes) or redirects to the login page.
- `<Navigate>`: Replaces the deprecated `<Redirect>` for redirecting in React Router v6+.

#### 93. How do you handle 404 errors or page not found in React Router?

To handle 404 errors or page not found in React Router, create a catch-all route at the end of your route configuration that renders a custom 404 component.

Example:

```jsx
import { Routes, Route } from 'react-router-dom';

function NotFound() {
  return <h1>404 - Page Not Found</h1>;
}

function App() {
  return (
    <Routes>
      <Route path="/" element={<Home />} />
      <Route path="/about" element={<About />} />
      <Route path="*" element={<NotFound />} />
    </Routes>
  );
}
```

In this example, the `NotFound` component is rendered when no other routes match the URL, indicating a 404 error.

#### 94. How do you perform an automatic redirect after login in React Router?

To perform an automatic redirect after login in React Router, use the `useNavigate` hook to navigate to the desired route after successful authentication.

Example:

```jsx
import { useNavigate } from 'react-router-dom';

function Login() {
  const navigate = useNavigate();

  const handleLogin = () => {
    // Perform login logic
    navigate('/dashboard');
  };

  return (
    <div>
      <button onClick={handleLogin}>Login</button>
    </div>
  );
}
```

In this example, the `handleLogin` function navigates to the `/dashboard` route after successful login.

#### 95. Which state belongs in the URL rather than only in React state?

Put shareable, navigable choices such as search terms, filters, sorting, and pagination in the URL when Back, refresh, and copied links <mark>should reproduce the view</mark>. Keep transient interaction details such as hover or an unfinished password outside it.

Parse and validate values, supply defaults, and choose push versus replace based on the desired history behavior. Avoid duplicating URL state into local state without a clear synchronization policy. See [React Router state management](https://reactrouter.com/explanation/state-management).

#### 96. How do React Router declarative, data, and framework modes differ?

Declarative mode supplies URL matching and navigation through components such as `BrowserRouter`. Data mode adds route loaders, actions, and pending-state coordination through a data router. Framework mode adds build integration and route-module conventions, including rendering strategy support.

Choose the mode before proposing a loader or server-rendering API. A component under plain `BrowserRouter` <mark>does not gain data-router capabilities</mark> simply by calling a data Hook. See [choosing a mode](https://reactrouter.com/start/modes).

### Internationalization and accessible interfaces

#### 97. What are React Portals used for?

React Portals allow rendering children into a DOM node outside the parent component's hierarchy. This is useful for modals or tooltips that need to <mark>escape parent overflow or z-index constraints</mark>.

#### 98. How do you localize React applications?

Localization typically involves libraries like react-i18next or react-intl. Set up translation files for different languages and configure the library within your app using provided hooks or components.

```jsx
// Example using react-i18next
import { useTranslation } from 'react-i18next';

const MyComponent = () => {
  const { t } = useTranslation();
  return <p>{t('welcome_message')}</p>;
};
```

#### 99. What is `react-intl`?

`react-intl` is a library that provides internationalization (i18n) support for React applications. It helps in formatting numbers, dates, strings, and handling translation/localization. It integrates with the `Intl` API in JavaScript to provide locale-specific data and translation management.

#### 100. What are the main features of `react-intl`?

- **Formatted text**: Helps in formatting messages and strings with placeholders.
- **Number formatting**: Allows for formatting numbers, currencies, and percentages according to the locale.
- **Date and time formatting**: Helps in formatting dates and times in various formats based on the locale.
- **Plural and gender support**: Provides plural and gender-aware string formatting.

#### 101. What are the two ways of formatting in `react-intl`?

- Component-based formatting: Using React components like `<FormattedMessage />`, `<FormattedNumber />`, `<FormattedDate />`, etc., to format content.
- Hook-based formatting: Using hooks like `useIntl` for formatting messages, numbers, or dates imperatively within components.

#### 102. How to use `FormattedMessage` as a placeholder using `react-intl`?

You can use the `FormattedMessage` component to handle placeholders within strings. Placeholders are replaced dynamically with variables in the translated string.

```jsx
import { FormattedMessage } from 'react-intl';

function WelcomeMessage() {
  return (
    <FormattedMessage
      id="welcome"
      defaultMessage="Hello, {name}!"
      values={{ name: 'John' }}
    />
  );
}
```

Here, `{name}` is a placeholder, and `John` will replace it.

#### 103. How to access the current locale with React Intl?

You can access the current locale using the `useIntl` hook or the `IntlProvider`'s `locale` prop.

Using `useIntl`:

```jsx
import { useIntl } from 'react-intl';

function LocaleDisplay() {
  const intl = useIntl();
  return <div>Current locale: {intl.locale}</div>;
}
```

Using `IntlProvider`:

```jsx
<IntlProvider locale="en" messages={messages}>
  <MyComponent />
</IntlProvider>
```

Here, `locale="en"` defines the current locale.

#### 104. How to format date using `react-intl`?

You can format dates using the `<FormattedDate />` component or the `useIntl` hook's `formatDate` method.

Using `<FormattedDate />` component:

```jsx
import { FormattedDate } from 'react-intl';

function DateComponent() {
  return (
    <FormattedDate
      value={new Date()}
      year="numeric"
      month="long"
      day="2-digit"
    />
  );
}
```

Using `useIntl` hook:

```jsx
import { useIntl } from 'react-intl';

function DateComponent() {
  const intl = useIntl();
  const formattedDate = intl.formatDate(new Date(), {
    year: 'numeric',
    month: 'long',
    day: '2-digit',
  });
  return <div>{formattedDate}</div>;
}
```

These methods allow you to format the date in a locale-sensitive manner.

#### 105. What makes a portal-based modal accessible?

Use a native modal dialog or appropriate `role="dialog"` and `aria-modal="true"` semantics. Give the dialog an accessible name, <mark>move focus inside when it opens</mark>, keep keyboard focus within a modal, support Escape, and restore focus appropriately when it closes. Prevent interaction with background content while the modal is active.

A portal only changes DOM placement. Prefer a tested dialog primitive, then verify its keyboard behavior and labeling in the actual application. See the [WAI-ARIA modal dialog pattern](https://www.w3.org/WAI/ARIA/apg/patterns/dialog-modal/).

#### 106. How would you make form validation usable with a keyboard and screen reader?

Use visible labels connected to controls, associate error text with its input using `aria-describedby`, and set `aria-invalid` when a value is invalid. Preserve entered data after failure and provide clear guidance for correction.

On submission failure, consider an error summary or <mark>moving focus to the first invalid field</mark>. Do not communicate errors through color alone, and avoid announcing every keystroke as an urgent alert. See [WAI form notifications](https://www.w3.org/WAI/tutorials/forms/notifications/).

### Testing asynchronous behavior

#### 107. How do you test asynchronous code in React components?

To test asynchronous code in React components, you can use `async/await` with `waitFor` from React Testing Library to handle asynchronous operations like data fetching or API calls.

Example:

```jsx
import { render, screen, waitFor } from '@testing-library/react';
import MyComponent from './MyComponent';

test('fetches data and renders it', async () => {
  render(<MyComponent />);
  await waitFor(() => {
    expect(screen.getByText('Data loaded')).toBeInTheDocument();
  });
});
```

In this example, the test waits for the data to be loaded before asserting that the text 'Data loaded' is present.

#### 108. How do you mock API calls in React component tests?

To mock API calls in React component tests, you can use Jest's `jest.mock` to mock the API module and return mock data. This allows you to simulate API responses without making actual network requests.

Example:

```jsx
import { render, screen } from '@testing-library/react';

jest.mock('./api', () => ({
  fetchData: jest.fn(() => Promise.resolve('mocked data')),
}));

import MyComponent from './MyComponent';

test('fetches data and renders it', async () => {
  render(<MyComponent />);
  expect(screen.getByText('Loading...')).toBeInTheDocument();
  expect(await screen.findByText('mocked data')).toBeInTheDocument();
});
```

In this example, the `fetchData` function from the `api` module is mocked to return 'mocked data' for testing purposes.

#### 109. How do you test React hooks in functional components?

Render the hook inside a test using `renderHook` from `@testing-library/react`, then call `act` to <mark>drive any state updates</mark>.

```jsx
import { renderHook, act } from '@testing-library/react';
import useCounter from './useCounter';

test('increments counter', () => {
  const { result } = renderHook(() => useCounter());
  act(() => {
    result.current.increment();
  });
  expect(result.current.count).toBe(1);
});
```

Older sources import `renderHook` from `@testing-library/react-hooks`. That package was deprecated and merged into `@testing-library/react` in v13; use the import shown above.

#### 110. How do you test custom hooks in React?

Same approach as above: render the hook with `renderHook` and assert on `result.current`. For hooks that depend on context (e.g., a router or theme provider), pass a `wrapper` option.

```jsx
import { renderHook, act } from '@testing-library/react';
import useCustomHook from './useCustomHook';

test('hook behavior', () => {
  const { result } = renderHook(() => useCustomHook());
  act(() => {
    result.current.doSomething();
  });
  expect(result.current.value).toBe('expected value');
});

// With a context provider:
const wrapper = ({ children }) => (
  <MyProvider value="test">{children}</MyProvider>
);
const { result } = renderHook(() => useCustomHook(), { wrapper });
```

#### 111. What is Snapshot Testing in React?

Snapshot Testing in React is a testing technique that captures the rendered output of a component and saves it as a snapshot. Subsequent test runs compare the current output with the saved snapshot to <mark>detect any unexpected changes</mark>. If the output differs from the snapshot, the test fails, indicating that the component's output has changed.

Here's an example of using Snapshot Testing with Jest:

```jsx
import React from 'react';
import renderer from 'react-test-renderer';
import MyComponent from './MyComponent';

test('renders correctly', () => {
  const tree = renderer.create(<MyComponent />).toJSON();
  expect(tree).toMatchSnapshot();
});
```

In this example, the `renderer.create` function renders the `MyComponent` and converts it to a JSON tree. The `toMatchSnapshot` function saves the snapshot of the component's output. Subsequent test runs compare the current output with the saved snapshot, ensuring the component's output remains consistent.

#### 112. How do you test React components that use context?

To test React components that use context, you can wrap the component in a context provider with the desired context values for testing. This allows you to simulate the context values and test the component's behavior based on those values.

Example:

```jsx
import { render } from '@testing-library/react';
import { MyContextProvider } from './MyContextProvider';
import MyComponent from './MyComponent';

test('renders correctly with context', () => {
  const { getByText } = render(
    <MyContextProvider value="test value">
      <MyComponent />
    </MyContextProvider>,
  );
  expect(getByText('test value')).toBeInTheDocument();
});
```

In this example, the `MyComponent` is wrapped in a `MyContextProvider` with a specific context value for testing. The test verifies that the component renders correctly with the provided context value.

#### 113. How do you test React components that use Redux?

To test React components that use Redux, you can use the `redux-mock-store` library to create a mock store with the desired state for testing. This allows you to simulate the Redux store and test the component's behavior based on the state.

Example:

```jsx
import { render } from '@testing-library/react';
import configureStore from 'redux-mock-store';
import { Provider } from 'react-redux';
import MyComponent from './MyComponent';

const mockStore = configureStore([]);

test('renders correctly with Redux state', () => {
  const store = mockStore({ counter: 0 });
  const { getByText } = render(
    <Provider store={store}>
      <MyComponent />
    </Provider>,
  );
  expect(getByText('Counter: 0')).toBeInTheDocument();
});
```

In this example, the `MyComponent` is wrapped in a `Provider` with a mock Redux store containing the initial state `{ counter: 0 }` for testing. The test verifies that the component renders correctly with the provided Redux state.

#### 114. How would you test a debounced input without waiting in real time?

Enable the runner's fake timers, type through `user-event`, and <mark>advance the relevant delay inside `act`</mark> when it triggers React updates. Configure `userEvent.setup({ advanceTimers: jest.advanceTimersByTime })` for Jest so its own scheduled work advances correctly.

Assert that no request occurs before the delay and that the final value is submitted afterward. Flush pending timers and restore real timers during cleanup. See [Testing Library fake timers](https://testing-library.com/docs/using-fake-timers/).
