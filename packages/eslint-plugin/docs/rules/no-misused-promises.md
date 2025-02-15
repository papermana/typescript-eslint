# `no-misused-promises`

Avoid using Promises in places not designed to handle them.

This rule forbids providing Promises to logical locations such as if statements in places where the TypeScript compiler allows them but they are not handled properly.
These situations can often arise due to a missing `await` keyword or just a misunderstanding of the way async
functions are handled/awaited.

:::tip
`no-misused-promises` only detects code that provides Promises to incorrect _logical_ locations.
See [`no-floating-promises`](./no-floating-promises.md) for detecting unhandled Promise _statements_.
:::

## Rule Details

This rule accepts a single option which is an object with `checksConditionals`
and `checksVoidReturn` properties indicating which types of misuse to flag.
Both are enabled by default.

## Options

```ts
type Options = [
  {
    checksConditionals?: boolean;
    checksVoidReturn?: boolean | ChecksVoidReturnOptions;
  },
];

interface ChecksVoidReturnOptions {
  arguments?: boolean;
  attributes?: boolean;
  properties?: boolean;
  returns?: boolean;
  variables?: boolean;
}

const defaultOptions: Options = [
  {
    checksConditionals: true,
    checksVoidReturn: true,
  },
];
```

### `"checksConditionals"`

If you don't want to check conditionals, you can configure the rule with `"checksConditionals": false`:

```json
{
  "@typescript-eslint/no-misused-promises": [
    "error",
    {
      "checksConditionals": false
    }
  ]
}
```

Doing so prevents the rule from looking at code like `if (somePromise)`.

### `"checksVoidReturn"`

Likewise, if you don't want functions that return promises where a void return is
expected to be checked, your configuration will look like this:

```json
{
  "@typescript-eslint/no-misused-promises": [
    "error",
    {
      "checksVoidReturn": false
    }
  ]
}
```

You can disable selective parts of the `checksVoidReturn` option by providing an object that disables specific checks.
The following options are supported:

- `arguments`: Disables checking an asynchronous function passed as argument where the parameter type expects a function that returns `void`
- `attributes`: Disables checking an asynchronous function passed as a JSX attribute expected to be a function that returns `void`
- `properties`: Disables checking an asynchronous function passed as an object property expected to be a function that returns `void`
- `returns`: Disables checking an asynchronous function returned in a function whose return type is a function that returns `void`
- `variables`: Disables checking an asynchronous function used as a variable whose return type is a function that returns `void`

For example, if you don't mind that passing a `() => Promise<void>` to a `() => void` parameter or JSX attribute can lead to a floating unhandled Promise:

```json
{
  "@typescript-eslint/no-misused-promises": [
    "error",
    {
      "checksVoidReturn": {
        "arguments": false,
        "attributes": false
      }
    }
  ]
}
```

### `checksConditionals: true`

Examples of code for this rule with `checksConditionals: true`:

<!--tabs-->

#### ❌ Incorrect

```ts
const promise = Promise.resolve('value');

if (promise) {
  // Do something
}

const val = promise ? 123 : 456;

while (promise) {
  // Do something
}
```

#### ✅ Correct

```ts
const promise = Promise.resolve('value');

// Always `await` the Promise in a conditional
if (await promise) {
  // Do something
}

const val = (await promise) ? 123 : 456;

while (await promise) {
  // Do something
}
```

<!--/tabs-->

### `checksVoidReturn: true`

Examples of code for this rule with `checksVoidReturn: true`:

<!--tabs-->

#### ❌ Incorrect

```ts
[1, 2, 3].forEach(async value => {
  await doSomething(value);
});

new Promise(async (resolve, reject) => {
  await doSomething();
  resolve();
});

const eventEmitter = new EventEmitter();
eventEmitter.on('some-event', async () => {
  synchronousCall();
  await doSomething();
  otherSynchronousCall();
});
```

#### ✅ Correct

```ts
// for-of puts `await` in outer context
for (const value of [1, 2, 3]) {
  await doSomething(value);
}

// If outer context is not `async`, handle error explicitly
Promise.all(
  [1, 2, 3].map(async value => {
    await doSomething(value);
  }),
).catch(handleError);

// Use an async IIFE wrapper
new Promise((resolve, reject) => {
  // combine with `void` keyword to tell `no-floating-promises` rule to ignore unhandled rejection
  void (async () => {
    await doSomething();
    resolve();
  })();
});

// Name the async wrapper to call it later
const eventEmitter = new EventEmitter();
eventEmitter.on('some-event', () => {
  const handler = async () => {
    await doSomething();
    otherSynchronousCall();
  };

  try {
    synchronousCall();
  } catch (err) {
    handleSpecificError(err);
  }

  handler().catch(handleError);
});
```

<!--/tabs-->

## When Not To Use It

If you do not use Promises in your codebase or are not concerned with possible
misuses of them outside of what the TypeScript compiler will check.

## Further Reading

- [TypeScript void function assignability](https://github.com/Microsoft/TypeScript/wiki/FAQ#why-are-functions-returning-non-void-assignable-to-function-returning-void)

## Related To

- [`no-floating-promises`](./no-floating-promises.md)

## Attributes

- Configs:
  - [x] ✅ Recommended
  - [x] 🔒 Strict
- [ ] 🔧 Fixable
- [x] 💭 Requires type information
