# Different React Packages Topics

## Git hooks

Git hooks are just files in the `.git/hooks` directory which get executed on a given git command/event.

We use [`lint-staged`](https://www.npmjs.com/package/lint-staged) to lint the changed files before a commit and [`npm-autoinstaller`](https://www.npmjs.com/package/npm-autoinstaller) which installs/updates dependencies if the `package.json` got changed (on pull and checkout).

## Props in TypeScript

```typescript
type SliderProps = {
    // what to put in here?
};

const Slider = ({ color, value, setValue }: SliderProps) => (
    <input type="range" value={value} onChange={setValue} style={{ color }} />
);

export default compose(
    withState('value', 'setValue', 10),
)(Slider);



// ---------------------------------------------------------------------------



const MyPage = () => (
    <Slider color="red" />
);
```

<!--
    When we put `color, `value` and `setValue` in `SliderProps`, we get an error when using the component.
    If we don't put it there, we get an error inside the component.
    How can we solve this?

    Separate inner & outer props.
    compose<OuterProps>(...)
-->
