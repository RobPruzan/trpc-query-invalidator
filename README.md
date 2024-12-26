The goal of this library is:

- server defined query invalidations
- single round trip invalidations supported
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

`createBackendCaller` returns a function used to describe the queries that should be revalidated. You can specify part of, or an entire query key to be revalidated.

If only part of the query key is provided, all query keys that start with the provided
query key will be revalidated

- an empty list will invalidate all queries

> Note: if the entire query key is specified, the revalidation will happen in a single round trip

> Note:
> Any errors thrown by a query used in revalidation will not affect independent queries that were expected in the same response. When an
> error on a revalidated query is encountered, we alert the `QueryClient` instance that the query is error'd

```typescript
const withInvalidation = createInvalidator<typeof appRouter>({
  ...options,
});

createPost: procedure.mutation(() => {
  //...

  return withInvalidation({
    // normal response data, or omitted if nothing is being returned
    data: ...,
    // all queries that have been defined on the router are available
    getPosts: [... partial query key],
  });
});
```

you may directly pass the revalidated queries data if you already have access to the result in the mutations scope. You must manually provide the query key in this case, since we are no longer to able to automatically determine the data's query key without the argument to the query fn

```typescript
const withInvalidation = createInvalidator<typeof appRouter>()


createPost: procedure.mutation(() => {
  //...

  return withInvalidation(
    {
      getPosts: {
        data: [{...}, {...}, {...}],
        queryKey: []
      }
    }
  )
})

```
