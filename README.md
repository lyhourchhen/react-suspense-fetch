# react-suspense-fetch

[![CI](https://img.shields.io/github/workflow/status/dai-shi/react-suspense-fetch/CI)](https://github.com/dai-shi/react-suspense-fetch/actions?query=workflow%3ACI)
[![npm](https://img.shields.io/npm/v/react-suspense-fetch)](https://www.npmjs.com/package/react-suspense-fetch)
[![size](https://img.shields.io/bundlephobia/minzip/react-suspense-fetch)](https://bundlephobia.com/result?p=react-suspense-fetch)
[![discord](https://img.shields.io/discord/627656437971288081)](https://discord.gg/MrQdmzd)

A primitive library for React Suspense Render-as-You-Fetch

## Introduction

The new [Render-as-You-Fetch](https://reactjs.org/docs/concurrent-mode-suspense.html#approach-3-render-as-you-fetch-using-suspense) pattern is mind-blowing.
So far, only Relay implemented that pattern for GraphQL.
This library aims at implementing that pattern for REST APIs.

This is an experimental library.
Here's the list of design decisions:

*   No React Hooks interface
*   No global cache
*   Primitive API for libraries

## Install

```bash
npm install react-suspense-fetch
```

## Usage

```javascript
import React, { Suspense, useState, unstable_useTransition as useTransition } from 'react';
import ReactDOM from 'react-dom';

import { createFetchStore } from 'react-suspense-fetch';

const DisplayData = ({ result, update }) => {
  const [startTransition, isPending] = useTransition();
  const onClick = () => {
    startTransition(() => {
      update('2');
    });
  };
  return (
    <div>
      <div>First Name: {result.data.first_name}</div>
      <button type="button" onClick={onClick}>Refetch user 2</button>
      {isPending && 'Pending...'}
    </div>
  );
};

const fetchFunc = async userId => (await fetch(`https://reqres.in/api/users/${userId}?delay=3`)).json();
const store = createFetchStore(fetchFunc);
store.prefetch('1');

const Main = () => {
  const [id, setId] = useState('1');
  const result = store.get(id);
  const update = (nextId) => {
    store.prefetch(nextId);
    setId(nextId);
  };
  return <DisplayData result={result} update={update} />;
};

const App = () => (
  <Suspense fallback={<span>Loading...</span>}>
    <Main />
  </Suspense>
);

ReactDOM.unstable_createRoot(document.getElementById('app')).render(<App />);
```

## API

<!-- Generated by documentation.js. Update this documentation by updating the source code. -->

### FetchStore

fetch store

`get` will throw a promise when a result is not ready.
`prefetch` will start fetching.
`evict` will remove a result.

There are three cache types:

*   WeakMap: `input` has to be an object in this case
*   Map: you need to call evict to remove from cache
*   Map with areEqual: you can specify a custom comparator

Type: {get: function (input: Input): Result, prefetch: function (input: Input): void, evict: function (input: Input): void}

#### Properties

*   `get` **function (input: Input): Result** 
*   `prefetch` **function (input: Input): void** 
*   `evict` **function (input: Input): void** 

### createFetchStore

create fetch store

#### Parameters

*   `fetchFunc` **FetchFunc\<Result, Input>** 
*   `cacheType` **({type: `"WeakMap"`} | {type: `"Map"`, areEqual: function (a: Input, b: Input): [boolean](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Boolean)?})?** 
*   `preloaded` **Iterable\<any>?** 

#### Examples

```javascript
import { createFetchStore } from 'react-suspense-fetch';

const fetchFunc = async (userId) => (await fetch(`https://reqres.in/api/users/${userId}?delay=3`)).json();
const store = createFetchStore(fetchFunc);
store.prefetch('1');
```

## Examples

The [examples](examples) folder contains working examples.
You can run one of them with

```bash
PORT=8080 npm run examples:01_minimal
```

and open <http://localhost:8080> in your web browser.

You can also try them in codesandbox.io:
[01](https://codesandbox.io/s/github/dai-shi/react-suspense-fetch/tree/master/examples/01\_minimal)
[02](https://codesandbox.io/s/github/dai-shi/react-suspense-fetch/tree/master/examples/02\_typescript)
[03](https://codesandbox.io/s/github/dai-shi/react-suspense-fetch/tree/master/examples/03\_props)
[04](https://codesandbox.io/s/github/dai-shi/react-suspense-fetch/tree/master/examples/04\_auth)
[05](https://codesandbox.io/s/github/dai-shi/react-suspense-fetch/tree/master/examples/05\_todolist)
[06](https://codesandbox.io/s/github/dai-shi/react-suspense-fetch/tree/master/examples/06\_reactlazy)
[07](https://codesandbox.io/s/github/dai-shi/react-suspense-fetch/tree/master/examples/07\_wasm)

## Blogs

*   [Diving Into React Suspense Render-as-You-Fetch for REST APIs](https://blog.axlight.com/posts/diving-into-react-suspense-render-as-you-fetch-for-rest-apis/)
