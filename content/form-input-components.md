---
title: "Form and Input Validation Components"
date: 2024-10-15
description: "A Simple Form and Input Components that validates your inputs automatically and can easily connect with server actions"
tags:
  - react
  - typescript
---

# Form and Input Validation Components

A React Simple Form and Input Components that validates your inputs on blur and can correct it on Change. It can easily connect with server actions if you wish to do so or the simple useSubmit Button. This Requires `Zod` for creating the input schema, This way the Zod schema that you're using in your _server action_ or in the _on Submit_ for validation can also be use here.

Example UI :
[My Portfolio Dashboard](https://www.tingexe.cc/dashboard)

- You can Play around in this dashboard it also have a 0Auth where you have to connect to github in order to submit the data. although the data submit will not reflect on the Project page, it will just to showcase the input

Example code snippets or boiler plate:

```typescript

   const userSchema = z.object({
     name: z.string().min(3, { message: "Name must be atleast 3 characters" }),
     email: z.string().email("invalid email address"),
     age: z.coerce.number().min(18, { message: "age must be atleast 18 years old" }),
     bio: z.string().max(100, "Bio must be must not exceed 100 characters"),
     avatar: z
       .instanceof(File)
       .refine((image) => image.size > 0, { message: "Image is required" })
       .refine((image) => image.type.startsWith("image/"), { message: "Must be an image" }),
   });

   const { Form, Input, Textarea, ErrorMessage } = GenerateFormComponents({ schema: userSchema });

   function MainPage(){
     return (
      <Form >
         <fieldset className="relative">
           <Input name="name" type="text"  />
           <ErrorMessage useDefaultStyles name="name"  />
         </fieldset>
          <fieldset className="relative">
            <Input name="email" type="email"  />
            <ErrorMessage useDefaultStyles name="email"  />
          </fieldset>
          <fieldset className="relative">
            <Input name="age" type="number"  />
            <ErrorMessage useDefaultStyles name="age"  />
          </fieldset>
          <fieldset className="relative">
            <Textarea name="bio"  />
            <ErrorMessage useDefaultStyles name="bio"  />
          </fieldset>
          <fieldset className="relative">
            <Input name="avatar" type="file"  />
            <ErrorMessage useDefaultStyles name="avatar"  />
          </fieldset>

          <button type="submit">Submit</button>
      </Form>
     )
   }
```

## Usage

The usage of this Component is straight Forward and dont need a lot boiler plate to setup.

1. **Create your schema**

   As mentioned above this component heavily depends on zod schema in order the validation to work. Let's set it up

   ```typescript
   const userSchema = z.object({
     name: z.string().min(3, { message: "Name must be atleast 3 characters" }),
     email: z.string().email("invalid email address"),
     age: z.coerce.number().min(18, { message: "age must be atleast 18 years old" }),
     bio: z.string().max(100, "Bio must be must not exceed 100 characters"),
     avatar: z
       .instanceof(File)
       .refine((image) => image.size > 0, { message: "Image is required" })
       .refine((image) => image.type.startsWith("image/"), { message: "Must be an image" }),
   })
   ```

2. **Setting up**

   First you have to generate the form and pass your schema.

   > It is Important to declare the `GenerateFormComponents`
   > Outside of your main component

   ```typescript
   const { Form, Input, Textarea, ErrorMessage } = GenerateFormComponents({ schema: userSchema });


   function MainPage(){
     return (
       <div>Im a Main Page</div>
     )
   }
   ```

   After Passing your schema **GenerateFormComponents** returns 4 components. Which are the following:

   - **Form** - like the normal Form where you can wrap your input

   - **Input and Textarea** - for your Inputs with a strict and autocomplete **name** attribute that only receives the keys/properties of your schema as a name.

   - **ErrorMessage** - Component where it will render/display the error message. Like Input and textArea it also strictly needs a **name** attribute and depending on the **name** attribute you passed in it to it will display the error validation of the input where it is connected.

   ErrorMessage Props:

   - name: Required Props with a string tupples of your schema keys/properties. It is restricted to only receive these names.
   - useDefaultStyles : A boolean to opt in for the out of the box styles. defaults to `false`, and by default the ErrorMessage Component will act like a normal **div**.
     > It is important to set the parent of input, wrapper element of the input, to be set to **position: relative** in order for the ErrorMessage Positions to work properly.
   - position: When opt-in for **useDefaultStyles** props. A tupples of position options where the ErrorMessage component will show. Choices are: _left_,_middle_,_right_ of **top** and **bottom**, combinations of these position, defaults to `bottomLeft`
   - errorMessageVariant: When opt-in for **useDefaultStyles** props. A tupple of **error** and **warning** where it the color of the ErrorMessage will change.

   The Components are stateless and the **ErrorMessage** is also stateless by default to support customization for your own design although you can also opt in for its out of the box styles.

3. **Using the Components**

   You can use the Components the same way Like you normally do with forms and inputs. It just have some features and an out of the box styles that you can opt in to.

   ```typescript

   function MainPage(){
     return (
      <Form >
         <fieldset className="relative">
           <Input name="name" type="text"  />
           <ErrorMessage useDefaultStyles name="name"  />
         </fieldset>
          <fieldset className="relative">
            <Input name="email" type="email"  />
            <ErrorMessage useDefaultStyles name="email"  />
          </fieldset>
          <fieldset className="relative">
            <Input name="age" type="number"  />
            <ErrorMessage useDefaultStyles name="age"  />
          </fieldset>
          <fieldset className="relative">
            <Textarea name="bio"  />
            <ErrorMessage useDefaultStyles name="bio"  />
          </fieldset>
          <fieldset className="relative">
            <Input name="avatar" type="file"  />
            <ErrorMessage useDefaultStyles name="avatar"  />
          </fieldset>

          <button type="submit">Submit</button>
      </Form>
     )
   }
   ```

   With Just that Structure you can now have a form and a validation error that can be display to the user. Not just that, The ErrorMessage Component will automatically clear as well after correcting the failed input
