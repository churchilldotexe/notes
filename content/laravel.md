---
title: All About Laravel
date: 2024-11-12
description: a progressive notes while learning laravel starting from basic
tags:
  - laravel
  - php
  - database
  - blade
---

## Blade templating

Blade templating is a way to write php code with html in a clean and readable manner.
It is heavily supported by Laravel because of its simplicity and easy to read syntax but under the hood the code is being converted to vanilla php code.

### `$slot`

You can think of a way to reserved a slot or certain part of the code. With having that reserved slot, it will make the code more dynamic and become usable on different files.
This is where the concept of component is.
In JS/React it is the same as `children`

Examples:

```php
<?php
//layout.blade.php
<body>{{$slot}}</body>


//home.blade.php
<x-layout>
  <h1>Home</h1>
</x-layout>
```

### Named `$slot`

While slot can make the component dynamic, named $slot will make the component even more dynamic.
The concept of named slot is to have a multiple slot in your component without creating an intersection or conflict with the other slot.

Example:

```php
<?php
//layout.blade.php
<body>
<nav>
  <h1>{{$title}}</h1>
</nav>

{{$slot}}
</body>


//home.blade.php
<x-layout>
  <x-slot:title>Home</x-slot:title>
  <section>

  <p>This is Home</p>
  </section>
</x-layout>
```

To resolve a conflict with slot in order to tell blade which slot to use the naming `<x-slot:title>` important, this way we can tell blade that the content inside the slot title will be render on where `$title` variable is.

### `$attributes`

Is a helper to pass down the props to the component. Meaning, once set in the component you can use the **Official** attribute of the html tag it was passed to.

Example:

```blade

<!-- component -->
<div {{$attributes}}>
  {{$slot}}
</div>

<!-- usage -->
<x-component class="text-red-100"></x-component>

```

- Merge
  is a method of `$attributes` to merge your component's attribute to the attribute on where the component is being use.

  Example

  ```blade
  <!-- component -->
  <div {{$attributes->merge(['class'=> 'bg-red-500 text-white'])}}>{{$slot}}</div>

  <!-- usage -->
  <x-component class='text-blue-500'>foo</x-component>

  <!-- output -->

  <div class='bg-red-500 text-blue-500'>foo</div>
  ```

- Get
  is another method of `$attributes`, it gets the value of the attribute. You can also define a fallback.
  This is good if you want to create a fallback for components that needs an attributes.

  Example:

  ```blade
  <!-- component -->
  <div class="{{$attributes->get('class','bg-red-500')}}">
    {{$slot}}
  </div>

  <!-- usage -->
  <x-component class="text-blue-500">foo</x-component>
  ```

### `$props`

has the same usage as attributes, you can define it inline.
If _named slot_ you can define as a children
_props_ can define inline.
It has the same characteristic with **named slot** but the usage is the same with **$attributes**

You can define the props in the component through laravel helper `@props()` function.

```blade
<!-- component -->
{{@props(['foo'=>true])}} <!--array or assoc array to provide default-->

<div class="{{foo? bg-red-500 : bg-blue-500}}"></div>
```

#### :

By Default, if you pass a value through props or attribute it is consider as a string.
To pass the actual data type to the component `:` must be used before the prop.

```blade
<!-- usage  -->
<x-component :foo="true" >bar</x-component>
```

### `$props` vs `$attributes`

props is for custom attributes
attrbiutes is the official attributes

## Helpers

A helper functions and classes directly from laravel.

### Request

[laravel docs for request](https://laravel.com/docs/11.x/requests)

Is a helper that returns the current request(http request) instance.

- #### `is` method

  A part of request method that checks the **path** and return a boolean if it matches the path.
  Example:

  ```blade
  <h1>
     @if(request()->is('/'))
        Im home
     @endif

     Not Home
  </h1>
  ```

  **wildcard** can also be used to match more/subsequent path.

```php
<?php
   request()->is('/dashboard/*');
```

<!-- TODO:  add more .-->

## Eloquent

Laravel ORM. It handles a lot of things when it comes to mapping database to your code such as :

- migration schema - you can define your tables in Database folder
- factory - fast setup and scaffold your database data.
- seed - Best pair with _factory_ and also pairs well with `migrate:fresh --seed`.
- Model

  - defining relationship - using Model to define relationship between tables.
  - setting up pagination.
  - can setup lazy or eager loading.

  <!-- TODO:  better deep dive on this topic especially how to use model methods. Read the docs-->

## Terminal Php artisan

A terminal helper that can scaffold a code depending on what you need.
For example it can help create a model which you can also create migration,factory and test for you in a single `php artisan make:model` And through this, you can define your table columns through the migration and factory so that you can use the helper again to migrate it for you and seed it as well with a single command `php artisan migrate --seed`

There are a lot of artisan command but you can define `help` before the command of the one that you need a help with to check the available flags and options .
Example:
`php artisan help migrate` will give you the flags and options in migrating your schema to the database .

Alternatively, you can run the command `php artisan help` to list all the artisan command.

## Model

This is where your database connections, authentication , tables relationship like foreign keys relationship and even CRUD operation

It binds your database table throught Eloquent and connect it to your Laravel code so that you can call it to query in your code base instead of querying directly to your database.

It also binds it to other class like policies(authorization), your Auth facade(authentication), or to your controller

You can define the model by using the terminal command `php artisan make:model` . Where you'll be prompted with questions like names

Example model:

```php
<?php

class User extends Authenticatable
{
    /** @use HasFactory<\Database\Factories\UserFactory> */
    use HasFactory;
    use Notifiable;

    /**
     * The attributes that are mass assignable.
     *
     * @var array<int, string>
     */
    protected $fillable = [
        'first_name',
        'last_name',
        'email',
        'password',
    ];

    /**
     * The attributes that should be hidden for serialization.
     *
     * @var array<int, string>
     */
    protected $hidden = [
        'password',
        'remember_token',
    ];

    /**
     * Get the attributes that should be cast.
     *
     * @return array<string, string>
     */
    protected function casts(): array
    {
        return [
            'email_verified_at' => 'datetime',
            'password' => 'hashed',
        ];
    }
}
```

This is a model that extends the Authenticatable which is responsible for authentication and the class that connects your User Model to the authentication related helpers provided by laravel, [more info About authentication. ](#authenticaton)

Another example that extends the model

```php
<?php

class Job extends Model
{
    use HasFactory;
    protected $table = 'job_listings';

    // protected $fillable = ['salary','title','employer_id','created_at','updated_at'];
    // or
    protected $guarded = ['id'];
    /**
     * @return \Illuminate\Database\Eloquent\Relations\BelongsTo<App\Models\Employer,App\Models\Job>
     */
    public function employer(): \Illuminate\Database\Eloquent\Relations\BelongsTo
    {
        return $this->belongsTo(Employer::class);
    }

    /**
     * @return \Illuminate\Database\Eloquent\Relations\BelongsToMany<App\Models\Tag,App\Models\Job>
     **/
    public function tags(): \Illuminate\Database\Eloquent\Relations\BelongsToMany
    {

        return $this->belongsToMany(Tag::class, foreignPivotKey: "job_listing_id");

    }

}
```

### Syntax /method of model class

#### **HasFactory**

- is from the `model` extension that binds with the factory class. This is responsible for seeding and the class that connects to the seesing class.

#### **$table**

- is the table name from your database. To be able to connect your orm model to the correct database table.

#### **$fillable**

- one of the security feature of the `model` class. To protect your database from sql injections you can set what are allowed to be filled in your database and if other field/columns were being stored in your database an _execption_ will be thrown.

#### **guarded**

- the same security feature like **fillable** but the way it define is the opposite. Here you will define what are the columns that must not be query/stored

#### **hidden**

- is another security feature for your model. It is a way to avoid the columns define here from being serialized/access when querying the table. In the example model `User` , it defined `password` and `remember_token` as hidden. This means when the user database is being queried it will not include the password and remember_token.

  Example:

  ```php
  <?php
  $user = User::find(1);
  return response->json();
  ```

  Will return json without password and remember_token.

  This is a useful feature to avoid accidental leaking of sensitive information.

#### **Cast**

- Is a way for laravel/eloquent on how to handle the column when it is being retrieved or stored to the database. Laravel will convert(cast) that data to a certain datatype that you need.

- Usage
  create a protected method name `cast` and return an `assoc array` where the **key** is the `table name` and the **value** is the cast types. Refer in the document for the cast types

  [laravel docs about cast system](https://laravel.com/docs/11.x/eloquent-mutators#attribute-casting)

  For example(from the table above):

  - Hashing the **password**, laravel will hash the password when it is being stored to the database
    and, for example, the user login with the password laravel will also convert it and with the help of `Auth` facade it will compare both of the password to authenticate the user.

### Binding

To bind your route with the model to automatically query your database.
This is helpful to lessen the logic since you can use the binded model for your CRUD operation.
Most used in a route with a wild card.

- Usage
  Just need to define your model as a **type**.

Example:

```php
<?php
    //this
    public function show(Job $job): View
    {
        return view('jobs.show', ['job' => $job  ]);
    }

    // instead of this
    public function show($job): View
    {
        $job = Job::query->find($job);
        return view('jobs.show', ['job' => $job  ]);
    }
```

## Authenticaton

The most common process for authentication are as follows:

1. ### Validation

   checking the data/input that was sent to the server.

   Example:

   ```php
   <?Php
    $attributes = request()->validate([
      'email' => ['email','required'],
      'password' => ['required',Password::min(6)]
    ]);

     // this will check two passwords against each other the input name should be
     'password' => ['required',Password::min(6),'confirmed'],

     // password_confirmation
   ```

   This will check and validate the data and it returns the validated data
   or the error message back to its source ,
   then this is where you'll receive and display the error message.

   The pattern is

   using assoc array:

   - **`"name" => ['rule','rule',...]`** or

   using regular expression:

   - **`"name" => "'rule'|'rule'|rule..."`**

   [List of rules](https://laravel.com/docs/11.x/validation#available-validation-rules)

   `'confirmed'` is used if you have an input that needs to be repeated for validation.

   **`<name>_confirmation`** must be the name so that laravel can check and validate both input against each other.

   - **FAILED validation**

   If the validation fails the user will be redirected to the source page and the message will be [flashed to session storage](https://laravel.com/docs/11.x/session#flash-data).

   Which then you can access with blade helper @error()

   e.g.

   ```blade
    @error("$name")
        <p class="text-red-500  font-semibold text-xs line-clamp-1 ">{{ $message }}</p>
    @enderror
   ```

   **XHR request** if the source request is an XHR request, then the validation will
   **redirect and return a HTTP response of 422 status code with the json response containing the validation errors.**

   This is useful for AJAX requests and this is what **inertia** uses for validation.

   #### Illuminate Password

   [docs here](https://laravel.com/docs/11.x/validation#validating-passwords)

   `Password` came from `use Illuminate\Validation\Rules\Password;`

   A helper from laravel that validates the received data for passwords restrictions

   - min : character length restriction
   - max : character length restrictions
   - mixedCase : uppercase and lowercase
   - symbols : special characters
   - uncompromised : will check if the password is exposed in [ haveibeenpwned ](https://haveibeenpwned.com/)
     This will check if the data have been leaked in the said website once(or the amount that you put in the argument)

   This validation can be **chained** with other validation rules.

   Example:

   ```php
   <?php

   $attributes = request()->validate([
     'password' => ['required', Password::min(6)->symbols()->mixedCase()->uncompromised(3)]
   ]);
   // this will check for minimum 6 character that must have a symbol and a mixed mixedCase
   // and that wasnt leaked 3 times.
   ```

   #### Request validation

   Different ways to do validation:

   - Manual validation `Validator::make`

     [full docs for manual validation](https://laravel.com/docs/11.x/validation#manually-creating-validators)

     This came from `Illuminate\Support\Facades\Validator;`

     Where you can call and manually setup the validation
     And you can also set a custom message for the validation

     This is good if you want to fine tune the validation.

   - Laravel helper `Request()->validate(['name'=>['rules'...]])`

     The more straightforward validation where you can either get the validated data or redirected back to source with the flashed validation errors

   - Own class validation (`php artisan make:request`)

     That will be located in `app/Http/Requests` folder.

     This folder is where requests take place its normally connected with the [ controller ](#controller).

     By separating the validation , you have a dedicated class for handling requests like validation.
     And to use it you just have to bind it to your controller and type hint it.

   ##### Customizing validation message

   [laravel docs for customizing validation message](https://laravel.com/docs/11.x/validation#manual-customizing-the-error-messages)
   There are different ways to customize the error message, one of which is by creating a `public message` function in your Request class. (Example shown below)

   This involves by creating a `public message` function in your Request class.

   Example:

   ```php

    <?php

    namespace App\Http\Requests;

    use Illuminate\Foundation\Http\FormRequest;

    class InfoRequest extends FormRequest
    {
      /**
      * Determine if the user is authorized to make this request.
      */
      public function authorize(): bool
      {
          return true;
      }

      /**
      * Get the validation rules that apply to the request.
      *
      * @return array<string, \Illuminate\Contracts\Validation\ValidationRule|array<mixed>|string>
      */
      public function rules(): array
      {
          return [
                'name' => ['required', 'min:3'],
                'email' => ['required', 'email', 'unique:App\\Models\\User'],
                'phone' => ['required', 'min:10'],

          ];
      }

      public function messages(): array
      {
          return [
                'required' => ':attribute field is required',
                'min' => 'Atleast :min character is required',
                'email.email' => 'Please provide a valid email',
                'email.unique' => 'Email already taken',
          ];
      }
    }

   ```

   Usage example:

   ```php

   <?php

   namespace App\Http\Controllers;

   use App\Enums\FormSection;
   use App\Http\Requests\InfoRequest;
   use App\Traits\HandlesFormSessions;
   use Illuminate\Contracts\View\View;
   use Illuminate\Http\RedirectResponse;

   class InfoController extends Controller
   {
      use HandlesFormSessions;

      public function store(InfoRequest $request): RedirectResponse
      {
         // this will validate all the inputs and return the value if success, redirect to source if fails
         $validatedData = $request->validated();

         //you can also FILTER the validation data
         $validatedData = $request->safe()->only(['name', 'email']);

         // you can also exclude some
         $validatedData = $request->safe()->except(['name', 'email']);

         $this->storeFormSessionData(FormSection::INFO, $validatedData);

         return redirect('/plans');
      }
   }
   ```

2. creating and storing user

This is only necessary for user registration.

After validation, we can interact to the User model(for eloquent), to create a user.

The faster/easier way is to get the returned value from the validation and use it to create a user.

```php
<?php
$validatedData = $this->validate($request, [
   'name' => 'required|string|max:255',
   'email' => 'required|string|email|max:255|unique:users',
   'password' => 'required|string|min:8|confirmed',
]);
$user = User::create($validatedData);
```

3. ### Attempt to login(Auth facade)

[detailed documentation about Authentication](https://laravel.com/docs/11.x/authentication)

[Auth facade api docs](https://laravel.com/api/11.x/Illuminate/Support/Facades/Auth.html)

After a successful validation, **Auth** facade `Illuminate\Support\Facades\Auth`
can be use to login.

**Auth facade** is a static class from laravel service container.
That can help you interact with the currently Authenticated User.

The recommended process are the following:

- For registration process:

  - validate the User and store data to database

  - `Auth::login($user)` to login the user, where `$user` is the user returned data.

- For login process:

  ⚠️ It is important to set the logout to `POST` method not a `GET` method like an anchor tag.

  - after validation, you can use attempt to login using `Auth::attempt($attributes)`
    This attempt will return a boolean

    **handling failed login**

    When failed login occurs, like wrong password, email not found, etc,.
    `Auth::attempt($attributes)` will return false and with this you can
    throw an exception like `throw ValidationException::withMessages(['email' => 'Invalid Credentials, Please Try again']);`
    and redirect the user back to login page

    📝 Note: if you use `ValidationException::withMessages`,
    it will redirect you back to the source url including the message

  - once the user is logged in, it is important to
    [regenerate the session](https://laravel.com/docs/11.x/session#regenerating-the-session-id) id
    to avoid session hijacking.

  - then redirect to the desired page.
    `redirect()->intended();` : home page default

  Simple example for **login authentication**:

  ```php
  <?php
  public function store(Request $request): RedirectResponse
  {
      //validate
      $attributes = $request->validate([
          'email' => ['email','required'],
          'password' => ['required',Password::min(6)]
      ]);
      //attempt to login
      if (! Auth::attempt($attributes)) {
          //throw exception
          throw ValidationException::withMessages(['email' => 'Invalid Credentials, Please Try again']);
      }
      //regenerate session id
      $request->session()->regenerate();
      //redirect

      return redirect('/jobs');
  }
  ```

- For Logout Process:

  ⚠️ It is important to set the logout to `POST` method not a `GET` method like an anchor tag.

  Logging out is a two step process:

  - first, you need to destroy the session
    `Auth::logout();`

  - then redirect to the desired page.
    `redirect()->intended();` : home page default

For redirect:
redirect()->guest('login');
redirect()->intended();//home page default

## authorization

There are two ways to control authorization in Laravel.

**Policies** is the way to control authorization that is coupled with a model or resource.

> **Gates and Policies** can worked together.
>
> You can use **gates** to control authorization that is simple and not models related like viewing certain authorized pages
>
> and use **policies** to control authorization that is tightly coupled with a model or resource like editing a data in the dashboard or deleting entry.

**NOTE 📝**

> Map only works for the table that has a relationship with the user table.
> In order for laravel to know and compare the currently authenticated user with the column data . That column data must be connected to user or its related table is connected to a user.

<!-- defined by relationship between the user and the model -->

Example of simple Authorization:

```Php
<?php

// assuming this in a method in the Controller
public function edit(Job $job){ //binding the job

   //will check if the user is logged in
   if(Auth::guest()){
      return redirect()->guest('login');
   }

   // the autohorization
   // will check if the logged in user is the same to the jobs user(relationship)
   if(! $job->user->is(Auth::user())){ // will check if the  user column data(e.g. id) will match with Auth::user()
      abort(403); // will display a forbidden error page
   }
}
```

As mentioned that this is simple because it only belongs to the controller so if you need to do some logic elsewhere you need to recreate this logic again, so its better to use [gate](#gates).

### Gates

**Gates** is the simpler way to control authorization. It is mostly used for authorization that is not tightly coupled with a model. Although you can, Policies is probably the best approach for that

#### Defining gates

Gates can be defined in the `app/Providers/AuthServiceProvider` file so that it initializes on boot and be ready to use.

Gate::define() takes two parameters:

1. name: the name of the gate to be **accessed globally** in the application
2. a callback that determines whether the action is authorized or not.(will return boolean)

Example:

```php
<?php
    public function boot(): void
    {
                     //name          |callback  User  Model , Post Model
        Gate::define('update-post', function (User $user, Post $post) {
            return $user->id === $post->user_id;
        });

         // or with more arguments for more context on logic
        Gate::define('foo', function (User $user, Post $post, bool $bar) {
            if($user->id === $post->user_id){
               return true;
            } if ($bar) {
               return false;
            }
        });

         // alternatively you can use class callback array

         Gate::define('update-post', [PostPolicy::class, 'update']); // will use the update method from the PostPolicy class
    }
```

- User: the binded model for the logins
- Post: the binded model that you want to check against the user
- Return: bool, the callback will return truthy or falsy depending on the matching logic.
  or a class callback array where you can use the [policy](#policies) method name instead of a closure.

> ✅ Good to know .
>
> If you have a [policy](#policies) created for autorization, you can use to define your gates
> this way you can access it using `Gate::authorize` , `Gate::allows` , `Gate::denies` and some others.

> 📝 **NOTE**
>
> the **User** in the closure will always check for the authenticated user.
> Means if the user is not logged in and the gate is called
> it will not hit the gate logic and will automatically return false
> ✅ To Fix it you can make the `$user=null` or make it optional `?User $user`

If the user is not logged in and will try to access the gate it will redirect it to the named login route.

#### Defining Gates using Responses

This is useful if you want to return a more detailed response then a boolean.

You can use the `Response` static class from `Illuminate\Auth\Access\Response`

Where you can pass a message together with the HTTP status response.

Example:

```php
<?php

use Illuminate\Auth\Access\Response

    public function boot(): void
    {
                     //name          |callback  User  Model , Post Model
        Gate::define('update-post', function (User $user, Post $post) {
            return $user->id === $post->user_id  ? Response::allow() : Response::deny('You are not authorized to update this post');
        });
    }
```

Alternatively, it is a best security practice to return a 404 instead a 403 to hide the existence of a resource.

Here is why:

- to avoid enumeration attacks, if the response is 403 it will hint the attacker that the resource exists just dont have enough authorization instead of 404 it will hint that the page doesnt exist even if it may does.

- for better ux, it will avoid confusion to the user.

laravel provides a helper function to return a 404 response using `Response`

```php
<?php

use Illuminate\Auth\Access\Response

    public function boot(): void
    {
                     //name          |callback  User Model , Post Model
        Gate::define('update-post', function (User $user, Post $post) {
            return $user->id === $post->user_id  ? Response::allow() : Response::denyWithStatsus(404, "Post not Found"); // you can customize the message too
        });

         // or laravel helper
        Gate::define('update-post', function (User $user, Post $post) {
            return $user->id === $post->user_id  ? Response::allow() : Response::denyAsNotFound();
        });
    }
```

#### Intercepting gate checks

[laravel docs for Intercepting gate checks](https://laravel.com/docs/11.x/authorization#intercepting-gate-checks)

If you need to intercept the authorization before the [defined gates](#defining-gates) you can use:
`Gate::before` where you can bypadd the gate checks. This is useful if for admins that needs access to all the routes.

You can define `Gate::before` in the `app/Providers/AuthServiceProvider` file so that it initializes on boot and can be access globally.

Example:

```php
<?php
    public function boot(): void
    {
        Gate::before(function (User $user,string $ability) {
            if ($user->isAdmin()) {
                return true;
            }
        });
    }
```

Alternatively you can use `Gate::after` to run after the gate checks. This is useful if you want to do a logging for successful or failed authorization.

Example:

```php
<?php
public function boot(): void
{
   Gate::after(function (User $user, string $ability, bool|null $result) {
      \Log::info("User {$user->id} attempted {$ability}: " . ($result ? 'Success' : 'Failure'));
   });
}
```

📝 Note:

> Both `Gate::before` and `Gate::after` will be called for all gates.

#### Accessing the gate

There are several ways to access the gates.

1. Accessing directly in the controller

   - Directly in the controller

     ```php
     <?php
     public function edit(Post $post){
        //gates can be used in the controller
        if(Authh::user()->cannot('update-post',$post)){
           abort(403);
        }

        // or
        Gate::authorize('update-post',$post);
        Gate::allows('update-post',$post);
        Gate::denies('update-post',$post);
        Gate::any(['update-post','foo'],[$post,$bar]);
        Gate::none(['update-post','foo'],[$post,$bar]);

        // or accessing gate with multiple arguments
        Gate::any('foo',[$post,$bar]);
        Gate::none('foo',[$post,$bar]);
     }
     ```

     **Gate::authorize** - will run the action and will check if the user is authorized for that action.
     if not it will **throw an instance of AuthorizationException** from `Illuminate\Auth\Access\AuthorizationException` which will throw an **HTTP response of 403(forbidden)**

     **Gate::allows** - will run the action and will check if the user is authorized for that action.
     this will return a `boolean` instead of `Exception`.
     False if not allowed to the such action else true.

     **Gate::denies** - the opposite of **Gate::allows**

     **Gate::any** - is like **Gate::allows** but it will check if the user is authorized for _any or either_ of the actions. So it is a gate for multiple actions.

     **Gate::none** - is like **Gate::denies** but it will check if the user is authorized for _none or neither_ of the actions.

   - using Gate defined through [ Responses ](#defining-gates-using-responses)

     Accessing using **Gate::inspect**

     ```php
     <?php
     public function edit(Post $post){

        $result = Gate::inspect('update-post',$post);

        // will return a boolean
        if($result->allowed()){
           // now authorized do something..
           // like updating the db and redirecting
        } else {
           echo $result->message();
           // inertia you can pass the message as a prop
            return back()->with([
                'message' => $result->message(),
            ]);
        }
     }
     ```

     or using **Gate::authorize**

     ```php
     <?php
     public function edit(Post $post){
        Gate::authorize('update-post',$post);
       // if fails will throw 403 forbidden with the custom message from the Response->deny()
      // which is : You are not authorized to update this post
     }
     ```

     [check inertia error handling ](https://inertiajs.com/error-handling)

   - using Can for `Auth::user()` facade

     ```php
     <?php
     public function edit(Post $post){
        if(Auth::user()->cannot('update-post',$post)){
           abort(403); // or fail the request
        }
     }
     ```

2. using blade

   you can use `@can` blade directive to check if the user is authorized for the action.

   You need to match the gate name (first argument) with the data it needs to check against.

   ```blade
   // gates can be used in blade template
   @can('update-post',$post)
      <button>Update</button>
   @endcan
   ```

   If you have [policies](#policies), there is no change in the syntax.
   But you might want to update your `can` directive to match to the policy method name.

   ```blade
   @can('update',$post)
      <button>Update</button>
   @endcan
   ```

3. ##### Accessing gate in Route level through middleware

   The problem with accessing the gate in controller is that you have to define it in the controller everytime you need it.

   Middleware solves this instead of accessing gate in the controller it self you can do a gate check in the route level where controllers are connected.

   There are different ways to do it.

   ```php
   <?php
      Route::get('/edit/{post}', [JobController::class, 'edit'])->middleware(['auth','can:update-post,post']);
   ```

   the `can:update-post,post` , the **post** here represents the wildcard from the first argument for **Route::get**

   alternatively you can use `can` method for better readability

   ```php
   <?php
      //                                                                                   gate name , wildcard
      Route::get('/edit/{post}', [JobController::class, 'edit'])->middleware('auth')->can('update-post','post');
   ```

   **using [Policy](#policies)**

   if youre using policy you can access it through middleware just like gate but with a little different syntax

   ```php
   <?php
      Route::get('/edit/{post}', [JobController::class, 'edit'])
         ->middleware('auth')
         ->can('update','post'); // no more update-post
   ```

   What this does under the hood is, it will check the JobController and its **binded** model then will check if a policy exists and finally it will then check and **run** the update method inside that policy.

   That is why it is important to connect the policy to the model.

   _alternatively_, you can group controller together like so:

   ```Php
    <?php

   Route::controller(JobController::class)->group(function () {

      // index
      Route::get('/jobs', 'index');

      //store
      Route::post('/jobs', 'store')
         ->middleware('auth');

      //create
      Route::get('/jobs/create', 'create')
         ->middleware('auth');

      //show
      Route::get('/jobs/{job}', 'show');

      //edit
      Route::get('/jobs/{job}/edit', 'edit')
         ->middleware('auth')
         ->can('edit', 'job');

      //update
      Route::patch('/jobs/{job}', 'update')
         ->middleware('auth')
         ->can('edit', 'job');

      // delete
      Route::delete('/jobs/{job}', 'destroy')
         ->middleware('auth')
         ->can('edit', 'job');
   });
   ```

   or

   ```php
   <?php
      Route::middleware(['auth', 'can:edit,job'])->group(function () {
         Route::get('/jobs/{job}/edit', 'edit');
         Route::patch('/jobs/{job}', 'update');
         Route::delete('/jobs/{job}', 'destroy');
      });
   ```

   you can also create a middleware that will check for the gates and/or policies.

   ```php
      <?php
      // In app/Http/Middleware/EnsureJobCanBeEdited.php
      class EnsureJobCanBeEdited
      {
         public function handle($request, Closure $next)
         {
            $job = $request->route('job');

            if (!Gate::allows('edit', $job)) {
                  abort(403);
            }

            return $next($request);
         }
      }

      // Then in routes
      Route::patch('/jobs/{job}', 'update')
         ->middleware(['auth', EnsureJobCanBeEdited::class]);
   ```

### Policies

Policies is the way to control authorization that is tightly coupled with a model or resource.

`php artisan make:policy` will scaffold a policy for you. To connect it to your model there will be a prompt that will ask you the model you want to connect it to.

Connecting to the model is important because laravel is going to use it to check for the policy that is connected to your model, especially when run with `can` through **middleware**, there is a [demo example here.](#accessing-gate--route-level-through-middleware)

- **Accessing policy methods through gates**

  You can use the policy logic to your [gates](#gates) by adding the policy class instead of callables when defining the gates. [example shown here](#defining-gates)

📝 **NOTE**

> You can use use the logic from defining gates to your policies methods.
> The logic/idea is just the same where the method from policy just need to return a boolean | [Response](#defining-gates-using-responses).
> same with how you [define gates](#defining-gates)

#### Policy Responses

just like [gate responses](#defining-gates-using-responses) you can also use it in Policies method too.
You can also create a custom response message.

Example:

```php
<?php
public function update(User $user, Post $post): Response
{
   return $user->id === $post->user_id  ? Response::allow() : Response::deny('You are not authorized to update this post');
}
```

or use the `denyAsNotFound` helper or `denyWithStatus` helper.

**Accessing the policy**

As what mention above, you have to use the policy class instead of the callable when defining the gates.
to be able to use it in through [gates](#gates) but if you're going to access it through [middleware](#accessing-gate-in-route-level-through-middleware) you can use it as it is.

### Authorization with Inertia

[docs regarding authorization with inertia](https://laravel.com/docs/11.x/authorization#authorization-and-inertia)

you can use the **HandleInertiaRequests** middleware to use **can** method for authorization.

example:

```php
<?php

namespace App\Http\Middleware;

use App\Models\Post;
use Illuminate\Http\Request;
use Inertia\Middleware;

class HandleInertiaRequests extends Middleware
{
    // ...

    /**
     * Define the props that are shared by default.
     *
     * @return array<string, mixed>
     */
    public function share(Request $request)
    {
        return [
            ...parent::share($request),
            'auth' => [
                'user' => $request->user(),
                'permissions' => [
                    'post' => [
                                 // this is where you set the authorization through can.
                        'create' => $request->user()->can('create', Post::class),
                    ],
                ],
            ],
        ];
    }
}

```

with this you can now access it in your frontend of choice Vue/React/svelte though grobal props.

Example:

```vue
<template>
  <div>
    <h1>Create a Post</h1>

    <!-- Conditionally show the create button if the user has permission -->
    <div v-if="canCreatePost">
      <button @click="createPost">Create New Post</button>
    </div>

    <div v-else>
      <p>You do not have permission to create a post.</p>
    </div>
  </div>
</template>

<script setup>
import { computed } from "vue"

// Props from Inertia (automatically passed to every page)
defineProps({
  auth: Object, // Receiving the shared 'auth' object from Laravel middleware
})

// Computed property to check if the user can create a post
const canCreatePost = computed(() => {
  return auth?.permissions?.post?.create ?? false
})

// Function to handle the "Create Post" button click
const createPost = () => {
  alert("Post creation logic goes here!")
}
</script>
```

**Another approach**

This approach uses the `can` method directly at route level.
This is good if you want to only access the autohorization per route and not globally through the middleware.

_Steps_:

Add can(gate) to as a prop for your frontend to receive
or fine grained control (like only for admin)

Things to note:

- to access the gate pass it as a prop

```php
<?php

// pass the prop like this

Route::get('/', function () {
    return Inertia::render('Home', [
        'name' => 'ting',
        'laravelVersion' => Application::VERSION,
        'phpVersion' => PHP_VERSION,
        'can' => [ 'createUser': Auth::user()->email === 'admin@admin.com' ]
    ]);
});

```

or create a policy

```php
<?php

// pass the prop like this

'can' => [
'createUser': Auth::user()->can('create', User::class)
  //                               ^ this is the gate from policy
]
```

<!-- ❌ -->
<!-- ⚠️ --> // warning
<!-- ✅ -->
<!-- 📝 -->
