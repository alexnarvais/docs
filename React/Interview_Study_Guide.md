# React Interview Study Guide

This guide outlines the key concepts and topics that software companies typically focus on during React interviews. Mastering these areas will prepare you for both theoretical questions and practical coding challenges.

## 1. Core Concepts & Fundamentals

Understanding the basics of how React works internally is crucial.

*   **Virtual DOM & Reconciliation:**
    *   What is the Virtual DOM, and why does React use it?
    *   Explain the reconciliation process and the diffing algorithm (React Fiber).
    *   How does this approach improve performance compared to direct DOM manipulation?

*   **Components:**
    *   Difference between Functional and Class components.
    *   When to use one over the other (Note: Functional components with Hooks are the modern standard).
    *   Component lifecycle methods (for class components) vs. `useEffect` (for functional components).

*   **Props vs. State:**
    *   Clearly define the role of `props` (immutable, passed down) and `state` (mutable, managed internally).
    *   Explain how data flows in React (unidirectional data flow).
    *   What is "prop drilling" and how can it be avoided?

*   **JSX:**
    *   What is JSX?
    *   How is it different from HTML, and why is it used?
    *   How does Babel transpile JSX into standard JavaScript (e.g., `React.createElement()`)?

*   **Event Handling:**
    *   How are events handled in React (e.g., `onClick`, `onChange`)?
    *   Explain the concept of Synthetic Events.

## 2. React Hooks (Modern Standard)

Hooks are a primary focus in modern interviews and demonstrate competency with current best practices.

*   **`useState`:**
    *   How to manage state in functional components.
    *   Explain that state updates are asynchronous and how to handle updates that depend on the previous state (using a function in `useState` setter).

*   **`useEffect`:**
    *   Explain its purpose for handling side effects (data fetching, DOM manipulation, subscriptions).
    *   Master the dependency array:
        *   `[]` (runs once after the initial render).
        *   No array (runs after every render).
        *   Array with dependencies (runs when dependencies change).
    *   How to perform cleanup to avoid memory leaks (returning a function from `useEffect`).

*   **`useContext`:**
    *   How to use the Context API for application-wide state management.
    *   When is it a good alternative to libraries like Redux?

*   **`useRef`:**
    *   Accessing and interacting with the DOM directly.
    *   Storing mutable values that do not trigger a component re-render.

*   **`useMemo` & `useCallback`:**
    *   **Crucial for performance optimization.**
    *   `useMemo`: Caches a computed value.
    *   `useCallback`: Caches an entire function instance (useful for preventing unnecessary re-renders of child components that receive callbacks as props).

*   **Custom Hooks:**
    *   Explain how to create custom hooks to share and reuse stateful logic across multiple components.

## 3. Advanced Topics & Best Practices

Demonstrate an understanding of building efficient and robust applications.

*   **Performance Optimization:**
    *   `React.memo` (for functional components) vs. `PureComponent` (for class components).
    *   Techniques for optimizing large lists (windowing/virtualization using libraries like `react-window`).
    *   Lazy loading components and data fetching using `React.lazy` and `Suspense`.
    *   The importance of using unique `key` props when rendering lists.

*   **State Management:**
    *   Discuss different strategies: Context API, Redux, Zustand, React Query.
    *   Be prepared to justify why you would choose a specific library for a given project.

*   **Error Boundaries:**
    *   How to catch JavaScript errors in the component tree and display a graceful fallback UI.

*   **Forms:**
    *   Differentiate between controlled and uncontrolled components when handling form input data.

*   **Routing:**
    *   Familiarity with React Router for handling navigation and different views within a single-page application.

## 4. Practical Skills

Interviewers will want to see you apply your knowledge to real-world scenarios.

*   **Data Fetching:**
    *   Demonstrate fetching data from an API (using `fetch` or Axios).
    *   Handling different states: `loading`, `error`, and successful `data` presentation.
    *   Consider modern data fetching libraries like React Query or SWR.

*   **Testing:**
    *   Discuss how you test components.
    *   Mention tools: Jest (test runner) and React Testing Library (RTL - for testing user behavior).

*   **Debugging:**
    *   Familiarity with using React DevTools (browser extension) for inspecting component trees, props, and state.

## 5. General Knowledge (JavaScript/TypeScript)

React is just JavaScript, so core language fundamentals are always tested.

*   **JavaScript:** Closures, scope, prototypes, the event loop, Promises/async-await, and essential array methods (`map`, `filter`, `reduce`).
*   **TypeScript:** Familiarity with typing props, state, and API responses is often a major plus.