# babel-plugin-spock

> Structured JS test cases, inspired by
> [Spock Framework](http://spockframework.org)

[![build status](https://img.shields.io/travis/jeysal/babel-plugin-spock/master.svg?style=flat-square)](https://travis-ci.org/jeysal/babel-plugin-spock)
[![AppVeyor build status](https://img.shields.io/appveyor/ci/jeysal/babel-plugin-spock/master.svg?style=flat-square&label=windows+build)](https://ci.appveyor.com/project/jeysal/babel-plugin-spock)
[![code coverage](https://img.shields.io/codecov/c/github/jeysal/babel-plugin-spock/master.svg?style=flat-square)](https://codecov.io/gh/jeysal/babel-plugin-spock)

[![npm package](https://img.shields.io/npm/v/babel-plugin-spock.svg?style=flat-square)](https://www.npmjs.com/package/babel-plugin-spock)
[![license](https://img.shields.io/github/license/jeysal/babel-plugin-spock.svg?style=flat-square)](https://github.com/jeysal/babel-plugin-spock/blob/master/LICENSE)

**Note:** This module is in an early stage of development and currently provides
only a small set of features.

## Example test case

```javascript
test('basic arithmetic', () => {
  expect: {
    1 + 2 === 3;
    3 * 3 >= 4 * 4; // falsy
  }
});
```

```
Expected value to be (operator: ==):
  true
Received:
  false

  assert(3 * 3 >= 4 * 4)
           |   |    |
           |   |    16
           9   false
```

## Installation

    npm install --save-dev babel-plugin-spock power-assert

This module is a [Babel](https://babeljs.io/) plugin that needs to transform
your test sources in order to generate assertions, so your test runner will need
support for Babel. Babel integrates quite nicely into most modern test runners.
Check the documentation of your test runner for instructions on how to configure
Babel (e.g. for
[Jest](https://facebook.github.io/jest/docs/en/getting-started.html#using-babel),
[AVA](https://github.com/avajs/ava/blob/master/docs/recipes/babel.md) etc.) or consult Babel's own
[documentation](http://babeljs.io/docs/setup/).

Once Babel is set up for your test files, simply add `"babel-plugin-spock"` to
the `plugins` array in your babel configuration and you're good to go.

**Note:** The last version of this plugin that supports Babel 6 is `0.1.0`

## Usage

### Assertion blocks

Inside of a block labeled with `expect:` or `then:`, all statements will be
considered assertions and evaluated to check for truth:

```javascript
expect: {
  1 < 2;
}
```

`when`-`then` blocks can be particularly useful and expressive for code with
side effects:

```javascript
// The 'when' label here does not have a special meaning
// It is used simply to make the test more structured
when: {
  abc.setXyz(1);
}

then: {
  abc.getXyz() === 1;
}
```

Single labeled statements are also possible:

```javascript
expect: 'a' + 'b' === 'ab';
```

Note that these blocks can only contain statements that can be evaluated as
expressions. For example, an if statement would not be valid:

```javascript
// BAD
expect: {
  if (x < 1) x === 0.5;
  else x === 2;
}
```

However, you can nest an assertion block into other structures:

```javascript
// GOOD
if (x < 1) expect: x === 0.5;
else expect: x === 2;
```

If you want to perform more complicated checks, it might be helpful to look for
small helper libraries on npm. For example,
[deep-strict-equal](https://github.com/sindresorhus/deep-strict-equal) can help
perform deep equality checks on object structures. In the future, this plugin
might provide special syntax for such use cases.

Of course, you still have the option to use your native assertion library
alongside assertion blocks wherever you consider it appropriate.
Some assertion libraries may provide features on top of what this plugin supports.

### Linters

The test you will write using this plugin often employ syntax that is otherwise
uncommon in JavaScript code. For this reason, if you use a linter such as
[ESLint](https://eslint.org/), you will likely see annoying warnings all over
your tests. To work around this, most linters will give you multiple options:

* Disable the problematic rules with special annotations in your test files.
  This can be a hassle because it needs to be done for every file.
* Completely disable the rules in the configuration. This means that they will
  no longer apply to production code either.
* Create a separate config for tests that extends the base config, but disables
  the rules.

If you're using TypeScript, tsc might also complain about unused labels.
`allowUnusedLabels` can turn those warnings off.

## Configuration

You can configure this plugin using Babel's regular
[plugin configuration mechanism](https://babeljs.io/docs/plugins/#pluginpreset-options).
The following options are available:

**`powerAssert`**

The plugin can seamlessly generate assertions that produce detailed mismatch
messages to help you figure out what exactly about the assertion went wrong.
Turning this feature off could be useful if you're tests appear to run slowly or
you are experiencing other issues with your assertions.

This feature is powered by the awesome project
[power-assert](https://github.com/power-assert-js/power-assert).

type: `boolean`  
default: `true`

**`autoImport`**

The plugin transforms your assertion blocks to calls to an assert function.
Be default (`true`), this function is automatically imported from `power-assert`.
You can set this option to a string containing the name of a module
that exports an assert function as its default export
to use that module for assertions instead.

You can also set this option to `false` to disable automatic imports.
You will then have to provide a function named `assert` yourself
in your test files wherever you use assertion blocks.

type: `boolean | string`  
default: `true` (`'power-assert'`)

**`staticTruthCheck`**

The plugin can try to statically evaluate your assertion expressions at compile-time
and throw an error if they can be inferred to always be truthy or to always be falsy.
Such expressions sometimes indicate a test that does not provide any value.

Here's an example of an assertion expression that can be inferred to always be truthy:

```javascript
const x = 1;
expect: x === 1;
```

type: `boolean`  
default: `false`

**`assertFunctionName`**

The plugin will automatically define an assert import or use an existing one.
You can set this option to enforce a specific name for the generated assert calls.

type: `string`  
default: empty string (generated identifier)
