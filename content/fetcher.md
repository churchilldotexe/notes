---
title: "Fetcher"
date: 2024-10-15
description: "A JSON Fetch utility with schema validation and enforce error handling."
tags:
  - fetch
  - javascript
  - typescript
---

A simple JSON fetch utility that validates the response from your API that also returns an error to make sure that error is being handled properly. It was tested with _Vitest_ for the unit testing for its use cases and integration test with **JSON place holder api**.

### USAGE

The usage is pretty simple, what you really just need is `zod` to validate the schema that you expect from the api.

```bash
pnpm add zod
```

- You can copy the fetcher code from this repository
  [fetcher code](https://github.com/churchilldotexe/components-hooks-utils/blob/main/src/lib/utils/fetcher.ts)

you can put it in your `lib` directory or your `utils`
then you can use it like this:

We are going to use [JSON placeholder](https://jsonplaceholder.typicode.com/) for this [endpoint](https://jsonplaceholder.typicode.com/todos/1)

1. define your zod schema:

   ```typescript
   const jsonPlaceHolderSchema = z.object({
     userId: z.number(),
     id: z.number(),
     title: z.string().min(1),
     completed: z.boolean(),
   })
   ```

   Here we created a zod object of userId,id,title,completed base on the response of the endpoint.

2. Use the fetcher

   ```typescript
   const url = "https://jsonplaceholder.typicode.com/todos/1"

   const returnedData = await fetcher(url, jsonPlaceHolderSchema)
   ```

   the `fetcher` accepts three arguments.

   - Url - your api url.
   - Validator - your zod schema that you expects from your api.
   - Init - the `Request Init` or the second parameter of the fetch api where you can put the necessary headers, method, body and other init options.

3. Receiving the fetcher

   first we have to handle the error in order to access the data properly like so:

   ```typescript
   if (!result.success) {
     if (result.error instanceof ZodError) {
       throw new Error(`Validation error: ${result.error.message}`)
     } else {
       throw new Error(`Error: ${result.error.message}`)
     }
   }

   // you can safely access the data
   console.log(result.data)
   ```

It is following a pattern of zod safeParse where you have to handle the _failed success_ in order to access the parsed data. This ensures that whenever the fetcher fails you can show to the UI or log it to know what fails. It is better this way, in my opinion, because you can catch if there's a bug right away.

```typescript
type FetcherReturnTypes<T extends ZodTypeAny> =
  | {
      success: true
      data: z.TypeOf<T>
    }
  | { success: false; error: Error | ZodError<any> }

export async function fetcher<T extends z.ZodTypeAny>(
  url: string | Request | URL,
  init: RequestInit,
  responseValidator: T,
): Promise<FetcherReturnTypes<T>> {
  const response = await fetch(url, init)
    .then((res) => {
      if (!res.ok) {
        throw new Error(`${res.status}: ${res.statusText}.`)
      }

      return res
    })
    .catch((e) => {
      return e instanceof Error ? e : new Error(e)
    })

  if (response instanceof Error) return { success: false, error: response }

  const data = await response
    .json()
    .then((resData) => {
      const parsedData = responseValidator.parse(resData) as z.infer<T>
      return parsedData
    })
    .catch((e) => {
      return e instanceof Error ? e : e instanceof ZodError ? e : new Error(e)
    })

  if (data instanceof ZodError) {
    return { success: false, error: data }
  } else if (data instanceof Error) {
    return { success: false, error: data }
  }
  return { success: true, data }
}
```
