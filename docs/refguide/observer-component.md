# @observer

The `observer` function / decorator can be used to turn ReactJS components into reactive components.
It wraps the component's render function in `mobx.autorun` to make sure that any data that is used during the rendering of a component forces a rerendering upon change.
It is available through the separate `mobx-react` package.

```javascript
import {observer} from "mobx-react";

var timerData = observable({
	secondsPassed: 0
});

setInterval(() => {
	timerData.secondsPassed++;
}, 1000);

@observer class Timer extends React.Component {
	render() {
		return (<span>Seconds passed: { this.props.timerData.secondsPassed } </span> )
	}
});

React.render(<Timer timerData={timerData} />, document.body);
```

_Note: when `observer` needs to be combined with other decorators or higher-order-components, make sure that `observer` is the most inner (first applied) decorator;
otherwise it might do nothing at all._

### Characteristics of reactive components

So here we are, simple, straightforward reactive components that will render whenever needed. These components have some interesting characteristics:

* They only subscribe to the data structures that were actively used during the last render. This means that you cannot under-subscribe or over-subscribe. You can even use data in your rendering that will only be available at later moment in time. This is ideal for asynchronously loading data.
* You are not required to declare what data a component will use. Instead, dependencies are determined at runtime and tracked in a very fine-grained manner.
* Usually reactive components have no state. But you are still free to use state.
* `@observer` implements `shouldComponentUpdate` so that children are not re-rendered unnecessary.
* Reactive components sideways load data; parent components won't re-render unnecessarily even when child components will.
* `@observer` does not depend on React's context system.

### Reactive local component state

Just like normal classes, you can introduce observable properties on a component by using the `@observable` decorator.
This means that you can have local state in components that doesn't need to be manged by React's verbose and imperative `setState` mechanism, but is as powerful.
The reactive state will be picked up by `render` but will not explicitly invoke other React lifecycle methods like `componentShouldUpdate` or `componentWillUpdate`.
If you need those, just use the normal React `state` object.

The example above could also have been written as:

```javascript
import {observer} from "mobx-react";
import {observable} from "mobx";

@observer class Timer extends React.Component {
	@observable secondsPassed = 0;
	
	componentWillMount() {
		setInterval(() => {
			this.secondsPassed++;
		}, 1000);
	}
	
	render() {
		return (<span>Seconds passed: { this.secondsPassed } </span> )
	}
});

React.render(<Timer timerData={timerData} />, document.body);
```

### MobX-React-DevTools

In combination with `@observer` you can use the MobX-React-DevTools, it shows exactly when your components are rerendered and you can inspect the data dependencies of your components.

### Alternative syntaxes

In ES5 environments, reactive components can be simple declared using `observer(React.createClass({ ... `. See also the [syntax guide](../best/syntax)

When using `@observer`, a lot of components will become stateless.
In such cases you can also write reactive function components using `@observer` (this works also on ES5 and with React 0.13):

```javascript
const Timer = observer( ({timerData}) =>
	(<span>Seconds passed: { timerData.secondsPassed } </span> )
);
```

### Enabling decorators in your transpiler

Decorators are not supported by default when using TypeScript or Babel pending a definitive definition in the ES standard.
* For _typescript_, enable the `--experimentalDecorators` compiler flag or set the compiler option `experimentalDecorators` to `true` in `tsconfig.json` (Recommended)
* For _babel5_, make sure `--stage 0` is passed to the babel CLI
* For _babel6_, see the example configuration as suggested in this [issue](https://github.com/mobxjs/mobx/issues/105)
