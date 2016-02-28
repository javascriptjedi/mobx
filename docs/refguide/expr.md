# Expr

`expr` can be used to create temporarily computed values inside computed values.
Nesting computed values is useful when you use it to split of cheap computations to prevent expensive computations to run.

In the following example the expression prevents that the `TodoView` component is re-rendered if the selection changes.
Instead the component will only re-render each time that the relevant todo is (de)selected.

```javascript
const TodoView = observer(({todo, editorState}) => {
    const isSelected = mobx.expr(() => editorState.selection === todo);
    return <div className={isSelected ? "todo todo-selected" : "todo"}>{todo.title}</div>;
});
```