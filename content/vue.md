---
title: "All About Vue"
date: 2024-11-25
description: "A progressive notes while learning Vue starting from the basic"
tags:
  - vue
  - javaScript
  - typescript
---

# Vue starting from the basic

Vue is a js framework that is like react.
But the distinct difference between the two is how they re-render

- React is **OPT-IN** reactivity. Meaning, it does a **Global Rerender** pattern where when a state change the whole component will do a re-render(like a scan) and then react will do the **diffing** and compare it with virtual DOM and optimize the DOM tree by only update that actual DOM on which part of the component changed.
  although, it is great, because it is optimized you have to be careful with the **re-rendering** part as it is one of the reason of bottlenecks especially if you have a heavy logic and if you have a piece of code that changes its value on every re-render (like useEffect with omitted dependencies) it will cause you an infinite re-render

- Vue is **OPT-OUT** reactivity. Meaning, it is doing a **Granular Rerender** where it will only re-render the part of the code that actually change.

## Access Point

In order to connect to the Vue instance, You can either use the cdn to play around or use a package manager like npm to instal the Vue package

```bash
npm create vue@latest
```

or cdn

```html
<script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>
```

Once installed, the structure typically consist of access point. the one that connects vue to html.
like:

```html
<body class="size-full bg-white text-gray-950">
  <!-- This is where all the content/pages will be rendered -->
  <div id="app" class="contents"></div>
  <!-- This is where the vue instance is located and will be mounted -->
  <script type="module" src="/src/main.ts"></script>
</body>
```

Vue Root instance/access point or what is called **App Level**

This is where you create connection to you Top level component or layout.

This is also where you can initialize the state management library like [Pinia](#pinia) or You app level [provide](#provide)

```js
// the typical root instance

import "./assets/main.css" // you css

import { createApp } from "vue"
import App from "./App.vue"
import router from "./router"
import { createPinia } from "pinia"

import { ref } from "vue"

// sample ref for app level provide
const foo = ref("hi i am foo")

// creating pinia instance
const pinia = createPinia()

// creating the app instance
const app = createApp(App)

// so you can use the SPA router
app.use(router)

// global dependency injection
app.provide("about", about)

// so you can use pinia
app.use(pinia)

// this corresponds to the div where all the content will be rendered
app.mount("#app")
```

---

## Syntaxes and concepts

### V-model

- #### Vue Implementation

  - [V-model Documentation.l](https://vuejs.org/guide/components/v-model.html#multiple-v-model-bindings) Where you can find other uses of v-model like multiple v-model binding and handling it.

  - ##### Definition

    - Is a two way binding directive from vue.
    - It is being passed as an **attributes**.

      - By two way means, it is reactive if either of the reference changes state the other one will change as well.

    ```vue
    <div>
       <input type="text" v-model="greeting"/>
       <p>{{ greeting }}</p>
    </div>

    <script>
    Vue.createApp({
      data() {
        return {
          greeting: "hello, world",
        }
      },
    })
    </script>
    ```

    Whats happening here is the initial state of input now is **"hello, world"** but when you start typing in the input the `p` tag's _hello world_ will also change base on the value that you typed likewise if you manually/dynamically change the `greeting` directly in the `data function` the input's `value` and the `p` tag will also change

  - ##### V-model the under the hood

    - input

    ```vue
    <script>
    template:`
       <input :value="greeting" @input="greeting = $event.target.value" />
       <p>{{ greeting }}</p>
    `,

    data() {
       return {
          greeting: "hello, world",
       };
    },
    </script>
    ```

    This is the longer version of v-model, where we have to define and bind the `value` and setup a `v-on:input` or `@input` to listen to the changes in the input and assign it to `greeting` using `$event`.
    [You can find the event handling in this section](#vues-v-binding-and-v-on)

  - ##### V-model for parent-child component syncing

    - v-model can also be use to create a communicate between parent and its child component.
    - [this is the implementation without using v-model](#vue-v-on-and-event-hanling)

    - Usage:

      - define the v-model in the parent component

      ```js
      import AssignmentTags from "./AssignmentTags.js"

      export default {
        components: {
          AssignmentTags,
        },
        template: `
            <assignment-tags v-model:currentTag="currentTag" :initial-tags="assignments.map(a=>a.tag)" ></assignment-tags>
         `,
        props: {
          assignments: Array,
        },
        data() {
          return {
            currentTag: "all",
          }
        },
        computed: {
          filteredAssignments() {
            if (this.currentTag === "all") {
              return this.assignments
            }

            return this.assignments.filter((a) => a.tag === this.currentTag)
          },
        },
      }
      ```

      You can also alias it like the one above. `foo` is the alias

      - using it in the child component

      ```js
      export default {
        template: `
               <div 
                  class="flex gap-2 text-xs"
                  >
                     <button 
                     @click="$emit('update:currentTag', tag )"
                     v-for="tag in tags" 
                     class="px-1 py-px border border-slate-600 rounded"
                     :class="{'text-blue-600 ': tag === currentTag}"
                     >
                        {{ tag }}
                     </button>
               </div>
            `,
        props: {
          initialTags: Array,
          currentTag: String,
        },
        computed: {
          tags() {
            return ["all", ...new Set(this.initialTags)]
          },
        },
      }
      ```

- #### React Implementation state + event listener(onChange)

  In react, You can do the equivalent by first creating a **state hook** and connect it with **event listener** specifically, \*_Onchange_  
  like so:

  ```tsx
  import React, { useState } from "react"

  function App() {
    // Declare a state variable `greeting` with an initial value of "hello, world"
    const [greeting, setGreeting] = useState("hello, world")

    // Handle the change event for the input field
    const handleInputChange = (event) => {
      setGreeting(event.target.value) // Update the greeting state with the new input value
    }

    return (
      <div>
        <input
          type="text"
          value={greeting} // Bind the input field's value to the `greeting` state
          onChange={handleInputChange} // Handle input change
        />
        <p>{greeting}</p> {/* Display the current value of greeting */}
      </div>
    )
  }

  export default App
  ```

- Key difference

  - Vue's implementation is two-way data binding so when the value change the model **greeting** will also change or vice versa
  - React's implementation is doing it in a controlled component, where you bind the input value and onchange handler to the state. The only way to change the state is using the **setGreeting** and must not do it directly to the **greeting** due to react's principle of immutability

### binding and event handlers

- #### Vue's v-binding and v-on

  `v-bind:` - is a way to bind your state to a html attributes to make it more dynamic.

  - `:` -shorthand

  - `v-bind:class` or `:class` - usage

  `v-on:` - [explained in the event handling section](#event-handling-and-passing-data-from-child-to-parent)

  example:

  ```vue
  <section>
        <button type="button" v-on:click="toggle">
          change the color below
        </button>
        <p v-bind:class="active? text : 'text-blue-500'">im red</p>
  
  
        <button type="button"  @:click="toggle">
        This is the shorthand
        </button>
        <p :class="active? text : 'text-blue-500'">Im also red</p>
  </section>
  <script>
  Vue.createApp({
    data() {
      return {
        greeting: "hello, world",
        text: "text-red-500",
        active: false,
      }
    },
    methods: {
      toggle() {
        this.active = !this.active
      },
    },
  }).mount("#app")
  </script>
  ```

- #### React `{}` and `on` event handlers

  `{}` - in react this directive it also used to bind the attributes to be dynamic. in the this braces is like switching to JS world
  `on` - react use `on` + event handlers to bind js to html event handler

  Example:

  ```tsx
  import React, { useState } from "react"

  const App = () => {
    const [active, setActive] = useState(false)

    const toggle = () => {
      setActive(!active)
    }

    return (
      <section>
        {/* Button to toggle the color */}
        <button type="button" onClick={toggle}>
          Change the color below
        </button>
        <p className={active ? "text-red-500" : "text-blue-500"}>I'm red</p>

        {/* no Shorthand */}
      </section>
    )
  }

  export default App
  ```

### Looping , conditionals and Computed properties

- #### Vue `v-for` , `v-if`,`v-show`, `:key` and `computed()`

  - **`v-for`** is vue way of looping and iterating over an array or objects.

    - implemented through attributes
    - `<div v-for="foo in foos" > {{ foo.bar }} </div>`

    -**`:key`** to avoid bug on when the iterated value changes in a way that it is being destroy and recreated.

    - it is a special attribute for vue to keep track of changes in the itterated value.

  - **`v-show`** is a vue helper that is used to conditionally render an element base on its truthy-ness..

    - under the hood it uses display: none to hide the element.
    - this is useful for performance since you're not destrying and recreating the element.

      - **`v-show vs v-if`** - as stated above, v-show under the hood uses display: none to avoid recreating elements.
        while `v-if` is the opposite, it completely remove the element in the DOM.

  - **`v-if`** conditional syntax of vue. Renders an element base of its truthy-ness.. (the same with js/ts `if`)

  > [!WARNING]
  >
  > Do not use `v-for` with `v-if` on the same element
  > `v-if` will always have a higher priority than `vi-for`
  > thus if you use the value/itterated from `v-for` to `vi-if` it will throw an error
  > because `v-if` was being checked first than `v-for`

  [docs regarding v-for and v-if](https://vuejs.org/style-guide/rules-essential.html#avoid-v-if-with-v-for)

  - **`v-else-if`** conditional syntax of vue. For nesting more conditionals.

    - **Restriction:** must be used together with `v-if` or `v-else-if`.

  - **`v-else`** vue syntax for `else`. Where it renders as a fallback of `v-if` when the `v-if` condition wasnt met this can be used as a fallback/default.

    - does not express expression
    - **Restriction:** previous sibling must be either `v-if` or `vi-elseif`

  ```vue
  <div v-if="condition1">Condition 1</div>
  <div v-else-if="condition2">Condition 2</div>
  <div v-else>Fallback</div>
  ```

  - **`computed Properties`** are a special kind of property that is only computed once and then cached. It needs `data properties` and **automatically** keeps track of it and cached the data.

    - It only changes when the data from the `data properties` changes. Which makes it performant.
    - as long as the data properties are not changed it will not re-compute, _no matter how many times how many times it is being used_.

  - **`useMemo`** - React uses `useMemo` to memoize the value of the function. It is the same with computed properties from view , it achieves the same objective that is, but it they also have differences.

    - **explicit** dependency array is required in order to memoize the value correctly if not provided correctly, it will make the value's stale. While computed Properties, automatically tracks and cache the value and will only change the value if its dependency changes.

  Example:

  ```vue
  <section class="space-y-4">
      <fieldset v-show="inProgressAssignments.length">
        <legend class="text-2xl font-bold">Not Accomplished</legend>
  
        <ul v-for="assignment in inProgressAssignments">
          <label v-show="!assignment.completed" :key="assignment.id">
            {{ assignment.name }}
            <input type="checkbox" v-model="assignment.completed" />
          </label>
        </ul>
      </fieldset>
  
      <fieldset v-show="accomplishedAssignments.length">
        <legend class="text-2xl font-bold">Accomplished</legend>
  
        <ul v-for="assignment in accomplishedAssignments ">
          <label :for="assignment.name" :key="assignment.id">
            {{ assignment.name }}
            <input
              type="checkbox"
              :id="assignment.name"
              v-model="assignment.completed"
            />
          </label>
        </ul>
      </fieldset>
    </section>
  <script>
  Vue.createApp({
    data() {
        assignments: [
          { name: "Finish project", completed: false, id: 1 },
          { name: "Read documentation", completed: false, id: 2 },
          { name: "Write homework", completed: false, id: 3 },
        ],
      };
    },

    computed: {
      accomplishedAssignments() {
        return this.assignments.filter((assignment) => assignment.completed);
      },

      inProgressAssignments() {
        return this.assignments.filter((assignment) => !assignment.completed);
      },
    },
  }).mount("#app");
  </script>
  ```

- #### React `map`, `ternary operator`, `key`, `show/hide` and `useMemo`

  - **`map`** - React uses `map` to iterate over an array and return a new array that is now being used to diffing and then render

  - **`key`** - React uses `key` to keep track of the changes in the iterated value.

    - it is the same with vue.

  - `**`show/hide`**` - react have different implementations for `show/hide` an element although it also relies truthy-ness.

    - It uses ternary operator and also it is being implemented inline inside the attribute like style and className. although, they can achieve the same thing like display:none

    - since it uses ternary operator, it can also be used to remove the element from the DOM directly but this implementation is not being implemented inside the attributes

- **`Ternary operator`** - React uses the ternary operator instead of `if`, `else`.

  - and in order todo it the template must opt in to js/ts using `{}`

  example:

  ```tsx
  {
    condition1 ? <div>Condition 1</div> : condition2 ? <div>Condition 2</div> : <div>Fallback</div>
  }
  ```

  ```tsx
  import React, { useMemo } from "react"

  const App = () => {
    const assignments = [
      { name: "Finish project", completed: false },
      { name: "Read documentation", completed: true },
    ]

    // although in this context it is unnecessary to use useMemo
    // but for the sake of comparison, it is being used here
    const completedAssignments = useMemo(
      () => assignments.filter((a) => a.completed),
      [assignments],
    )

    const inProgressAssignments = useMemo(
      () => assignments.filter((a) => !a.completed),
      [assignments],
    )

    return (
      <section>
        {inProgressAssignments.length > 0 && (
          <fieldset>
            <legend>Not Accomplished</legend>
            {inProgressAssignments.map((assignment) => (
              <label key={assignment.name}>
                {assignment.name}
                <input type="checkbox" defaultChecked={assignment.completed} />
              </label>
            ))}
          </fieldset>
        )}

        {completedAssignments.length > 0 && (
          <fieldset>
            <legend>Accomplished</legend>
            {completedAssignments.map((assignment) => (
              <label key={assignment.name}>
                {assignment.name}
                <input type="checkbox" defaultChecked={assignment.completed} />
              </label>
            ))}
          </fieldset>
        )}
      </section>
    )
  }

  export default App
  ```

---

## Components, events and props

### Built-in Components

Vue Integrated Components that can be used without importing them.
It is a helper component that is modified to its speicific needs.

> ⚠️
> Transition and transition group will not work if using Display:none
> But will work with [v-show](#vue-v-for-v-ifv-show-key-and-computed)

- #### Transition

  [in depth explanation here](https://vuejs.org/guide/built-ins/transition.html#transition)

  Is helper that is use to animate **1** Children Element.

  [available attributes here](https://vuejs.org/api/built-in-components.html#transition)

  Example:

  ```vue
  <template>
    <transition
      onBeforeEnter="opacity-0"
      onAfterEnter="opacity-100"
      onBeforeLeave="opacity-100"
      onAfterLeave="opacity-0"
    >
      <div v-show="show">
        <p>Im transitioning</p>
      </div>
    </transition>
  </template>
  ```

- #### Transition Group

  Is a helper that is the same with [transition](#transition) but it can be used with multiple elements.

  [available attributes here](https://vuejs.org/api/built-in-components.html#transitiongroup)
  Example:

  ```vue
  <template>
    <transition-group tag="ul" name="list">
      <li v-for="item in items" :key="item.id">
        {{ item.text }}
      </li>
    </transition-group>
  </template>
  ```

- #### Teleport

  Is a helper to render the component in the selected DOM element. (normally in body)

  It is really useful for z-index stacking context where you want it to be on top of other elements.
  If used to where it is located as long as it is a child of a element z-index is problem will likely to affect
  because it is bound to its parent element. Which makes debugging of z-index stacking context difficult.

  **Syntax**:

  - to : String (required) the selector of the element to teleport to.

  - disabled : Boolean (optional) if true, the element will be render back to where it is originally located.
    (can be dynamically activated)

  - defer: Boolean (optional) it will render only after all other elements have been mounted.

  Example:

  ```vue
  <template>
    <div id="app">
      <Teleport to="body">
        <p>I'm in the body</p>
      </Teleport>
    </div>
  </template>
  ```

- #### KeepAlive

  Is a helper to cache the elements/components of its child

  Really useful to preserve the state of [dynamic Components](#dynamic-component) and [transitions](#transition)

  By default, `component` and `v-if/v-else` components when swtiching states are not being presserved,
  Unless, you are using v-show , because of destroy/create of DOM.
  This resolves the issue of maintaining state for those elements.

  ⚠️ when used inside the `v-if/v-else` component, **1** component must be rendered to cache it..
  It wont work with multiple components.

  **Attributes**

  - include - String | RegExp : A string or regular expression that matches the **name of component**

  - exclude - String | RegExp : the opposite with include but the same matching usage.

  - max - Number | string : The maximum number of components to cache

  Example:

  Using `component`

  ```vue
  <script setup>
  import Foo from "./FooComponent.vue"
  import Bar from "./BarComponent.vue"
  import { ref } from "vue"

  const currentComponent = ref(Foo)
  </script>

  <template>
    <keep-alive exclude="Bar">
      <component :is="currentComponent" />
    </keep-alive>

    <button @click="currentComponent = Foo">Show Foo</button>
    <button @click="currentComponent = Bar">Show Bar</button>
  </template>
  ```

  Using v-if/v-else

  ```vue
  <script setup>
  import Foo from "./FooComponent.vue"
  import Bar from "./BarComponent.vue"
  import Baz from "./BazComponent.vue"
  import { ref } from "vue"

  const currentComponent = ref("FooComponent")
  </script>

  <template>
    <keep-alive include="Foo">
      <Foo v-if="currentComponent === 'FooComponent'" />
      <Bar v-else-if="currentComponent === 'BarComponent'" />
      <Baz v-if="currentComponent === 'BazComponent'" />
    </keep-alive>

    <button @click="currentComponent = 'FooComponent'">Show Foo</button>
    <button @click="currentComponent = 'BarComponent'">Show Bar</button>
    <button @click="currentComponent = 'BazComponent'">Show Baz</button>
  </template>
  ```

- #### Suspense

  [In depth guide for suspense](https://vuejs.org/guide/built-ins/suspense.html)

  [also check async components from docs](https://vuejs.org/guide/components/async.html)

  A helper that wraps a async component and run it in memory/background,
  while async component is being run a [slot#fallback](#vue-custom-components-template-slot-named-slots-and-flags)
  is being run until the async component is loaded/resolved.

  **Attributes/props**

  **Timeout** `optional string|number` in milliseconds for the suspense to wait before showing the fallback slot.

  **Suspensible** `optional boolean` to opt out of suspense component and let the child handle the loading/fallback.

  It takes two slot which is the **default** slot and the **fallback** slot.(**required**)

  **Suspense Events**

  - **`pending`** - when it deems the default slot as async and resolve it on memory
  - **`fallback`** - the loading state. will trigger when pending state triggers
  - **`resolve`** - will run default slot during this event.

  What it does is on _initial render_ it will run the default slot if it detects
  that it is async _including the children_ it will become in **pending state** and run the _fallback slot_
  once the async resolves it will now become **resolve event** and will render the default slot.

  **Usage with [transition](#transition) and [keep-alive](#keepalive)**

  It can be use with transition and keep alive state for animation and caching of the components.

  But order is important to make it work. **Suspense** must be a child of **KeepAlive** .

  **Example**

  ```vue
  <RouterView v-slot="{ Component }">
  <template v-if="Component">
    <Transition mode="out-in">
      <KeepAlive>
        <Suspense>
          <!-- main content -->
          <component :is="Component"></component>
  
          <!-- loading state -->
          <template #fallback>
            Loading...
          </template>
        </Suspense>
      </KeepAlive>
    </Transition>
  </template>
  </RouterView>
  ```

### Custom components

- #### Vue `custom components` , `template` , `slot`, `named slots` and `flags`

  - **`custom Components`** - Vue uses the syntax `components` to create reusable and custom html elements.

    - custom components must be under the `components:{}` object.
      - the **key** is the name of the component
      - the **value** can be treated the same as the `app` object (the initialized object from `Vue.createApp` )

  - **`template`** - is the where you write the html for your component.

  - it must be define under the components object and your custom component object.
    like so:

    ```vue
    const app = { components: { 'my-component': { template: '
    <div>Hello World</div>
    ' } } }
    ```

  - **`slot`**

    [full docs about slot](https://vuejs.org/guide/components/slots#renderless-components)

    - is vue's way to dynamically render content inside a component or nest it.
    - Purpose: Slots allow you to inject custom content into a component, making it highly reusable and flexible.
      Meaning you can use the component in a completely different way from one another.
    - it must be defined under the `template`
    - can be nested with another custom component
    - Usage: once defined you can use the custom component as if it was a native html element and its children will
      render.

  Example:

  ```vue
  <body class="h-full">
     <div id="app" class="contents">
        <foo class="">bar</foo>
     </div>
  </body>

  <script>
  const app = {
    components: {
      foo: {
        template: `<button @click="toggle"><slot/></button>`,
        mounted() {
          console.log("foo")
        },
        methods: {
          toggle() {
            alert("baz")
          },
        },
      },
    },
  }

  Vue.createApp(app).mount("#app")
  </script>
  ```

  - **`named slots`**

    - If you need to render multiple slots and also want to optionally render them, like for example, you're using your component in different places.

    - syntax: `<slot name="slotName" />`
    - usage: `<template #slotName></template>` or `<template v-slot:slotName></template>`

      - the `#` is the shorthand for `v-slot`

    - example:

    ```vue
    <template>
      <header>
        <h1>
          <slot name="header" />
        </h1>
      </header>
       <main>
        <slot />
      </main
      <footer>
        <div>
          <slot name="footer" />
        </div>
      </footer>
    </template>
    ```

    - Usage:

    ```vue
    <template>
      <div>
        <!-- will render header and default but not footer -->
        <my-component>
          <template #header> Header </template>
          <p>im default</p>
        </my-component>

        <!-- will render all of them -->
        <my-component>
          <template #header> Header </template>
          <p>im default</p>
          <template #footer> Footer </template>
        </my-component>
      </div>
    </template>
    ```

    - **flags**

      - Vue uses flags to control the behavior of the component.
      - to make it more composable, like if you want to conditionally render a component base of a prop.

- #### React `components`, `JSX`, and `children`

  - **`Components`** - React uses components as the building blocks for creating reusable and custom UI elements.

    - Components can be defined as **functions** or **classes**.
    - A component must return JSX, which describes the UI.

  - **`JSX`** - is the syntax extension for JavaScript that allows you to write HTML-like code within React.

    - It is used to define the structure of the component's UI.
    - JSX must be wrapped in a single parent element.

    Example of a functional component:

    ```jsx
    function MyComponent() {
      return <div>Hello World</div>
    }
    ```

    Example of a class component:

    ```jsx
    class MyComponent extends React.Component {
      render() {
        return <div>Hello World</div>
      }
    }
    ```

  - **`children`** - React uses the `children` prop to render dynamic content inside a component or to nest elements.

    - It allows passing content between opening and closing component tags.
    - Usage: Define a component that accepts `children` and use it as a wrapper.

    Example:

    ```jsx
    import React from "react"
    import ReactDOM from "react-dom"

    function Foo({ children }) {
      return <button onClick={() => alert("baz")}>{children}</button>
    }

    const App = () => (
      <div id="app">
        <Foo>bar</Foo>
      </div>
    )

    ReactDOM.render(<App />, document.getElementById("root"))
    ```

### Props

- #### Vue `props`

  - Vue uses `props` to pass data from a parent component to a child component or data outside/external to the component.
  - it is defined in a `props` object inside the component props.
  - it receives the following key-value pairs:
    - `type` - the type of the prop. **Constructor** types
    - `required` - whether the prop is required or not. **Boolean**
    - `default` - the default value of the prop. if this is defined and the prop is not passed, this default value will be used.
    - `validator` - a function that validates the prop value. It is a **Function** that returns a **Boolean**.
      - if returns **false**. Vue will throw an warning|error in the console and the prop will not work.

  Example:

  ```vue
  <script>
  export default {
    template: ` 
     <button 
        :class="{
           'bg-gray-500': variant==='muted',
           'bg-blue-500': variant==='primary'
        }"
        >
           <slot/>
     </button> 
     `,

    props: {
      variant: {
        type: String,
        default: "primary",
        validator: (value) => {
          const variants = new Set(["primary", "muted"])
          return variants.has(value)
        },
      },
    },
  }
  </script>
  ```

  Usage:

  ```vue
  <!-- // will work -->
  <button-component variant="primary">
    Primary
  </button-component>

  <!-- //will not work -->
  <button-component variant="secondary">
    Secondary
  </button-component>
  ```

  Vue error:

  ```bash
   [Vue warn]: Invalid prop: custom validator check failed for prop "variant".
   at <Foo class="px-4 py-2 rounded" variant="secondary" >
   at <App>

  ```

- #### React `props`

  - React uses `props` to pass data from a parent component to a child component.
  - Props are defined as a paramater in the component function or as properties in class components.
  - if using with typescript, validation is defined by using types if with vanila js you probably need a functon to validate the prop.

  Example:

  using javascript:

  ```jsx
  import React from "react"

  const ButtonComponent = ({ variant, children }) => {
    const variants = new Set(["primary", "muted"])

    if (!variants.has(variant)) {
      throw new Error(`Invalid variant: ${variant}`)
    }

    const variantClasses = {
      muted: "bg-gray-500",
      primary: "bg-blue-500",
    }

    return (
      <button className={`${variantClasses[variant] || variantClasses["primary"]}`}>
        {children}
      </button>
    )
  }

  export default ButtonComponent
  ```

  using typescript:

  ```tsx
  import React from "react"
  import PropTypes from "prop-types"

  type ButtonProps = {
    variant: "primary" | "muted"
    children: React.ReactNode
  }

  const ButtonComponent = ({ variant, children }) => {
    const variantClasses = {
      muted: "bg-gray-500",
      primary: "bg-blue-500",
    }

    return (
      <button className={`${variantClasses[variant] || variantClasses["primary"]}`}>
        {children}
      </button>
    )
  }

  export default ButtonComponent
  ```

  Usage:

  ```jsx
  function App() {
    return (
      <div>
        {/* will work */}
        <ButtonComponent variant="primary">Primary</ButtonComponent>

        {/* will not work */}
        <ButtonComponent variant="secondary">Muted</ButtonComponent>
      </div>
    )
  }
  ```

  Error :
  for javascript:

  (most likely you will get this error)

  ```bash
  Error: Invalid variant: secondary
  ```

  for typescript:

  ```bash
    Warning: Failed prop type: Invalid prop `variant` of type `string` supplied to `ButtonComponent`, expected `one of: primary, muted`.
    in ButtonComponent (at App.js:12)
    in App (at App.js:8)
  ```

### Event Handling and Passing data from child to parent

- #### Vue `v-on` and Event Hanling

  - `v-on`

  to attach event handlers to component or html elements.
  is for event handlers to connect the handlers and make it dynamic

  `@` -shorthand

  Usage:

  - `v-on:click` or `@click`
  - `$event` ,when used _inline_, it is the event object that can be use to access the DOM event properties and can also be use to access the data that came from $emit component instance of the child component.

  must be defined under the `methods` object.

  - inside the `methods` object, you can define the event handler the same way as you define a function.

  ```js

  template: `<button v-on:click="handleClick">Click Me</button>`,

  methods: {
    handleClick() {
      console.log('clicked!');
    },
  },

  ```

  **Event Modifiers**

  [official Docs](https://vuejs.org/guide/essentials/event-handling.html#event-modifiers)

  A helper to lessen a use of `e.preventDefault()` and `e.stopPropagation()` in event handlers.

  - **syntax** : `v-on:click.prevent="handleClick"`

    This will do preventDefault

  ⚠️ When using modifiers **sequence matters**. For example,

  `@click.self.prevent` - will only prevent the click event on the element itself, not on its children.

  `@click.prevent.self` - will include the children.

- #### **emit**

  is a special component instance that can pass an event and a payload(data) to the parent component. [more of component instance here](https://vuejs.org/api/component-instance.html#component-instance)

  by doing this, it allows communication between parent and child components very easily.

  - when used inline in the attribute.

    - **syntax**: `$emit('eventName', payload)` ,

  - when used inside the method object

    - **syntax**: `this.$emit('eventName', payload)`

  - the `eventName` (**`string`**) representing the name of the function/method/event that will be called/passed in the parent component.

  - `payload` (**`optional`**): Data passed with the event. This can be any data type—string, number, object, etc.

  - it can be one or multiple arguments. separated by a comma.

  **Accessing the event in the parent component**

  - when used inline in the attribute.

    **syntax**: `<div @eventName="foo = $event"></div>`

    - assuming _foo_ here is a variable in data property of the parent component.
    - the `$event` is the accessible data(payload) that was defined in the `$emit` component instance that came from child component.

  - when used inside the method object

    **syntax**: `<div @eventName="funcFoo"></div>`

    - assuming _funcFoo_ here is a function in methods property of the parent component

##### Take Note:

> [!NOTE]
> The `eventName` must be **`CamelCase`** and when used in the parent component, it must be **`kebab-case`**. **The same way with props and components**

example:

- in the child component

```js
 <template>
    <button @click="handleClick">Click Me</button>
 </template>

 <script>
 export default {
 methods: {
    handleClick() {
       this.$emit('custom-event', 'Hello from Child');
    },
 },
 };
 </script>
```

or directly in the html

```vue
<button @click="$emit('custom-event', 'Hello from Child')">Click Me</button>
```

- in the parent component

```vue
<template>
  <div>
    <ChildComponent @custom-event="handleCustomEvent" />
  </div>
</template>

<script>
import ChildComponent from "./ChildComponent.vue"

export default {
  components: { ChildComponent },
  methods: {
    // or can be any name other than payload.
    handleCustomEvent(payload) {
      console.log(payload) // Logs: "Hello from Child"
    },
  },
}
</script>
```

- #### React `onClick` and `props` Callback Functions

  - React uses the `on`+`<handler>` attribute to attach event handlers to components or HTML elements.

    - `<handler>` in this manner is the name of the event handler function.
      E.g. `onClick` , `onChange` , `onSubmit` etc.

    - `event` can be accessed by adding a parameter to a event handler function. It is use to access properties from the DOM.

    - It allows components to listen for user interactions and handle events dynamically.

  - **`on` + `handler` Usage:**
    - can be used inline in the attribute or can be used as a reference to a function

  ```jsx
  import React from "react"

  function Button() {
    const handleClick = () => {
      console.log("Button clicked!")
    }

    return <button onClick={handleClick}>Click Me</button>
  }

  export default Button
  ```

  or inline in the html

  ```jsx
  import React from "react"

  function Button() {
    return <button onClick={() => console.log("Button clicked!")}>Click Me</button>
  }
  ```

  - **Passing Data to Parent Using Callback Props:**

  - In React, child components can communicate with their parent components by invoking callback functions passed as props.

  - This is similar to Vue's $emit, where data is passed to the parent through an event.
    the difference is it is define from the parent to the child then the child can use it.

    - it is more of like the parent is in control.

  - Syntax: props.onEventName(data)
    - onEventName: A function passed from the parent as a prop.
    - data: Any payload (string, object, etc.) passed from the child to the parent.

  Example:

  parent component

  ```jsx
  import React from "react"
  import ChildComponent from "./ChildComponent"

  function ParentComponent() {
    const handleCustomEvent = (payload) => {
      console.log(payload) // Logs: "Hello from Child"
    }

    return (
      <div>
        <ChildComponent onCustomEvent={handleCustomEvent} />
      </div>
    )
  }
  ```

  child component

  ```jsx
  import React from "react"

  function ChildComponent({ onCustomEvent }) {
    const handleClick = () => {
      // Passing data to the parent via callback prop
      onCustomEvent("Hello from Child")
    }

    return <button onClick={handleClick}>Click Me</button>
  }

  export default ChildComponent
  ```

---

## Lifecycles

- ### How lifecycles work in Vue

  - [illustration on what/how lifecycles work in vue](https://vuejs.org/guide/essentials/lifecycle.html)
  - ![Lifecycles in Vue](https://vuejs.org/assets/lifecycle.MuZLBFAS.png)

  - Vue lifecyles are the initialization steps of a component. You can think of it as the steps needs to take to mount your component in the screen.
    By knowing the `initialization steps` of a component, you can take advantage of the `lifecycle hooks` to control the component's behavior or to do some logic before,during or after the component is mounted.

    Common Lifecycles:

    - `created` - before your code is _compiled_.

      - **Best usage**:
        - Usually the good time to do some **fetching** of your data..
        - Best time to Setup initial state of your component.
      - **Things to avoid**:
        - DOM manipulation(Component is not rendered yet, try `mounted` instead)

    - `beforeMount` - right before DOM is _rendered_.
    - `mounted` - after your component is _rendered_ in the DOM.

      - **Best usage**:
        - DOM manipulation.
          Example: (composition api)
        ```js
        // Lifecycle hook to add event listener when the component mounts
        onMounted(() => {
          console.log("Component mounted: Adding event listener")
          window.addEventListener("keydown", handleKeyDown)
        })
        ```

    - `beforeUnmount` - before your component unmounts from the DOM.

      - **Best usage**:

        - resource cleanup like remove event listeners or timers.
          Example: (composition api and following the example above)

        ```js
        // Lifecycle hook to remove event listener when the component unmounts
        onBeforeUnmount(() => {
          console.log("Component unmounted: Removing event listener")
          window.removeEventListener("keydown", handleKeyDown)
        })
        ```

    - `beforeUpdate` - _before_ the DOM updates due to reactivity changes.
    - `updated` - _after_ the DOM updates due to reactivity changes.
    - `unmounted` - after your component is removed from the DOM.

---

## About Files and Routing

- ### Vue `single file components`

  - Vue uses single file components to split your code into smaller, reusable pieces.
  - it contains the template, script and style to encapsulate them in 1 file. It is not required that all of them must be in a single file, you can omit some.

- ### Vue `routing`

  - Vue uses `routing` to navigate between different pages or views, _without a full page reload_ .
    This is the feature of SPA(Single Page Application) using javascript to intercept the traditional multiple page serving.

  - **Defining Routes**

    - Vue uses its official `vue-router` package to handle SPA routing.

    - **Usage**:

      - You can define your paths and views in the `routes` array, where it is inside the `createRouter` method.
        like so:

        ```jsx
        import { createRouter, createWebHistory } from "vue-router"

        // call the createRouter method
        const router = createRouter({
          history: createWebHistory(import.meta.env.BASE_URL),
          // here you can define the routes
          routes: [
            {
              path: "/", // the path it corresponds to like "/"  which will be like https://example.com/
              name: "home",
              component: HomeView, // the view or .vue file that you want that page to show
            },
            {
              path: "/contact",
              name: "contacts",
              // route level code-splitting
              // this generates a separate chunk (About.[hash].js) for this route
              // which is lazy-loaded when the route is visited.
              component: () => import("../views/ContactView.vue"),
            },
          ],
        })
        ```

  - **Using the router**

  - Vue uses `<RouterView />` to render the component that was defined the `routes` array.
    It acts the same way like a `<slot/>` but for the vue or the component that was declared as a view.

  - `RouterLink` is used to navigate between pages. It is a improved version of the `<a>` tag.
    Although, you can still use the `<a>` tag to navigate between pages but it is not recommended since it is against the SPA feature of vue-router(it will cause a full page reload).

    - **`to`** - the attribute that is the substitute `href`, they have the same functionality.

  Example:

  ```vue
  <template>
    <header>
      <div class="wrapper">
        <!-- this is where the navigation -->
        <nav>
          <RouterLink to="/">Home</RouterLink>
          <RouterLink to="/about">About</RouterLink>
          <RouterLink to="/contact">Contact</RouterLink>
        </nav>
      </div>
    </header>

    <!-- this is where the content will be rendered -->
    <RouterView />
  </template>
  ```

---

## 📝 Vue 3 Composition API

### Syntax and changes vs options API

#### **setup()**

is the new way to define a component and the entry point of compositon API.

It can receive two arguments:

- The `props` object. [docs here](https://vuejs.org/api/composition-api-setup.html#accessing-props)

  The `props` object that gives you access to the props passed to the component.

  > **📝 NOTE**:
  >
  > Props are reactive meaning if it is destrcutured it will not be reactive anymore.
  > to make it reactive you need to use `toRefs` or [ toRef ](#toref) turning it to a single props.

- the `context` object. [docs here](https://vuejs.org/api/composition-api-setup.html#setup-context)

  a non-reactive object that can give you access to the global context of the component.

  not reactive so it is okay to desctructure.
  Example context are `attrs` and `slots` which you can then access the same way with [global component instances](https://vuejs.org/api/component-instance.html#attrs)

  > **📝 NOTE**:
  >
  > You can destructure the context object but you musnt destructure attrs and slots since
  > they are stateful object which changes value depending on the component.
  > access them normally, `attrs.class` or `slots.default`

- **Creating and accessing reactivity**

  The setup function can return a reactive object that then can be access in the `template` or outside of setup function using the `this` directive.

  example:

  ```vue
  <script>
  import { ref, reactive, toRefs } from "vue"

  export default {
    setup() {
      const count = ref(0) // making the count variable reactive

      function increment() {
        count.value++ // to access and mutate the value of the count variable
      }

      return {
        count, // so that you can use it outside of the setup function
        increment, // same here
      }
    },
    mounted() {
      console.log(this.count) // to access the count without using the .value
    },
  }
  </script>

  <template>
    <div>
      <!-- same with this.count but this time you can directly access it. -->
      <p>Count: {{ count }}</p>
      <button @click="increment">Increment</button>
    </div>
  </template>
  ```

### script setup macro

A macro for [setup](#setup) function.

By adding **setup** in the <script> the code inside it will
automatically become a composition api function.

This means, you wont have to export default and call [setup](#setup) function.

and you wont have to declare [components](#custom-components)
so you can use it as it is or directly

```vue
<script setup>
import FooBar from "vue"
</script>
```

### Dynamic Component

Using the script setup macro tag, will give you this benefit,
**Dynamic Components**

It is a way to display a component base on a certain condition or logic.

- **Using `<component/>`**

  Renders the component base on the condition.

  It destroys/create the component when switching between them.
  Meaning, It will not preserve the state by default so if you
  have ,for example, an input and then the condition triggered
  to switch to another component and you switch back the input resets.

  ✅ Good for complex logic of dynamic components.
  Like having a lookup logic on which a component to render.
  That is in [computed](#computed) then display depending on the state of reactive.

  Example:

  ```vue
  <template>
    <component
      :is="someCondition ? FooBar : BarFoo"
      v-bind="someCondition ? { foo: 'foo' } : { bar: 'bar' }"
    />
  </template>
  ```

- **Using [ v-if/v-else ](#vue-v-for-v-ifv-show-key-and-computed) **

  The same with _dynamic component_ but different syntax
  and have a better readability

  Example:

  ```vue
  <template>
    <h2>2. v-if/v-else Rendering</h2>

    <FooBar v-if="someCondition" foo="foo" />
    <BarFoo v-else bar="bar" />
  </template>
  ```

- **Using [v-show](#vue-v-for-v-ifv-show-key-and-computed)**

  ✅ Better in terms of performance as it taking advantage of css `display:none` to hide the element.

  ✅ It can **retain state** since the component is just hidden, hence, it is not unmounted, destroyed, and recreated.

  ⚠️ If there are multiple components that needs to be conditionally rendered,
  It might cause a first page load issue since those components are being loaded as well.
  But for the most part, if you can leverage lazy loading, or even using prefetching which Renders
  the page first then render in low priority the other components, this is going to be a better approach.

  Example:

  ```vue
  <template>
    <h2>3. v-show</h2>

    <FooBar v-show="someCondition" foo="foo" />
    <BarFoo v-show="!someCondition" bar="bar" />
  </template>
  ```

### defineProps, defineEmits

⚠️ Both of these are only usable in **script setup** tag.

- **defineProps**

  Works the same with options api [props](#vue-props)

  But the syntax is more flexible.

  after `vue3.5` and under **script setup** tag,
  it is now possible to destructure props.

  And when destructured you can define a defaults there too.

  **Syntax**:

  ```js
  defineProps({ key: value });

  // with defaults for 3.5 and up
  const {key = default} = defineProps({ key: value });

  ```

  #### Typing **defineProps**

  It is also possible to set **defineProps** in purely typed way.

  **Syntax**:

  ```js
  defineProps<{ key: value }>();

  // destructured with defaults
  const {key = default} = defineProps<{ key: value }>();

   // 3.4 and below
  const props = withDefaults(defineProps<{ key: value }>(),
     {key = default});
  ```

  **Example**:

  ```vue
  <script setup>
  const props = defineProps({
    foo: String,
    bar: Number,
  })

  // works the same for 3.5 and above

  const { foo, bar } = defineProps({
    foo: String,
    bar: Number,
  })
  </script>
  ```

- **defineEmits**

  Works the same with options api version [emits](#vue-v-on-and-event-hanling)

  The syntax is also the same with **defineProps** with minor changes.
  First , you need to declare your emits/events .
  Then, you can use it the same way as options api.

  **Syntax**

  ```js
   // ** <eventName> string name of event/s
  defineEmits([<eventName>]);
  ```

- #### typing defineEmits

  Same with **defineProps** it is possible to define it as purely typed emits

  but there are two ways to do it:

  **Syntax**

  1.  ```js
         const emit = defineEmits<{
           (e: "<event>", value: string): void;
         }>();
      ```

  2.  ```js
         const emit = defineEmits<{
            <event>: [key: type] //tupple of payload
         }>();
      ```

      **event** - name of the event

      **key** - name of the payload

      **type** - type of the payload

  **Example**:

  ```vue
  <script setup>
  // declare emit for v-model bar and custom event foo
  const emit = defineEmits(["update:bar", "foo"]);

  const handleEvent = (e) => {
    emit("update:bar", e.target.value);
  };

  const handleOther = (e) => {
    emit("foo");
  };

  // purely type equivalent

  // 1.
  const emit = defineEmits<{ ( e:"update:bar", event: string ):void,
       ( e:"foo"):void
  }>();

  // 2.
  const emit = defineEmits<{
   'update:bar': [event: string],
   'foo': []
  }>();
  </script>
  ```

### Reactivity Fundamentals and core

- #### **ref**

  The recommended way to declare a variable to be **reactive**.

  good for a primitive types like `string`, `number`, `boolean` etc.

  > **📝 note**:
  >
  > ref creates a new reference to the value, so if you want a two-way binding, you need to use [toRef](#toref)

  syntax: `const <variable> = ref(<value>)`

  example: [you can check the example here](#setup)

  **Accessing `ref`**

  - inside the [ setup ](#setup) function: `<variable>.value`

    example: `count.value++` , you can directly mutate this.

  - accessing outside the function :

    - outside the `setup` function: `this.<variable>`

    - inside the `template`: `{{ <variable> }}`

    If accessing outside the `setup` function, `refs` are automatically unwrapped and you can then mutate it directly too (when inside the `<template>`)

    **Example**: ([variable used here is base on the example from syntax changes section](#syntax-and-changes-vs-options-api))

    ```vue
    <template>
      <button @click="count++">{{ count }}</button>
    </template>
    ```

    > 📝 NOTE
    >
    > mutating the _reactive ref variable_ inside the `<template>` comes with a caveat.
    > You can mutate it if it is **top level** but if the ref declaration is inside a **nested** object, you can't mutate it.

    To resolve this you can destructure that nested object to make it a top level or use [reactive](#reactive)

    example here:

    ```js
    const count = ref(1)
    const foo = { bar: ref(1) }
    ```

    ```vue
    // works
    <button @click="count++">{{ count }}</button>
    // doesn't work
    <button @click="foo.bar++">{{ foo.bar }}</button>
    ```

    ```js
    const { bar } = foo; // now it is a top level

    <button @click="bar++">{{ bar }}</button>

    ```

  - ##### Typing `ref`

    There are three ways to declare types for the `ref`.

    - automatic inference. if no types declared it will get the type of the argument and infer it.

    ```js
    const year = ref("2020") // will be type string
    ```

    - Importing the `Ref` type froom vue

    ```js
     import { ref } from 'vue'
     import type { Ref } from 'vue'

     const year: Ref<string | number> = ref('2020')

     year.value = 2020 // ok!
    ```

    - directly using Generics argument

    ```js
     import { ref } from 'vue'
     import type { Ref } from 'vue'

     const year = ref<string | number>('2020')

     year.value = 2020 // ok!
    ```

    or if the generic argument is not the same to the type that is being passed inside the ref method. it will become a union of the **type from argument and the type undefined**

    ```jsx
     const year = ref<string>(); // will be <string | undefined>
    ```

- #### Reactive

  Another way to declare a variable to be reactive.

  - good for a complex types like `object`, `array` etc.

  - if you want to have a store for your app, that can be used across the app, you can use `reactive` as a [state management](#state-management).

    this is like the `provide` and `inject` but not just fot the parent to child component props but also for the entire application that needs it.

    [it is explained here](#using-reactivereactive)

  - syntax: `const <variable> = reactive(<value>)`

  **Example**:

  ```vue
  <script setup>
  import { reactive } from "vue"

  const foo = reactive({ bar: 1 })

  // now can be mutated like so
  foo.bar++
  </script>
  ```

  - ##### **Typing Reactive**

    There are two ways to type reactive.

    1.  Implicit typing

        The value you pass inside the function will automatically be typed.

        ```typescript
        import { reactive } from "vue"

        // foo = {bar: number}
        const foo = reactive({ bar: 1 })
        ```

    2.  Explicit typing

        You can create your own type and pass it in as a generic.

        Useful when you're expecting

        ```typescript
        type Foo = { bar: string }

        // foo = {bar: string}
        const foo = reactive<Foo>({ bar: "bar" })
        const bar: Foo = reactive({ bar: "bar" })
        // these two works the same
        ```

    > Check: [ typing reactive docs](https://vuejs.org/guide/typescript/composition-api.html#typing-reactive)

- #### Computed

  [Docs for Computed](https://vuejs.org/api/reactivity-core.html#computed)

  Takes in a callback and returns a _Read only_ computed data from a reactive.

  It is the same with Options API's [computed property](#vue-v-for-v-ifv-show-key-and-computed)
  where it will change base on data but this time it is base on reactives like [ref](#ref)
  and with more additonal _features_

  **Additonal feature**

  - It can now do a _writable ref objectd_.
    What it does is it can now take two objects **set** and **get**

    **get** works the same way as read-only

    **set** will create a two-way binding to its dependency reactive.
    Where when you change the value from computed it will also change the reactive it depends to.

    **Example**:

    ```vue
    <script setup>
    import { ref, computed } from "vue"

    const count = ref(1)

    const plusOne = computed({
      get: () => count.value + 1, // Derives value
      set: (val) => (count.value = val - 1), // Custom logic on set
    })

    console.log(plusOne.value) // Output: 2
    plusOne.value = 5 // Sets count to 4 (5 - 1)
    console.log(count.value) // Output: 4
    </script>
    ```

  - ##### Typing Computed

  Like other [Reactivity Core](#reactivity-fundamentals-and-core)
  It can also infer types but note that it infers the **return value**

  It but you can also explicitly set a type for the returned value.
  You can pass it in a generic.

  **Example**: (basing the example above)

  ```vue
  <script setup>
  // inferred type: ComputedRef<number>
  const plusOne = computed({
    get: () => count.value + 1, // Derives value
    set: (val) => (count.value = val - 1), // Custom logic on set
  })

  const plusOne =
    computed <
    number >
    {
      //code here
    }
  </script>
  ```

- #### watch()

  [computed](#computed) can help you do some logic that base on the reactive value.
  But there are limitations with it. Here are few of them:

  - Need to access it in order to get the value or do the logic for you.

  - It is purely side effect free like async and DOM manipulation.

  **watch()** solves these problems.

  **Syntax**:

  ```js
  const { stop, resume, pause } = watch(source, callback, { options })

  // or

  const watcher = watch(source, callback, { options })
  //     ^ this is the same as stop. usage:
  watcher.stop()

  // can also be void
  watch(source, callback, { options })
  ```

  **source**: <reactive || reactive[]> the source that you want to watch.

  It can receive multiple sources as an array
  but it is better to use [watchEffect](#watcheffect) if it became too many sources.

  > 📝 NOTE:
  >
  > If the source is a reactive object like [props](#props) or [reactive](#reactive).
  > It is recommended to pass them as a callback so that when the object/props changes,
  > Itll be watched properly and optimized

  > 📝 NOTE:
  >
  > If the reactive is passed directly it will trigger the <deep> to keep track of the changes.
  > And you the callback wont be able to access the previous value.

  Example:

  ```js
  // watching a reactive: 1 value
  watch(count, () => );

  // watching props
  watch(()=>props, () => );

  //watching a reactive: 1 value with options
  watch(count, () => {}); // will have a deep:true imiplicitly

  // watching a reactive: multiple values
  watch([count, foo], () => );

  // watching a reactive: multiple values with options
  watch([count, foo], () => , { deep: true });
  ```

  **callback**: <function(newValue, oldValue, cleanup){}>

  the callback that will be called when the source changes.

  It can receive the new value as the first argument and the old value as the second argument.

  - newValue: the new value of the source, if the source is array. you can receive them as an array as well

  - oldValue: the old value of the source, will also be an array in the case of multiple sources.

  - cleanup: a function that will be called when the watch is about to be called.

    _cleanup_ is supported for async/propmises tasks

  **return value**: <object> the return value of the watch.

  Example:

  ```js
  watch(count, (newValue, oldValue) => {})

  // with multiple sources
  watch([count, foo], ([countNewValue, fooNewValue], [countOldValue, fooOldValue]) => {})
  ```

  **options**: <object> the configuration options that can change the behavior of the watch.

  - **deep**: <boolean> if it is true, it will keep track of the changes in the object/reactive.

  > 📝 NOTE:
  >
  > if the object is many levels deep, Performance issue might occur, especially if it is a large object.

  - **immediate**: <boolean> if it is true, it will run the watch immediately. Good for initialization

  - **flush**: <'sync' | 'post' | 'pre'> if it is true, it will trigger the watch immediately.

  - **once**: <boolean> if it is true, it will stop the watch() after the first time it is called.

  **return value**: <object> the return value of the watch.

  - **stop()**: <function> stop the watch when called.

  - **pause()**: <function> pause the watch when called.

  - **resume()**: <function> resume the watch when called.

  - **variable assignment**: the same as stop()

- ##### watchEffect()

  If the watch have so many sources, watchEffect is a better option.

  The tracking and the callback is now in one callback. All the reactive will now be automatically tracked.

  so instead of this way:

  ```js
  watch([count, foo], () => {
    console.log(count.value, foo.value)
  })
  ```

  you can do this way:

  ```js
  watchEffect(() => {
    console.log(count.value, foo.value)
  })
  ```

  **Syntax**:

  ```js
  watchEffect(callback, { options })
  ```

  **callback**: <function(Cleanup)> the callback that will be called when the source changes.

  > 📝 NOTE:
  >
  > You wont be able to access the previous value with watchEffect.

- ##### onWatcherCleanup()

  Is a helper from vue that will trigger if [watchEffect](#watcheffect) or [watch](#watch) is **about to be called** called.

  This is good if you have to ensure that you'll get the latest value of the reactive.

  This can be use for **side effects** like **DOM manipulation** or **fetching data**.

  **Things to consider**:

  - It must be in synchronous

### Reusable Reactive Code

- #### Composables

If you need to reuse your reactive code to another component `composables` is the way to go.

It is not only limited to reactivity but the whole composition API, itself.

It also not limited for reusing your logic to another component, it can also be used for refactoring and code organization.

- **creating one**:
  You just need to create a new file and export the function or logic.

  - The convention is to name the function in CamelCase with the `use` prefix.
    And the file name should be the same to the function name.
    e.g.

  ```js
  // useCount.ts
  function useCount() {}
  ```

  - should only be used inside the `setup` function or <script setup> tag. but in some cases can be used in onmounted lifecycle hooks and follow the proper [lifecycle hooks](#how-lifecycles-work-in-vue) especially when working with DOM(do a clean up).

  - since it is a normal function, it can receive arguments as well. It can receive a reactive argument, function, or just a value.

    > use `toValue` to normalize the argument value. [more info here](#tovalue)
    > and pair it with `watchEffect` .

  Example:

  ```js
  // count.js
  import { ref, reactive } from "vue"

  export function useCount() {
    const count = ref(0)

    function increment() {
      count.value++
    }

    return {
      count,
      increment,
    }
  }
  ```

  then to use it in another component:

  ```js
  import { useCount } from "./count"

  export default {
    setup() {
      const { count, increment } = useCount()

      return {
        count,
        increment,
      }
    },
  }
  ```

  or if with setup macro:

  ```vue
  <script setup>
  import { useCount } from "./count"

  const { count, increment } = useCount()
  </script>
  ```

- #### `Mixins`

  - although not recommended in composition API(vue3), mixins is another way to reuse reactive code.
    [docs fot mixins](https://vuejs.org/api/options-composition.html#mixins)

### Provide/Inject and State Management

When dealing with components especially with reusable one and most of the time you need to pass props to the child. Worst case scenario is you have to pass it multiple levels deep.

[props is explained here](#props)

- #### `Provide/Inject`

  To resolve prop drilling, We can use `provide` and `inject`.
  If you need to pass the data across your app, try [state management](#state-management) or [Pinia](#pinia) .

  - ##### `provide`

    Is a function where you can define the **data** that you want to pass to the children and its descendant.

    The data that is passed here as an argument _can only be access to its children and its descendant._

    This Can only be used inside <script setup> tag or in the `setup` function.

    **Syntax**:

    ```js
      provide(key, value)`
    ```

    - key: <string> the key is the data name, the only you call to access the data.
    - value:<payload> your payload/data that referenced by the key.

    Example:

    ```vue
    <script setup>
    import { provide } from "vue"

    provide("foo", 1)
    </script>
    ```

    **Using Provide in the APP LEVEL**

    If you want to have a data that can be accessed in all components, you can define a provide in the [access point](#access-point) by using `app.provide`

    **Example**:

    ```js
    // main.js
    import { createApp } from "vue"
    import App from "./App.vue"
    import router from "./router"
    import { createPinia } from "pinia"

    import { ref } from "vue"

    // the example
    const foo = ref("hi i am foo")

    // init codes here

    // app level provide
    app.provide("foo", about)
    app.mount("#app")
    ```

    which now then avaiable to all components. You can access it using [inject](#inject)

  - #### `Inject`

    Is a function that is used to access the data that was passed from the parent.

    If the data is ref, you can use it as is , this is intentional to retain reactivity.

    **Example**:

    ```vue
    <script setup>
    import { inject } from "vue"

    const foo = inject("foo")

    console.log(foo) // 1
    </script>
    ```

- #### State Management

  If you need to pass data not just in children but across the entire app, you can use state managers.
  It can store the data of your component normally in the store folrder.

  **Naming Convention:**

  The same with [composables](#composables) ,
  where you can name the file in PascalCase and
  the function must have a prefix **use** and a suffix **Store**

  e.g. ` useCountStore.ts`

  - ##### **Using [Reactive](#reactive)**

    The simplest and out of the box way to use state management.

    It is good for simple project but not for large project.
    because it has limited features and when the project complexity grows, it becomes hard to reason about.

    **Example**:

    ```js
    // CounterStore.ts
    import { reactive } from "vue"

    export function useCounterStore() {
      // state
      const count = reactive({ value: 0 })

      // methods or actions
      function increment() {
        count.value++
      }

      // getters or computed
      const doubleCount = computed(() => count.value * 2)

      // need to return it
      return {
        count,
        increment,
        doubleCount,
      }
    }
    ```

    then to use it in another component:

    ```vue
    import { useCounterStore } from "./CounterStore";

    <script setup>
    import { useCounterStore } from "./CounterStore"

    const count = useCounterStore()
    </script>

    <template>
      <div>
        <p>Count: {{ count.count.value }}</p>
        <button @click="count.increment">Increment</button>
        <p>Double Count: {{ count.doubleCount }}</p>
      </div>
    </template>
    ```

  - ##### **Pinia**

    ```bash
    npm install pinia
    ```

    Pinia is a state management library for Vue.js.

    It is a store that can be used across the app, and it is also reactive. Just like the simple store but it can handle complex logics and can also access `useRoute` and `inject`(for Setup Store)

    Just like vue it has two way so using and defining the store. Option store which is almost the same with vue's option API and Setup store which is the same with composition API
    The difference between the two is

    It is a good alternative to Vuex.

    [Docs for installation to get started](https://pinia.vuejs.org/introduction.html#installation)

    or [docs here](https://pinia.vuejs.org/core-concepts/)

    - options store have built ins like $state and $patch to do a batch store update.

    while

    - Setup store has aaccess to composition API like `ref` and `reactive`, just like the way you write composition API and can access _global provided_ properties(inject,useRoute)

      - Regarding , **[inject](#inject)**, You can only access the data that was **provided at the APP level**
        meaning, if the **[provide](#provide)** was defined on another parent component, you cant access it.

        You cant also return it (the variable that catched inject) as they dont belong in the the defineStore
        but you can create a **two-way binding** with it using [toRef](#toref) and note ❌ ref(will not have two-way binding, [more info here](#ref))

        **Here is what i mean:**

        The root or the Access point

        ```js
        //main.js
        import { createApp } from "vue"
        import App from "./App.vue"
        import router from "./router"
        import { createPinia } from "pinia"

        import { ref } from "vue"

        // the example
        const foo = ref("hi i am foo")

        const pinia = createPinia()
        const app = createApp(App)

        app.use(router)

        // app level provide
        app.provide("foo", about)
        app.use(pinia)
        app.mount("#app")
        ```

        Defining setup store

        ```js
        // SampleStore.ts
        import { defineStore } from "pinia"
        import { ref } from "vue"

        export const useSampleStore = defineStore("sample", () => {
          // state
          const accessFoo = inject("foo") // the same key as app level provide

          // methods or actions
          function increment() {
            accessFoo.value++
          }

          // getters or computed
          const doubleCount = computed(() => accessFoo.value * 2)

          // need to return it
          return {
            accessFoo,
            increment,
            doubleCount,
          }
        })
        ```

        ```vue
        // view or component that have access to pinia store
        <script setup lang="ts">
        import { useSampleStore } from "@/stores/SampleStore"

        const sample = useSampleStore()

        setTimeout(() => {
          sample.accessFoo = "im baz now"
        }, 2000)
        </script>

        <template>
          <div>
            <p>{{ sample.accessFoo }}</p>
          </div>
        </template>
        ```

    **Example**:

    ```js
    // main.js
    import { createApp } from "vue"
    import { createPinia } from "pinia"
    import App from "./App.vue"

    const pinia = createPinia()
    const app = createApp(App)

    app.use(pinia)
    app.mount("#app")
    ```

    Using Setup store.

    ```js
    // CounterStore.ts
    import { defineStore } from "pinia"

    export const useCounterStore = defineStore("counter", () => {
      // state
      const count = ref(0)

      // methods or actions
      function increment() {
        count.value++
      }

      // getters or computed
      const doubleCount = computed(() => count.value * 2)

      // need to return it
      return {
        count,
        increment,
        doubleCount,
      }
    })
    ```

    then to use it in another component:

    ```vue
    <script setup>
    import { useCounterStore } from "./CounterStore"

    const count = useCounterStore()
    </script>

    <template>
      <div>
        <p>Count: {{ count.count.value }}</p>
        <button @click="count.increment">Increment</button>
        <p>Double Count: {{ count.doubleCount }}</p>
      </div>
    </template>
    ```

### Utility Functions and Helpers

- #### toRef

  is a utility helper that can transform the value into a ref.
  Its general use if to create a reference to a reactive object or value making it reactive.
  Since, it creates a reference, it is now a **two way binding**.

  - core concept:

    - if the value passed is a ref, it will return existing refs(with two way binding).

      example:

      ```js
      const foo = ref(1)

      // have two way binding
      const bar = toRef(foo)

      bar.value++
      console.log(foo.value) // 2 and vice versa
      ```

    - if the value passed is a prop, it will create a read only ref of that prop.

    - if the value passed is a raw/normal value, it will create a ref of that value similar to [ref](#ref) and still make it reactive but a new ref(no two way binding.)

      example:

      ```js
      const foo = 1

      // bar will be reactive but it acts like (const bar = ref(foo))
      const bar = toRef(foo) // will create a new ref

      bar.value++
      console.log(foo) // 1 ; no reference to foo and vice versa
      ```

  works well with [reactive](#reactive) especially when destructuring reactive objects.
  creating a two way binding with `toRef`.

  Example:

  ```js
  import { ref, reactive, toRef } from "vue"

  const foo = reactive({ bar: 1 })

  const bar = toRef(foo, "bar")

  bar.value++
  console.log(foo.bar) // 2 and vice versa

  const bar = ref(foo.bar)

  bar.value++
  console.log(foo.bar) // 1 , no reference to foo
  ```

- #### toValue

  Is a utility helper that can normalize and returned value.
  It is best to use in the [composables](#composables) functions.

  - vs [toRef](#toref)

    - toValue unwraps the value and return it.
    - toRef transforms the value into a ref.

  - core concept:

    - If the value passed in is a ref, it will return the ref's value

    - If the value passed in is a function, it will call the function and return its value.

    - If the value passed in is just a value, it will return that value

  - syntax: `toValue(<value>)`
    the `<value>` can be refs,function, or just a value.

    <!-- ❌ -->
    <!-- ⚠️ --> // warning
    <!-- ✅ -->
    <!-- 📝 -->
