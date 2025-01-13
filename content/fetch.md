---
title: "Fetch Api deep dive"
date: 2024-09-23
description: "Understanding more about fetch method in JavaScript. This Blog will cover starting from the basic and then the different ways to pass url, understanding the initOptions which is the second paramter of the fetch method,how CORS works and when it triggers, multiple fetch request that depends on another request and also multiple independent request , and lastly how to know the fetching progress"
tags:
  - fetch
  - javascript
---

# Fetch API

a way of JavaScript to tell the browser to get a data without refreshing the entire page.
It uses a asynchronous method `fetch()` to get the data from the URL that is passed as an argument.

### Basic ways to handle the **fetch()** API

1. chaining fetch() using .then and .catch

```JavaScript
fetch(url)
   .then((response)=>{
      if(!response.ok){
         throw new Error("An error occurred")
      }

      return response.json()
   })
   .then((data)=>{
      return data // or do something with the data
   })
   .catch(error){
      console.error(error)
   }
```

- in this simple fetch and then with catch method is a way to handle the asynchronous nature of the fetch(). Since it is a Promise we have to wait for the data to give us a response and do something with data. The line `if(!response.ok)` is a way for us to check if we successfully get the data, which is 200+, if not then it will be catch in the .catch chain if it is a success we now mutate the data to convert the `json`, assumming that url/api gives a json response, and convert it to an object which is this line of code `return response.json()`. We now use that response that is being received by another .then chain and that is where we can do something about the data.

2. using `async/await` in conjunction of `try{}catch(){}`

```JavaScript
async function fetcher (url){

   try{
      const response = await fetch(url)

      if(!response.ok){
         throw new Error("An error Occured")
      }

      const data = await response.json()
   }
   catch(error){
      console.error(error)
   }

}

```

- this approach is the more preferred but it just does the same over the `then chaining syntax`. The error handling of `if(!response.ok)` is now being catched using the catch from `try catch`, which is necessary when handling fetch for error checking.

### Types of fetched response code

- `100 - 199` -> informational response
- `200 - 299` -> successful response: This is where the response.ok becomes `true`. The response that you want and expecting from the fetch, a happy path.
- `300 - 399` -> a redirect response (where you still get something but the url that you fetched got redirected for some reason)
- `400 - 499` -> a client error response (this error occurred due to mismatched from server or there is something wrong with the client api fetching)
- `500 - 599` -> a server error response (there is something wrong with a server, typically server down or programming from server side failure)

For a detailed information of the HTTP response status codes check: [mdn HTTP status codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)

the normal setup for checking is something like this :

```JavaScript
   if(!response.ok){
   // throw the error
   }
```

### fetch() argument

There are three ways that you can pass in the first argument of the fetch() api. Which are string, Url interface, and the Request object.

1. String - the most used argument for fetch. It is the full url information in string. Example: `https://api.example.com/path?query=value`
2. URL interface - if you want the bits and pieces of information from the string url. You can pass it in the Url constructor.

   ```JavaScript
      const url = new URL("https://api.example.com:456/path#hash?query=value")
      console.log(url.host) // api.example.com:456
      console.log(url.hash)  // hash
      console.log(url.hostname)  // api.example.com
      console.log(url.href)  // https://api.example.com:456/path#hash?query=value
      console.log(url.pathname)  // /path
      console.log(url.searchParams.get(quert))  // value

      fetch(url)
   ```

> [mdn URL constructor docs](https://developer.mozilla.org/en-US/docs/Web/API/URL)

3. Request object - by using the request object you can also pass the second parameter of the fetch method which is the , options.

   ```JavaScript
      const fooObj = {baz:bar}
      const url = new URL("https://api.example.com:456/path#hash?query=value")
      const request = new Request(url, {
         method: "POST",
         headers: { "Content-type":"application/json" },
         body: JSON.stringify(fooObj)
      })

      fetch(request)

   ```

> [mdn Request object docs](https://developer.mozilla.org/en-US/docs/Web/API/Request)

#### Method types

- `HEAD` = where you can send a request and ask for a response but only the response headers
- `GET` = the same with the head but includes body.
- `POST` = submit a new entity to a resource
- `PUT` = replace the entity of the target existing resource
- `PATCH` = its like an update where it only modifies part of the existing resource
- `DELETE` = delete the specified resource

> [more method from MDN docs](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods)

### Response object

Response have the same structure with Request where it has a head and a body,footer. The Response constructor can have two arguments which is the body, and the options on which you can put the headers and other options.

- the Body parameter can take the following as an argument:
  - Blob
  - string
  - formData
  - URLSEarchParams
  - more...
- the Options are:
  - status - the status the response that you want the client to receive. E.g. 200(for success), 400+/404 for client error, 500+ for server
  - statusText- that status message that is associated with your status code. [mdn HTTP status codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)
  - headers - the information and/or additional information that you want the client to receive. Which is a json of a custom headers or the HTTP headers like `Content-type`, `Content-length`
    - for the custom headers it is a key/value pair header where the key must start with _x_ like: `x-custom-header: "im a header"`

> [mdn HTTP headers](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers)

Example of Response object:

```JavaScript

   const obj = {
      randomId: crypto.randomUUID(),
      name: "bar"
      foo : 'baz',
   }


   function createResponse (){
      const stringifiedObj = JSON.stringify(obj)
      const file = new File([stringifiedObj], "obj.json", {type: "application/json"})

      const headers = new Response(file, {
         status: 200,
         statusText: "OK",
         headers: {
            'Content-type': 'application/json',
            'Content-length': file.size,
            'x-custom-header': 'im a custom header'
         }
      })

      // if you want to get the headers information
      console.log( response.headers.get('Content-type'))
      console.log( response.headers.get('Content-length'))

      return headers
   }

```

### Fetch response types

- response.json() = for json files
- response.text() = for text,html,xml,css, and js
- response.blob() = Binary large Object. Which are files,images, Fonts ,video and audio

#### fetch() for response.blob()

```JavaScript
async function getData (){
      try{
         const response = await fetch("https://image.photos/id/123")
         if(!response.ok){throw new Error("An error Occured")}
         const data = response.blob()

         const url = URL.createObjectURL(data)
         const img = document.getElementById("img")
         img.src = url

      } catch(e){
         console.error(e)
      }
}

```

In this fetch call we are getting the data from the url and receive it as a blob (Binary large Object). Now, the line `const data = response.blob()` get the data from the url as a chunk of memory that saves on the user computer so we can use this data. In the line `const url = URL.createObjectURL(data)` we create a URL and that url will server as a **pointer** for that memory chunk so we can use it as a source in our image. It is important to know that the url that was create by using URL.createObjectURL is not the url that is from the fetch argument. It is the URL that you get as a pointer for the saved blob in memory

#### fetch() using response.text()

```JavaScript

async function getData (){
   const header = document.querySelector("header")

      try{
         const response = await fetch("https://api.server.com/example")
         if(!response.ok){throw new Error("An error Occured")}
         // assuming that the server response a json array
         const data = response.json()

         header.innerHtml = data.map(({name,id})=>{
         return `<li data-id=${id}>
                  <p>${name}</p>
                 </li>`
         }).join("")

      } catch(e){
         console.error(e)
      }
}
```

This fetch call is the most common and the most used fetch calling. Where you have to get a data from an api and transform it as an object and use that object to display in the html. In the code we use the data from the api and do the map method to to iterate over the array and display the object that we need, in the example it is the name and the id and join them all as a single string. We use map here instead of normal iteration like `innerHtml +=` to avoid constant repainting of the dom because in using map method we are displaying the content all at one after the map.

### fetch headers, searchParams, api and authorization

In the fetch() method you can send specific information in the server using URL searchParams and headers.

- using searchParams:
  you can create a search params by just editing the url and add `?` and your key value pair or you can use the URL constructor

```JavaScript
const urlString = `https://sample.com/?key=value`

// using constructor
const url = new URL(urlString)
const searchParams = url.searchParams

searchParams.append('key','value1') // adds the key value pair
searchParams.append('x-api-key','your-api-key') // adds the key value pair
//`https://sample.com/?key=value&key=value1&x-api-key=your-api-key`

searchParams.set('key','value2') // will add but will delete the old key that has conflict(like an update)
//`https://sample.com/?key=value2&x-api-key=your-api-key`

fetch(url)
```

- using headers
  You can also send information to the server using headers. The headers being sent depends on the server requirements, especially for the custom headers.
  Not all headers that you put are valid headers, there are forbidden headers which is immutable and will be ignored by the browser once you do the fetch request this is to ensure the security and validity of the fetch. Some immutable headers are origins, host, and many more.

There are two ways of passing headers to the fetch() method

1. direct passing

```JavaScript

fetch(url,{method:"GET", headers:{
'x-api-key':'your-api-key',
'Authorization': 'Bearer yourJWTtoken'
'Content-type': 'application/json' // for post request
'origin':'https://sample.com' // forbidden hence will be ignored
}})

```

This approach is the most commonly used approach since it is straight forward.

2. using the Headers constructor
   [about headers constructor from mdn](https://developer.mozilla.org/en-US/docs/Web/API/Headers/Headers)

```JavaScript

const header = new Headers()
header.append( 'x-api-key','your-api-key')
header.append( 'Authorization', 'Bearer yourJWTtoken')
header.append( 'Content-type', 'application/json')// for post request
header.append( 'origin','https://sample.com' ) // forbidden hence will be ignored for GET method

fetch(url,{method:"GET", headers:header})
```

This approach have benefits like the constructor method like append(), set(), and delete() and can also do validation checking of the headers at run time, means before the fetch() do the network request, it will validate the format of the headers then throw error if it is malformed and also will check the forbidden headers and ignore them.

Forbidden headers :

- cookie
- Sec-
- Proxy -
- host
- origin
- and others : [forbidden header name, from MDN](https://developer.mozilla.org/en-US/docs/Glossary/Forbidden_header_name)

#### Content security policy

Is a way for the fetch() method to know where it is allowed to connect to. This can also be set in the html `<head>` tag and can also be set for fetch call.
It can receive a content of string to where you can specify where you want the fetch to connect to.

```JavaScript
// passing it in the head tag
<head>
<meta
http-equiv="Content-Security-Policy"
content="connect-src 'http' 'https' example.com;"
/>

using as a header
header: {
'Content-Security-Policy' : 'http example.com'
}
</head>
```

there are more content that can be used in for the Content-Security-Policy like `default-src` , `img-src`, `script-src`.

> [docs for Content-Security-Policy from mdn](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP#threats)

### Credentials

This is one of the property for the second parameter of the fetch(). It can receive three values:

1. omit - means never send the credentials in the fetch Request
2. same-origin - means you can send my credentials like cookies but only if we have the same origin
3. include - means you can send my credentials with this request but most of the browser will block this type of credentials due to cors error. It is a way for a browser to secure the credentials of the user.

### Uploading data using fetch()

The way to upload data is to use the methods: `POST`, `PATCH`, `PUT`. `POST` is to create new resources/entries typically used for form data. PUT replaces the resources or update it while PATCH only modifies part of it.

- using JavaScript uploading json

```JavaScript
const myfile = document.getElementById("my-file") // input type=file

document.getElementById("form").addEventListerner("submit",(event)=>{
   event.preventDefault()

   const obj = {
      id: 123,
      foo: "bar"
   }

   const objStringified = JSON.stringify(obj)

   fetch("https://url.to.post.com/",{
      method:"POST",
      headers:{
         'Content-type':'application/json'
      }
      body: objStringified
   })

})
```

In this fetch method `POST` is used to upload the data to the server.

> The `Content-type` in the headers is the type that you want to upload and not the response from the server

- Using FormData to do `POST` method

```JavaScript
const myfile = document.getElementById("my-file") // input type=file

document.getElementById("form").addEventListerner("submit",(event)=>{

const formData = new FormData(document.getElementById("form"))

   fetch("https://url.to.post.com/",{
      method:"POST",
      headers:{
         'Content-type':'multipart/form-data'
      }
      body: formData
   })

})
```

Every input that has a valid value will be sent to the server and upload the data from it. Commonly, you dont need to specify a headers because it will be automatically set by the browser

### CORS (Cross Origin Resource Sharing)

Is set by browser to prevent a request from another origin, unless specified. It also serves a layer of security measure from the server to only allow a certain origin to get access from their data.

> if you're sending request on the same origin like: `web.com/api`, CORS is not needed as it falls to `same-origin`

The server can set `ACCESS-Control-Allow-origin: *` to allow any access to the said server, the `*` made it possible. If the server set a certain origin like `http://127.0.0.1:3000` , only the specified origin will have access to the server (it can be other origin).

The method `GET` , `HEAD` and some of `POST` don't necessarily need to set CORS as it is set by browser by default. Since it is set by default you can only set a certain/default headers, these are also known as Simple Request, and these are:

- `Accept` : What file types that you're going to accept
- `Accept-language` or `content-language` : If given a choice what is your preferred language but only standard values.
- `Content-type` : what type of content that you're going to send to the server. Accepted content-types are:
  - `text/plain`
  - `multipart/form-data` : for files upload
  - `application/x-www-form-urlencoded` : for form data
- `Range` : if you only want parts of the data.
- Whole data - means if you request it must be the entire data that you need and not `ReadableStream`, this will trigger CORS

#### What triggers CORS ?

As mentioned above if it is beyond the Simple request, CORS will trigger. What will happen is browser will check for the headers and determines if the request is simple or not. If it is not a simple request the browser will now trigger a request method `OPTIONS` which includes the CORS options and those are:

- Access-Control-Allow-origin - if the origin is the same origin of where the request came from to what the server specified. E.g. `ACCESS-Control-Allow-origin: "http://127.0.0.1:3000"`, if your origin is this one then you're clear.
- Access-Control-Request-Method - What method is allowed. The browser will check if the method that you specify is allowed by the server or is included in the allowed set by the server. E.g. `Access-Control-Allow-Methods: POST, GET, OPTIONS`
- Access-Control-Request-Headers - what headers are allowed. The browser will check if the headers that you requested is the same to what is being set by the server for example : `Access-Control-Allow-Headers: Content-Type`. This will allow the likes of `application/json` to be accepted by the server.

Some Request headers name:

- Cache-Control
- content-language
- Content-length
- content-type
- Expires
- Last-modified
- Pragma

#### Request Modes

This can be set as a second argument along with the method, headers , body, cache. Although it is set to `mode:cors`, you can still specify the mode that you want.

#### Modes

- `cors` - will trigger CORS if the request goes beyond the simple request.
- `same-origin` - will not trigger a CORS if the request cmae from the same origin.

  > different subdomain doesnt fall to the `same-origin` category and will be handled as a different origin, thus CORS will trigger.

- `no-cors` - will bypass the CORS , meaning that if the request is not a simple request and this is set to no-cors the browser will still send the request to the server but the browser will response an `opaque response`. For some circumstances, that you just want to send data to the server without needing the response you can set this up as it will skip CORS thus will skip the OPTIONS fetch method by the browser in doing so the fetch will be faster but the tradeoff is you wont be able to access the response

- `navigate` - this is set by the browser only

#### Opaque Response

It is a response set by the browser if the no-cors mode is being set. It means that the data is successfully been received but you cant access it because the no-cors is being set. These data is being blocked by the browser but not all data is being black, there are certain types of data are accessible. If it is a data that you can put in a page, not all but most of it, you can get access to it. If it is a `text response` that you can put in `<script>` or `<link rel="stylesheet">` tags you can get access to it. If it is an image for `<img>`, video for `<video>` and audio for `<audio>`, these are allowed. If it is for `<iframe>`, `<embed>` or `<object>`, it is also allowed.

Some not allowed Data are: if for `<Canvas>` , Web Fonts, cache storage, and more.
[Docs for CORS from MDN](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy#cross-origin_network_access)

> the status code for opaque reponse is always status `0`.

### Sequence Fetching and multiple fetching

When calling fetch api you can sequence it with another fetch or you can concurrently call multiple fetch, you can even link fetch calls that depends on each other.

- Linking multiple fetch that depends on each other

```JavaScript
   fetch("url1").then(res=> {
      if(!res.ok){
         throw new Error("error")
      }

      return res.json()
   }).then(data=>{
     return fetch(`url2/${data.id}`)
   }).then(res2=>{
      if(!res2.ok){
         throw new Error("error")
      }
      return res2.json()
   }).then(data2=>{
      return fetch(`url3?id3=${data2.id}`)
   }).then(res3=>{
      if(!res3.ok){
         throw new Error("error")
      }
      return res3.json()
   }).then(data3=>{
      return fetch(url4,{
         headers:{
            x-data-id: data4.userId,
            x-data-value: data4.value,
            Authorization: `Bearer ${your-api-key}`
         }
      })
   }).then(res4=>{
      if(!res2.ok){
         throw new Error("error")
      }
      return res4.json()
   }).then(data4=>{
      console.log(datajson4,"mydata4")
   }).catch(console.warn)
```

This can be improve but this is to show the concept of how you can chain multiple fetch that depends on each other. You can also create a helper for the .then() since it repeats itself multiple times. This can also be easily handled with `async/await fetch method`

```JavaScript
// a helper to avoid repeating codes
const fetchWithErrorHandling = async (url, options = {}) => {
  const response = await fetch(url, options);
  if (!response.ok) {
    throw new Error(`HTTP error! status: ${response.status}`);
  }
  return response.json();
};

const apiChain = async () => {
  try {
    const data1 = await fetchWithErrorHandling("url1");

    const data2 = await fetchWithErrorHandling(`url2/${data1.id}`);

    const data3 = await fetchWithErrorHandling(`url3?id3=${data2.id}`);

    const data4 = await fetchWithErrorHandling("url4", {
      headers: {
        "x-data-id": data3.userId,
        "x-data-value": data3.value,
        "Authorization": `Bearer ${your-api-key}`
      }
    });

    console.log(data4, "mydata4");
    return data4;
  } catch (error) {
    console.warn("An error occurred:", error.message);
    // Here you could add more specific error handling if needed
  }
};
```

- Multiple fetch that is independent to each other
  For multiple fetch() that doesn't need to wait for the other fetch's response there are multiple helpers with Promise object. As using multiple fetch can slow down your code since you have to wait for each fetch but instead using Promise object you can run the fetch concurrently. These helpers have different method that you can use.
- Promise.all
  This helper will received all the fetch/promise and run them all and then returns an array of the resolved promised/fetch. Whatever the order when you passed the fetch as an array will be on the same index/order when you received it as It doesn't matter which of the promise got resolved first.

  ```JavaScript
  // the same helper to make the code a bit cleaner
  const fetchWithErrorHandling = async (url, options = {}) => {
  const response = await fetch(url, options);
  if (!response.ok) {
     throw new Error(`HTTP error! status: ${response.status}`);
  }
  return response.json();
  };

  function fetchPromiseAll(){
     const urls = ["url1","url2", "url3"]
     try{
        const result = await Promise.all(urls.map(url=>fetchWithErrorHandling(url)))
        result.forEach((data,index)=>{
         console.log(`Result from ${urls[index]} is ${data}`)
        })
     }
     catch(e){
        console.error(e)
     }
  }
  ```

The Caveat with Promise.all is if one of the promise in an array fails it will fail all of the promises/fetch. So this helper is only great if you need all of the promises to respond.

> [for more info about promise.all](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/all)

- Promise.allSettled
  In the case that you need atleast a resolved fetch/promise and still want to have a concurrent promises/fetch running, allSettled, is the helper that is best for this. What it does is the same to Promise.all but if not all of the promise got resolve instead of throwing error for all of the promise/fetch it will separate the unfulfilled promise and the resolved one, that way you can still access the resolved promise and can handle the unfulfilled one. This is useful if you want to show something in the UI/frontend even tho not all of the fetch resolved.

```JavaScript

  function fetchPromiseAll(){
     const urls = ["url1","url2", "url3"]
     try{
                                                      //the same helper
        const result = await Promise.allSettled(urls.map(url=>fetchWithErrorHandling(url)))

      console.log("All requests completed. Results:");
         results.forEach((result, index) => {
            if (result.status === 'fulfilled') {
            console.log(`Request to ${urls[index]} succeeded:`, result.value);
            } else {
            console.log(`Request to ${urls[index]} failed:`, result.reason);
            }
         });

         // Process successful results
         const successfulData = results
            .filter(result => result.status === 'fulfilled')
            .map(result => result.value);
         console.log("Combined data from successful requests:", successfulData);

         // Process failed results
         const failedRequests = results
            .filter(result => result.status === 'rejected')
            .map((result, index) => ({
            url: urls[index],
            error: result.reason
            }));
         console.log("Failed requests:", failedRequests);
     }
     catch(e){
        console.error(e) // for unexpected error
     }
  }
```

In this code, the resolved and failed fetch can be process separately, this way you have more control on the returned fetched and you can display a UI for failed and successful fetch if you want.

> [more docs for Promise.allSettled from MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/allSettled)

- Promise.race
  promise.race is different. This helper is better if you need the data which is resolved first, like a race. You can fetch/run multiple promise and whichever resolved first is your value and more importantly, if one of the promises failed first you wont get a value but an Error instead. So it is a race between all of the promises/fetch being run and a race between resolved and reject, whichever returns first is your value or Error

```JavaScript
function promiseRace(){

  const urls = ["url1","url2", "url3"]
  try {
    const data = await Promise.race(urls.map(url => fetchWithErrorHandling(url)));
    console.log("First successful request data:", data);
  } catch (error) {
    console.error("First request to fail:", error.message);
  }

}
```

> [more info for promise.race](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/race)

### Things that you can do with the response headers

Fetch api is a Thenable method where you can use .then or just await. But Why is it necessary to do atleast two await ? It is because the headers always comes first.It is a fast response from the server to know and be able to read what the headers are that is why you can almost always see a pattern where the `res.ok` is being checked first to see the https response and by then the `res.body` is being sent and you can now transform the res.body. This pattern is to get a response right away to know the fate of your res.body like if `!res.ok` you can handle it early before receiving the response and can throw an error early too.

The advantage of getting the headers first is you can do a progress loading to the UI. Since header returns first you can get the Content-length, if available and setup a reader on the response.body `response.body.getReader()`. Now on the reader you can user .read() `reader.read()` and it returns two object which are done, value. The `done` is a boolean will become true when the body is done fetching while `value` have an object called .length which reads the current length value progress of the fetched body. This way you can make a progress base on the value.length and content-length, the value.length must keep on updating to know the progress of the streamed body .

This can be done as long as you have content-length and also applicable on json, text and blob

Example:

- blob

```JavaScript
async function fetchblob(){
   const response = await fetch(url/blob)
   cosnt contentLength = response.headers.get('content-length')
   const reader = response.body.getReader()

   let receivedLength = 0 // init for the progress
   const chunks = [] // data streamed will be push here

   let inProgress = true
   while(inProgress){
      const {done, value} = await reader.read()

      if(done) {
         inProgress = false // to stop the loop
      }
      chunks.push(value)
      receivedLength += value.length
      console.log(`length ${receivedLength} of ${contentLength}`)
      // or you can do a UI progress
   }

   let byteArray = new Uint8Array(receivedLength)
   let position = 0
   for(let chunk of chunks){
      byteArray.set(chunk,position)
      position+=chunk.length
   }

    //for an image/blob
    let blob = new Blob([byteArray], { type: 'image/jpg' });
    let url = URL.createObjectURL(blob);
    let img = document.getElementById('pic');
    img.src = url;
    img.alt = imgstr;

   // for text
   let text = new TextDecoder('utf-8').decode(byteArray);
    console.log('Text content:', text.substring(0, 100) + '...'); // Show first 100 characters


   // for json
   // Convert byte array to text, then parse as JSON
    let text = new TextDecoder('utf-8').decode(byteArray);
    let jsonData = JSON.parse(text);
    console.log('JSON data:', jsonData);

}

```

> [more info about getReader](https://developer.mozilla.org/en-US/docs/Web/API/ReadableStream/getReader)

### Aborting a fetch request

While not required it is a good to the UX to let them know that the fetch takes long or at least give them an option to cancel the request. This is what AbortController() can do, this can be passed in the signal property of the second parameter from the fetch(). If you abort a fetch it will be considered as an Error and will be catch if you setup a trycatch or .catch().

```JavaScript
   const controller = new AbortController() //setup the AbortController
   const btn = document.getElementById("abort-btn")
   btn.addEventListerner("click",()=>{
      controller.abort() // pass it in the button so you can abort it
   })

   fetch(url,{signal:controller.signal}).then(res=>{
      if(!res.ok){
         return response.blob
      }
   })

```

With this you can setup or show the button after a second or if the fetch takes so long to give the user an option to abort

### References and resources used (others mentioned above)

- [Ten Steps to Mastering the Fetch api](https://www.youtube.com/watch?v=2sQ9xiEAXNo)
- [Why does JavaScript's fetch make me wait TWICE?](https://www.youtube.com/watch?v=Ki64Cnyf_cA&list=PLzjv0fR3AYQnnv3KOAn7vcNgGOzbh-cSb&index=22)
- [abort controller](https://www.youtube.com/watch?v=ZqerXMzt-EY)
