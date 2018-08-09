# Different React and TypeScript Topics

## Git hooks

Git hooks are just files in the `.git/hooks` directory which get executed on a given git command/event.

We use [`lint-staged`](https://www.npmjs.com/package/lint-staged) to lint the changed files before a commit and [`npm-autoinstaller`](https://www.npmjs.com/package/npm-autoinstaller) which installs/updates dependencies if the `package.json` got changed (on pull and checkout).

The later one can prevent errors when someone checks out another branch or pulls changes which require updated or new npm packages.

It can get disabled (but I don't recomment it, only if you always have that in mind and you are sure about it) by creating a `autoinstaller.json` file with the following content:
```json
{
  "npm": {
    "do": "nothing"
  }
}
```

## Props in TypeScript

Typings of props (or also typings in general) can be a bit tricky depending on the use-case.

Consider we have the following component:

```tsx
const Slider = ({ color, value, setValue }: SliderProps) => (
    <input type="range" value={value} onChange={setValue} style={{ color }} />
);

export default compose(
    withState('value', 'setValue', 10),
)(Slider);
```

And want to use it that way:

```tsx
const MyPage = () => (
    <Slider color="red" />
);
```

<details>
  <summary>How should the `SliderProps` type look like?</summary>

```tsx
type SliderProps = {
    color: string;
};

type SliderPropsInner = SliderProps & {
    value: number;
    setValue: (fn: (currentValue: number) => number) => void;
};
```

When we would define all props on the `SliderProps`, we **have** to specify all props when using it.
But we can't, because `value` and `setValue` get added by a higher order component.
And making the `value` and `setValue` optional (`value?: number`) would require us to use a `typeof` check every time we want to use the prop.

So we split the props into normal props (`SliderProps`) and inner props (`SliderPropsInner`).
The normal props should be the public props while the inner props are the ones which are available inside the component.
To easily distinguish them, inner props should *always* have the `Inner` suffix.

Now that we use two different types, we have to change the component a bit:
```tsx
const Slider = ({ color, value, setValue }: SliderPropsInner) => (   // <- use inner props here
    <input type="range" value={value} onChange={setValue} style={{ color }} />
);

export default compose<SliderPropsInner, SliderProps>(   // <- specify both, the inner and outer props
    withState('value', 'setValue', 10),
)(Slider);
```
</details>

## Component types in TypeScript

There are a lot of different ways to type a react component.
Here are the most common ones with their differences explained and when to use them.

### `React.ReactElement<Element>`

Specifies a react element of the given type. It can be a normal DOM element or a custom react component.

```tsx
const List = ({ item: React.ReactElement<HTMLDivElement> }) => (
    <div>First item: {item}</div>
);

// ..

<List item={<div>test</div>} />
```

You can also use `any` in the place of `HTMLDivElement` to allow all possible elements.

### `React.ReactNode`

Extends `ReactElement` by also allowing just strings, numbers and booleans which don't have an element. This is the type of the `children` prop.

```tsx
const List = ({ item: React.ReactNode }) => (
    <div>First item: {item}</div>
);

// ..

<List item={<div>test</div>} />
<List item={<MyItemComponent>test</MyItemComponent>} />
<List item={'strings are now also possible'} />
```

### `React.ComponentType`

The previous types require elements, which means you can't use them for components where you need to specify props. This type is now exactly for this use-case.

```tsx
const List = ({ Item: React.ComponentType }) => (
    <div>
        First item:
        <Item index={0} />
    </div>
);

// ..

<List Item={MyItemComponent}>
```

To only allow class- & stateful components, you could also use `React.ComponentClass` or to only allow stateless functional components `React.StatelessComponent`.
`React.ComponentType` actually merged these two types.

### `React.Component` and `React.PureComponent`

These one get often mixed and end up in a TypeScript error.
In fact, using them would work in other programming languages but `React.Component` and `React.PureComponent` are classes and not types/interfaces and TypeScript doesn't seem to like that.

## Exports

When exporting a react component, we should always use default exports.

```typescript
export default MyComponent; // good
export const MyComponent = .. // not so good
```

When using default exports, they can simply be named anything the developer wants when importing them. It is also a bit less verbose when renaming default exports.

```typescript
// The `teaser` and `video-teaser` package both export a `Teaser` component

import Teaser from 'teaser';

import VideoTeaser from 'video-teaser'; // short renamed import using the default export
import { Teaser as VideoTeaser } from 'video-teaser'; // longer renamed import when no default export is used
```

Of course, when multiple components get exported by a package, named exports have to be used then.

## Import paths

We often see these ugly import paths (`'../../../../../components/Teaser'`) in js projects.
It is hard to find out to which folder this import points, specially if components can be nested and there can be multiple `components` folder.

TypeScript has a `baseUrl` setting which we set to the root folder of the project.
This means, when the `components` folder from the example above is in the root folder of our project, we can simply import it this way instead: `'components/Teaser'`.

This works for all root folders, and in our case, the most used root folders would be `components`, `pages` and `shared`. So if you see one of these imports, you always know that you can start looking for it in the root folder.

<details>
  <summary>Is there still a reason to use a relative import?</summary>

Yes, for example when we import a child or parent component, a relative import makes more sense than a global one. This way, we can simply move the component to another place and don't have to update any import.
</details>

## `why-did-you-update`

Unnecessary re-renders can cause performance problems in react.
To discover this more easily, we include the [`why-did-you-update`](https://www.npmjs.com/package/why-did-you-update) package during development which tries to find such unecessary re-renders of components and warns you about them in the console.
