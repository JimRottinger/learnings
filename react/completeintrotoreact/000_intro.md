# Complete Intro to React V3 by Brian Holt, Frontend Masters

These notes pertain to me going through the Complete Intro to React V3 workshop hosted on Frontend Masters and given by Brian Holt. I am unsure if I am going to go through the entire course as my goal is to learn enough React to be able to write mobile applications using React Native.

## Simple React Components

You can think of React as being split into two parts. There is the top-level React which is shared between react native, react DOM, react VR, etc. It is the same. This is the library in which you create components, set up lifecycle events, etc. These libraries implement a glue layer that convert React components to DOM nodes, native components, VR components, etc. In this workshop, we are going to be using React DOM.

Here is our first React component:

```markup
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF=8">
    <title>Complete Intro to React</title>
</head>
<body>
    <div id="app"></div>
    <script type="text/javascript" src="node_modules/react/dist/react.js"></script>
    <script type="text/javascript" src="node_modules/react-dom/dist/react-dom.js"></script>
    <script type="text/javascript">
        const MyFirstComponent = function ( ) {
            return React.createElement('div', null,
                React.createElement('h1', null, 'This is my first component')
            )
        }

        ReactDOM.render(
            React.createElement(MyFirstComponent),
            document.getElementById('app')
        )
    </script>
</body>
</html>
```

Here, we included React and ReactDOM. Then, we use React to create a div element with an h1 inside of it. That, on its own, does nothing. It is not until we call render on ReactDOM and create an instance of that component and tell it where to render it, the `app` element. This creteElement function is variadic, meaning it can take in any number of arguments.

## React Paradigm

What we just saw was the creation of a reusable component. We made a simple h1 component, but you can imagine a very useful resuable component such as a date-picker input. This is the core of the react paradigm. We want a component-based architecture instead of an entangled MVC mess. That just didn't make sense for interfaces. React components encapsulates all of that logic in a single component and a single file.

But a component wouldn't be very reusable if it were not configurable. We want a common component to be able to work with our own, unique said of data. This is where props come in. We can pass a props object into the component constructor and have that component use that data to render itself. We can use those props in the markup itself or we can add attributes to the markup \(in the argument place that was previously null\)

```javascript
const myTitle = function (props) {
    return (
        React.createElement('div', { id: "my-first-component" },
            React.createElement('h1', { style: { color: props. color } }, props.title)
        )
    )
}
```

With that in mind, in seems a bit wordy to have to write createElement over and over and over again. With component-based composition, we just want to create elements with the name of the component and to have it just work. This is where JSX comes in which we will see in the next section

