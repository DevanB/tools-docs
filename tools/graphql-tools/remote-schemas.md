---
title: Remote schemas
order: 310
description: Work with remote GraphQL endpoints
---

It can be valuable to be able to treat remote GraphQL endpoints as if they were local executable schemas. This is especially useful for [schema stitching](./schema-stitching.html), but there may be other use cases.

## API

<h3 id="makeRemoteExecutableSchema" title="makeRemoteExecutableSchema">
  makeRemoteExecutableSchema({ schema: GraphQLSchema, fetcher: Fetcher }): GraphQLSchema
</h3>

Given a GraphQL schema (can be a non-executable client schema made by `buildClientSchema`) and a [Fetcher](#Fetcher), produce a GraphQL Schema that routes all requests to the fetcher.

<h3 id="introspectSchema" title="introspectSchema">
  introspectSchema(
    fetcher: Fetcher,
    context?: {[key: string]: any}
): Promise&lt;GraphQLSchema&gt;</h3>

Use `fetcher` to build a client schema using introspection query. For easy of use of `makeRemoteExecutableSchema`. Provides a *client* schema, so a non-executable schema. Accepts optional second argument `context`, which is passed to the fetcher.

<h3 id="Fetcher" title="Fetcher">
  Fetcher
</h3>

```js
type Fetcher = (
  operation: {
    query: string;
    operationName?: string;
    variables?: { [key: string]: any };
    context?: { [key: string]: any };
  },
) => Promise<ExecutionResult>;
```

#### Usage with [apollo-link](https://github.com/apollographql/apollo-link)

Basic usage

```js
import { HttpLink, makePromise, execute } from 'apollo-link';

const link = new HttpLink({ uri: 'http://api.githunt.com/graphql' });
const fetcher = (operation) => makePromise(execute(link, operation));
const schema = makeRemoteExecutableSchema({
  schema: await introspectSchema(fetcher),
  fetcher,
});
```

Authentication headers from context

```js
import {
  ApolloLink,
  HttpLink,
  SetContextLink,
  makePromise,
  execute,
} from 'apollo-link';

const link = ApolloLink.from([
  new SetContextLink((context) => ({
    ...context,
    headers: {
      'Authentication': `Bearer ${context.authKey}`,
    },
  }),
  new HttpLink({ uri: 'http://api.githunt.com/graphql' }),
]);
const fetcher = (operation) => makePromise(execute(link, operation));
const schema = makeRemoteExecutableSchema({
  schema: await introspectSchema(fetcher),
  fetcher,
});
```

#### Usage with [apollo-fetch](https://github.com/apollographql/apollo-fetch)

Basic usage

```js
import { createApolloFetch } from 'apollo-fetch';

const fetcher = createApolloFetch({ uri: 'http://api.githunt.com/graphql'});
const schema = makeRemoteExecutableSchema({
  schema: await introspectSchema(fetcher),
  fetcher,
});
```

Authentication headers from context

```js
const fetcher = createApolloFetch({ uri: 'http://api.githunt.com/graphql'});
fetcher.use({ request, option }, next) => {
  if (!options.headers) {
    options.headers = {};
  }
  options.headers['Authorization'] = `Bearer ${request.context.authKey}`;

  next();
});
const schema = makeRemoteExecutableSchema({
  schema: await introspectSchema(fetcher),
  fetcher,
});
```

#### Usage with a generic HTTP client (like node-fetch)

Basic usage

```js
import fetch from 'node-fetch';

const fetcher = async ({ query, variables, operationName, context }) => {
  const fetchResult = fetch('http://api.githunt.com/graphql', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({ query, variables, operationName })
  });
  return fetchResult.json();
};
const schema = makeRemoteExecutableSchema({
  schema: await introspectSchema(fetcher),
  fetcher,
});
```

Authentication headers from context


```js
import fetch from 'node-fetch';

const fetcher = async ({ query, variables, operationName, context }) => {
  const fetchResult = fetch('http://api.githunt.com/graphql', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authentication': `Bearer ${context.authKey}`,
    },
    body: JSON.stringify({ query, variables, operationName })
  });
  return fetchResult.json();
};
const schema = makeRemoteExecutableSchema({
  schema: await introspectSchema(fetcher),
  fetcher,
});
```
