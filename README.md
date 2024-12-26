The goal of this library is:

- server defined query invalidations
- single round trip invalidations
- type safety
- all with trpc + react-query

### On the frontend

wrap your query client in `TRPCInvalidator`

```typescript
// todo: example
```

This will:

- update your query client to listen for responses on mutations
  and update the query cache when invalidations are provided
- if `lazyInvalidation` is not disabled, it will include data in every request that
  allows the backend to only run queries if they are needed for the current page

### On the server

```typescript
// entrypoint-file-todo.ts
import { createBackendCaller } from "trpc-query-invalidator";
export const withInvalidation = createInvalidator<typeof appRouter>();
```

`createBackendCaller` returns a function used to describe the queries that should be revalidated. The arguments
passed to the query called from the singleton will be inferred as the query key of the associated data the function outputs

> Note:
> Any errors thrown by a query used in revalidation will not affect independent queries that were expected in the same response. When an
> error on a revalidated query is encountered, we alert the `QueryClient` instance that the query is error'd

> Note: `withInvalidation` enhances the response data you return in TRPC queries to include the data `TRPCInvalidator` will need to update the query cache

Because the TRPC + react-query integration automatically generates query keys through the queries input, we can generate this query key on the server
provided the inputs to the query.
ex.

```typescript
const withInvalidation = createInvalidator<typeof appRouter>({
  ...options,
});

createPost: procedure.mutation(() => {
  //...

  withInvalidation({
    // normal response data, or omitted if nothing is being returned
    data: ...,
    // all queries that have been defined on the router are available
    getPosts: [...argument list],
  });
});
```

you may directly pass the revalidated queries data if you already have access to the result in the mutations scope. You must manually provide the query key in this case, since we are no longer to able to automatically determine the data's query key without the argument to the query fn

```typescript
const withInvalidation = createInvalidator<typeof appRouter>()


createPost: procedure.mutation(() => {
  //...

  withInvalidation(
    {
      getPosts: {
        data: [{...}, {...}, {...}],
        queryKey: []
      }
    }
  )
})

```
