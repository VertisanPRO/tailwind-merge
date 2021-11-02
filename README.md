<div align="center">
    <br />
    <a href="https://github.com/dcastil/tailwind-merge">
        <!-- AUTOGENERATED START logo-image --><img src="https://github.com/dcastil/tailwind-merge/raw/v0.8.2/assets/logo.svg" alt="tailwind-merge" width="221px" /><!-- AUTOGENERATED END -->
    </a>
</div>

# tailwind-merge

Utility function to efficiently merge [Tailwind CSS](https://tailwindcss.com) classes in JS without style conflicts.

```ts
import { twMerge } from 'tailwind-merge'

twMerge('px-2 py-1 bg-red hover:bg-dark-red', 'p-3 bg-[#B91C1C]')
// → 'hover:bg-dark-red p-3 bg-[#B91C1C]'
```

-   Supports Tailwind v2.0 up to v2.2, support for newer versions will be added continuously
-   Works in Node >=12 and all modern browsers
-   Fully typed
-   [<!-- AUTOGENERATED START package-gzip-size -->4.8 kB<!-- AUTOGENERATED END --> minified + gzipped](https://bundlephobia.com/package/tailwind-merge) (<!-- AUTOGENERATED START package-composition -->96.5% self, 3.5% hashlru<!-- AUTOGENERATED END -->)

## Early development

This library is in an early pre-v1 development stage and might have some bugs and inconveniences here and there. I use the library in production and intend it to be sufficient for production use, as long as you're fine with some potential breaking changes in minor releases until v1 (lock the version range to patch releases with `~` in your `package.json` to prevent accidental breaking changes).

I want to keep the library on v0 until I feel confident enough that there aren't any major bugs or flaws in its API and implementation. If you find a bug or something you don't like, please [submit an issue](https://github.com/dcastil/tailwind-merge/issues/new) or a pull request. I'm happy about any kind of feedback!

## What is it for

If you use Tailwind with a component-based UI renderer like [React](https://reactjs.org) or [Vue](https://vuejs.org), you're probably familiar with the situation that you want to change some styles of a component, but only in one place.

```jsx
import React from 'react'

function MyGenericInput(props) {
    const className = `border rounded px-2 py-1 ${props.className || ''}`
    return <input {...props} className={className} />
}

function MySlightlyModifiedInput(props) {
    return (
        <MyGenericInput
            {...props}
            className="p-3" // ← Only want to change some padding
        />
    )
}
```

When the `MySlightlyModifiedInput` is rendered, an input with the className `border rounded px-2 py-1 p-3` gets created. But because of the way the [CSS cascade](https://developer.mozilla.org/en-US/docs/Web/CSS/Cascade) works, the styles of the `p-3` class are ignored. The order of the classes in the `className` string doesn't matter at all and the only way to apply the `p-3` styles is to remove both `px-2` and `py-1`.

This is where tailwind-merge comes in.

```jsx
function MyGenericInput(props) {
    // ↓ Now `props.className` can override conflicting classes
    const className = twMerge('border rounded px-2 py-1', props.className)
    return <input {...props} className={className} />
}
```

tailwind-merge makes sure to override conflicting classes and keeps everything else untouched. In the case of the `MySlightlyModifiedInput`, the input now only renders the classes `border rounded p-3`.

## Features

### Optimized for speed

-   Results get cached by default, so you don't need to worry about wasteful re-renders. The library uses a [LRU cache](<https://en.wikipedia.org/wiki/Cache_replacement_policies#Least_recently_used_(LRU)>) which stores up to 500 different results. The cache size can be modified or opt-out of by using [`extendTailwindMerge`](#extendtailwindmerge).
-   Expensive computations happen upfront so that `twMerge` calls without a cache hit stay fast.
-   These computations are called lazily on the first call to `twMerge` to prevent it from impacting app startup performance if it isn't used initially.

### Last conflicting class wins

```ts
twMerge('p-5 p-2 p-4') // → 'p-4'
```

### Allows refinements

```ts
twMerge('p-3 px-5') // → 'p-3 px-5'
twMerge('inset-x-4 right-4') // → 'inset-x-4 right-4'
```

### Resolves non-trivial conflicts

```ts
twMerge('inset-x-px -inset-1') // → '-inset-1'
twMerge('bottom-auto inset-y-6') // → 'inset-y-6'
twMerge('inline block') // → 'block'
```

### Supports prefixes and stacked prefixes

```ts
twMerge('p-2 hover:p-4') // → 'p-2 hover:p-4'
twMerge('hover:p-2 hover:p-4') // → 'hover:p-4'
twMerge('hover:focus:p-2 focus:hover:p-4') // → 'focus:hover:p-4'
```

### Supports custom values

```ts
twMerge('bg-black bg-[color:var(--mystery-var)]') // → 'bg-[color:var(--mystery-var)]'
twMerge('grid-cols-[1fr,auto] grid-cols-2') // → 'grid-cols-2'
```

### Supports important modifier

```ts
twMerge('!p-3 !p-4 p-5') // → '!p-4 p-5'
twMerge('!right-2 !-inset-x-1') // → '!-inset-x-1'
```

### Preserves non-Tailwind classes

```ts
twMerge('p-5 p-2 my-non-tailwind-class p-4') // → 'my-non-tailwind-class p-4'
```

### Supports custom colors out of the box

```ts
twMerge('text-red text-secret-sauce') // → 'text-secret-sauce'
```

### Ignores `undefined`, `null` and `false` values

```ts
twMerge('some-class', undefined, null, false) // → 'some-class'
```

## API reference

Reference to all exports of tailwind-merge.

### `twMerge`

```ts
function twMerge(...classLists: Array<string | undefined | null | false>): string
```

Default function to use if you're using the default Tailwind config or are close enough to the default config. You can use this function if all of the following points apply to your Tailwind config:

-   Only using color names which don't clash with other Tailwind class names
-   Only deviating by number values from number-based Tailwind classes
-   Only using font-family classes which don't clash with default font-weight classes
-   Sticking to default Tailwind config for everything else

If some of these points don't apply to you, it makes sense to test whether `twMerge` still works as intended with your custom classes. Otherwise, you can create your own custom merge function with [`extendTailwindMerge`](#extendtailwindmerge).

### `getDefaultConfig`

```ts
function getDefaultConfig(): Config
```

Function which returns the default config used by tailwind-merge. The tailwind-merge config is different from the Tailwind config. It is optimized for small bundle size and fast runtime performance because it is expected to run in the browser.

### `extendTailwindMerge`

```ts
function extendTailwindMerge(
    configExtension: Partial<Config>,
    ...createConfig: Array<(config: Config) => Config>
): TailwindMerge
function extendTailwindMerge(...createConfig: Array<(config: Config) => Config>): TailwindMerge
```

Function to create merge function with custom config which extends the default config. Use this if you use the default Tailwind config and just extend it in some places.

You provide it a `configExtension` object which gets [merged](#mergeconfigs) with the default config.

```ts
const customTwMerge = extendTailwindMerge({
    cacheSize: 0, // ← Disabling cache
    prefixes: [
        'my-custom-prefix', // ← Adding custom prefix
    ],
    // ↓ Here you define class groups
    classGroups: {
        // ↓ The `foo` key here is the class group ID
        //   ↓ Creates group of classes which have conflicting styles
        //     Classes here: foo, foo-2, bar-baz, bar-baz-1, bar-baz-2
        foo: ['foo', 'foo-2', { 'bar-baz': ['', '1', '2'] }],
        //   ↓ Functions can also be used to match classes.
        //     Classes here: qux-auto, qux-1000, qux-1001, …
        bar: [{ qux: ['auto', (value) => Number(value) >= 1000] }],
    },
    // ↓ Here you can define additional conflicts across different groups
    conflictingClassGroups: {
        // ↓ ID of class group which creates a conflict with …
        //     ↓ … classes from groups with these IDs
        foo: ['bar'],
    },
})
```

Additionally you can pass multiple `createConfig` functions (more to that in [`createTailwindMerge`](#createtailwindmerge)) which is convenient if you want to combine your config with third-party plugins.

```ts
const customTwMerge = extendTailwindMerge({ … }, withSomePlugin)
```

If you only use plugins, you can omit the `configExtension` object as well.

```ts
const customTwMerge = extendTailwindMerge(withSomePlugin)
```

### `createTailwindMerge`

```ts
function createTailwindMerge(
    ...createConfig: [() => Config, ...Array<(config: Config) => Config>]
): TailwindMerge
```

Function to create merge function with custom config. Use this function instead of [`extendTailwindMerge`](#extendtailwindmerge) if you don't need the default config or want more control over the config.

You need to provide a function which resolves to the config tailwind-merge should use for the new merge function. You can either extend from the default config or create a new one from scratch.

```ts
// ↓ Callback passed to `createTailwindMerge` is called when
//   `customTwMerge` gets called the first time.
const customTwMerge = createTailwindMerge(() => {
    const defaultConfig = getDefaultConfig()

    return {
        cacheSize: 0,
        prefixes: [...defaultConfig.prefixes, 'my-custom-prefix'],
        classGroups: {
            ...defaultConfig.classGroups,
            foo: ['foo', 'foo-2', { 'bar-baz': ['', '1', '2'] }],
            bar: [{ qux: ['auto', (value) => Number(value) >= 1000] }],
        },
        conflictingClassGroups: {
            ...defaultConfig.conflictingClassGroups,
            foo: ['bar'],
        },
    }
})
```

Same as in [`extendTailwindMerge`](#extendtailwindmerge) you can use multiple `createConfig` functions which is convenient if you want to combine your config with third-party plugins. Just keep in mind that the first `createConfig` function does not get passed any arguments, whereas the subsequent functions get each passed the config from the previous function.

```ts
const customTwMerge = createTailwindMerge(getDefaultConfig, withSomePlugin, (config) => ({
    // ↓ Config returned by `withSomePlugin`
    ...config,
    classGroups: {
        ...config.classGroups,
        mySpecialClassGroup: [{ special: ['1', '2'] }],
    },
}))
```

But don't merge configs like that. Use [`mergeConfigs`](#mergeconfigs) instead.

### `mergeConfigs`

```ts
function mergeConfigs(baseConfig: Config, configExtension: Partial<Config>): Config
```

Helper function to merge multiple config objects. Objects are merged, arrays are concatenated, scalar values are overriden and `undefined` does nothing. The function assumes that both parameters are tailwind-merge config objects and shouldn't be used as a generic merge function.

```ts
const customTwMerge = createTailwindMerge(getDefaultConfig, (config) =>
    mergeConfigs(config, {
        classGroups: {
            // ↓ Adding new class group
            mySpecialClassGroup: [{ special: ['1', '2'] }],
            // ↓ Adding value to existing class group
            animate: ['animate-magic'],
        },
    })
)
```

### `validators`

```ts
interface Validators {
    isLength(classPart: string): boolean
    isCustomLength(classPart: string): boolean
    isInteger(classPart: string): boolean
    isCustomValue(classPart: string): boolean
    isAny(classPart: string): boolean
}
```

An object containing all the validators used in tailwind-merge. They are useful if you want to use a custom config with [`extendTailwindMerge`](#extendtailwindmerge) or [`createTailwindMerge`](#createtailwindmerge). E.g. the `classGroup` for padding is defined as

```ts
const paddingClassGroup = [{ p: [validators.isLength] }]
```

A brief summary for each validator:

-   `isLength` checks whether a class part is a number (`3`, `1.5`), a fraction (`3/4`), a custom length (`[3%]`, `[4px]`, `[length:var(--my-var)]`), or one of the strings `px`, `full` or `screen`.
-   `isCustomLength` checks for custom length values (`[3%]`, `[4px]`, `[length:var(--my-var)]`).
-   `isInteger` checks for integer values (`3`) and custom integer values (`[3]`).
-   `isCustomValue` checks whether the class part is enclosed in brackets (`[something]`)
-   `isAny` always returns true. Be careful with this validator as it might match unwanted classes. I use it primarily to match colors or when I'm ceertain there are no other class groups in a namespace.

### `Config`

```ts
interface Config { … }
```

TypeScript type for config object. Useful if you want to build a `createConfig` function but don't want to define it inline in [`extendTailwindMerge`](#extendtailwindmerge) or [`createTailwindMerge`](#createtailwindmerge).

## Writing plugins

This library supports classes of the core Tailwind library out of the box, but not classes of any plugins. But it's possible and hopefully easy to write third-party plugins for tailwind-merge. In case you want to write a plugin, I invite you to follow these steps:

-   Create a package called `tailwind-merge-magic-plugin` with tailwind-merge as peer dependency which exports a function `withMagic` and replace "magic" with your plugin name.
-   This function would be ideally a `createConfig` function which takes a config object as argument and returns the modified config object.
-   If you create new class groups, prepend them with `magic.` (your plugin name with a dot at the end) so they don't collide with class group names from other plugins or even future class groups in tailwind-merge itself.
-   Use the [`validators`](#validators) and [`mergeConfigs`](#mergeconfigs) from tailwind-merge to extend the config with magic.

Here is an example of how a plugin could look like:

```ts
import { mergeConfigs, validators, Config } from 'tailwind-merge'

export function withMagic(config: Config): Config {
    return mergeConfigs(config, {
        classGroups: {
            'magic.my-group': [{ magic: [validators.isLength, 'wow'] }],
        },
    })
}
```

This plugin can then be used like this:

```ts
import { extendTailwindMerge } from 'tailwind-merge'
import { withMagic } from 'tailwind-merge-magic-plugin'

const twMerge = extendTailwindMerge(withMagic)
```

Also feel free to check out [tailwind-merge-rtl-plugin](https://github.com/vltansky/tailwind-merge-rtl-plugin) as a real example of a tailwind-merge plugin.

## Versioning

This package follows the [SemVer](https://semver.org) versioning rules. More specifically:

-   Patch version gets incremented when unintended behaviour is fixed which doesn't break any existing API. Note that bug fixes can still alter which styles are applied. E.g. a bug gets fixed in which the conflicting classes `inline` and `block` weren't merged correctly so that both would end up in the result.

-   Minor version gets incremented when additional features are added which don't break any existing API. However, a minor version update might still alter which styles are applied if you use Tailwind features not yet supported by tailwind-merge. E.g. a new Tailwind prefix `magic` gets added to this package which changes the result of `twMerge('magic:px-1 magic:p-3')` from `magic:px-1 magic:p-3` to `magic:p-3`.

-   Major version gets incremented when breaking changes are introduced to the package API. E.g. the return type of `twMerge` changes.

-   `alpha` releases might introduce breaking changes on any update. Whereas `beta` releases only introduce new features or bug fixes.

-   Releases with major version 0 might introduce breaking changes on a minor version update.

-   A changelog is documented in [GitHub Releases](https://github.com/dcastil/tailwind-merge/releases).
