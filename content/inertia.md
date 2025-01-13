---
title: Inertia
date: 2024-12-01
description: A progressive notes for Inertia, Connecting frontend and backend.
tags:
  - inertia
  - php
  - laravel
  - javascript
  - typescript
---

# Introduction

Inertia is a javascript framework that is used to connect frontend and backend.

It is a replacement for the traditional ajax requests and it is used to make the frontend and backend communicate with each other.

It is leveraging the SPA feature of frontend frameworks like vue ,react and svelte so that you only need to load your assets once, like html and css.

Then connect to the backend like laravel and rails.

So youll have a SPA and a traditional MVC application. And you can take advantage of laravel's backend feature too.

## Inertia Cores

### Inertia Link

The most important feature of Inertia is the **<Link>** component.

It is the one that handles the requests from frontend to backend.

It intercept the network request so that instead of doing a request like going to another page
and will do a full page reload, it will send a xhr request instead and feeds JSON to the server.

It is a **component** so you can just import it. It supports vue,react, and svelte.

```vue
<script setup>

import { Link } from '@inertiajs/vue3'

// for get request(anchor tag default behavior)
<Link href="/about">About</Link>

// for post request
<Link href="/logout" method="post" as="button" type="button">Logout</Link>
// Renders as...
<button type="button">Logout</button>
</script>
```

**Props**:

| Props             | Type                                | Description                                        | Default |
| ----------------- | ----------------------------------- | -------------------------------------------------- | ------- |
| href              | string                              | The url to send the request to                     |         |
| method            | "get","post","put","patch","delete" | The method of the request                          | "get"   |
| as                | string                              | The element to render as                           | "a"     |
| data              | object,formData                     | The data that you want to inlclude in the request  |         |
| replace           | boolean                             | to whether replace the history stack or not        | false   |
| preserver-state   | boolean                             | to avoid page re-render to avoid forms input reset | false   |
| preserver-scroll  | boolean                             | to preserve the current scroll position            | false   |
| only              | string[] , object                   | to only retrieve a specific prop from server       |         |
| $page.url(vue)    | string                              | will give you the current page url                 |         |
| $page.component() | string                              | will give you the current component                |         |
| headers           | object                              | The headers of the request                         |         |

<!-- TODO: Explain more here -->

- preserver-state

  Good for filters

- Active states

  url will give you the current path

  component will give you the currently active component (good if your link have query params and good for path with sub paths)

### Manual Visit(router)

if you need to visit or manipulate the url queries without using Link component.

This is also needed for sending **FORM** request to the server.
Like **post,put,patch,delete**.

The options here have a lot of similarities with the [Link component](#inertia-link).

[Official Docs here](https://inertiajs.com/manual-visits)

**Syntax**: `rounter.<method>(url, data, options)`

other syntax:(they are the same)

```vue
<script setup>
// this is the other way .. options here is the same as above
router.visit(url, {
  method: "get",
  data: {},
  replace: false,
  preserveState: false,
  preserveScroll: false,
  only: [],
  headers: {},
  errorBag: null,
  forceFormData: false,
  onCancelToken: (cancelToken) => {},
  onCancel: () => {},
  onBefore: (visit) => {},
  onStart: (visit) => {},
  onProgress: (progress) => {},
  onSuccess: (page) => {},
  onError: (errors) => {},
  onFinish: (visit) => {},
})
</script>
```

Example:

```vue
<script setup>
import { router } from "@inertiajs/vue3"

//     method
router.get(
  "/about", //url (for post it is like action)
  // in laravel will be like: request()->search
  { search: value }, // data that you want to send to the server
  { replace: true, preserveState: true, preserveScroll: true }, // options the same with the link component
)
</script>
```

Example for post request:

```vue
<script setup>
const form = reactive({
  name: "",
  email: "",
  password: "",
})

const handleSubmit = () => {
  router.post("/about", form)
}
</script>

<template>
  <Form method="post" action="/about" @submit.prevent="handleSubmit">
    <FormSection>
      <FormLabel>Name</FormLabel>
      <FormInput name="name" type="text" v-model="form.name" />
    </FormSection>
    <FormSection>
      <FormLabel>Email</FormLabel>
      <FormInput name="email" type="email" v-model="form.email" />
    </FormSection>
    <FormSection>
      <FormLabel>Password</FormLabel>
      <FormInput name="password" type="password" v-model="form.password" />
    </FormSection>
    <FormSection>
      <FormLabel>Remember me</FormLabel>
      <FormCheckbox name="remember" v-model="form.remember" />
    </FormSection>
    <FormSection>
      <FormButton>Submit</FormButton>
    </FormSection>
  </Form>
</template>
```

### Inertia Form

[Official docs for Inertia Form](https://inertiajs.com/forms)

Since example above is very common especially when handling forms.
The **`useForm()`** helper from inertia is used to send a request to the backend.
It can also receive a validation error object from the backend
and if the backend is laravel it will automatically handle the csrf token and redirect to the proper page.

**Syntax**

```vue
<script setup>
import { useForm } from "@inertiajs/vue3"

const form = useForm({
  title: "Hello World",
  body: "This is a test",
})
</script>

<template>
  <form @submit.prevent="form.post('/posts')">
    <input type="text" name="title" v-model="form.data.title" />
    <textarea name="body" v-model="form.data.body"></textarea>
    <button type="submit">Submit</button>
  </form>
</template>
```

- `useForm`
  method needs an object(key:value pair) argument that corresponds to your input.

  When used with vue this property is reactive which can be use in v-model as shown in the example above.

  The value that is passed inside the useForm method will be the default/initial value of your input.
  When use with typescript, it will automatically infer the type base on the value.

  - `form.post(url)`
    is a method that will send the data to the backend.

  other methods are also available like:

  ```
  form.submit(method, url, options)
  form.get(url, options)
  form.post(url, options)
  form.put(url, options)
  form.patch(url, options)
  form.delete(url, options)
  ```

  The `options` is an object which is the same with [manual visit options](#manual-visitrouter)

### Debounce and Throttle

If you're implementing a search bar, where it will search as you type
In order to not hit the server too much, you can use the debounce or throttle function.

- Throttle: will only run the function **after a certain amount** of time has passed.
- Debounce: will only run the function after a **pause** in a certain amount of time has passed.

Using lodash:

```js
import { throttle, debounce } from "lodash"

const throttledFn = throttle(fn, wait)
const debouncedFn = debounce(fn, wait)
```

Example:

```vue
<script setup>
import { throttle, debounce } from "lodash"

const search = ref("")

watch(
  search,
  (newValue)=>{
    debounce((newValue) => {
    router.get(
      "/about",
      { search: newValue },
      { replace: true, preserveState: true, preserveScroll: true },
    )
  }, 300),
  }
)
</script>

<template>
  <div>
    <input type="text" v-model="search" @input="handleSearch" />
  </div>
</template>
```

### Security of SPA
