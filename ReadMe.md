# Primitive JavaScript

JavaScript started in a web browser <script>. First, let's look at this most basic usage.

## JavaScript execution model

Since it is related to the story of synchronous/asynchronous loading of modules, we will organize the JavaScript execution model in advance.

JavaScript is single threaded. In other words, the following code will not be interrupted by another JavaScript process.

```js
window.counter = 0;
console.log(`counter = ${window.counter}`);
window.counter += 1;
// ←never separate JavaScript is executed in this position
console.log(`counter = ${window.counter}`); // ←only 1
```

This is a problem if the script runs for a long time. The real problem is with infinite loops. If something goes into an infinite loop, the browser can't continue doing anything about that page (because you can't run other JavaScript code at the same time).

If you have a synchronous sleep function, you'll have a similar problem.

```js
// If there is synchronous sleep function ...... 
console.log("foo");
sleep(1000); // ←During this run, the other JavaScript code can not execute 
console.log("bar");
```

Therefore, the above synchronous `sleep` function does not actually exist in JavaScript, and it is `setTimeout` configured to execute the next process asynchronously by.

```js
console.log("foo");
// setTimeout itself ends in an instant, the "baz" after one second is displayed 
setTimeout(() => {
  console.log("baz");
}, 1000);
// because setTimeout itself ends in an instant, "bar" is displayed immediately 
console.log("bar");
```

Later , the time-consuming library functions, including XMLHttpRequest , which built the ajax, inherit essentially the same design. However, if you write the callback explicitly as above, you will get into callback hell and the code will not be visible to the actual logic, so Promise which is library support and async function which is syntax support were introduced. It was.

## Primitive JavaScript

Originally, JavaScript was executed in a browser, there was no module system, and the value was defined in the global space unless otherwise specified.

```js
<!-Load and execute script.js-> 
<script type="text/javascript" src="./script.js"></script>
<!-Execute the script embedded in HTML -> 
<script type="text/javascript"><!--Alert("bar");//-> </ script> 
<noscript> Please enable JavaScript. </ noscript>
```

`script.js`

```js
alert("foo");
```

JavaScript on the browser has `window` a special variable called, and variables and functions defined in the global context are `windowt` reated as members of

```js
var x = 2;
function f() { return 3; }
console.log(window.x); // => 2
console.log(window.f()); // => 3
```

## Synchronous loading of script tags

There was in the original JavaScript `document.write`.

```js
<script>
  document.write("<p");
</script>
>hoge</p>
```

Due to this horrifying feature, the browser had `<script>` to block HTML parsing until the tag's JavaScript had finished loading and executing. Since parsing does not proceed, many operations such as loading and executing JavaScript and loading images will be blocked as a result.

In fact, you'll want to do a lot of the work you do with JavaScript after the document is all loaded. (For example, `document.getElementById("root")` the teeth `<div id="root"></div>`should want to do after that is loaded.) This kind of treatment had to be written in the following manner

```js
window.onload = function() {
  var root = document.getElementById("root");
  // with root processing ...
};
```
In that case `<script>`, putting the tag near the beginning of the document does not contribute to the script execution start timing. Rather, the disadvantage of blocking the loading of document components other than scripts (HTML body, images, etc.) is greater, so for a while `<script>` it was desirable to put it at the end of the document as follows .

```js
  <!-- ... -->
  <script src="jquery.js"></script>
  <script src="script.js"></script>
</body>
```

## Asynchronous reading of script tags

Old behavior of the above is improved in HTML5, `defer`, `async`, `type="module"` any of when is specified now so as not to block the parser. These relationships are as follows.

- `type="module"` Teeth `defer`, including the implicit
- `async` Is `defer` stronger
- `defer` Since the appearance is older, `async` when you specify, you may `defer` also specify it at the same time and aim for fallback

I'll omit the difference between async and defer.

Of course, if you specify these attributes, you will `document.write` not be able to use them.

For browsers that support asynchronous loading of script tags, it may be more efficient to specify what should be loaded early. The script tag is back towards the beginning of the document again.

```js
  <!-- ... -->
  <script src="jquery.js" defer></script>
  <script src="script.js" defer></script>
</head>
<body>
  <!-- ... -->
```

## Turn off scope (IIFE)

In the original JavaScript, `var` there was a problem that even if you declare a variable with much effort , if it is a global scope, it will be a global definition.

To avoid this, a pattern (IIFE; Immediately Invoked Function Expression) was established in which an anonymous function is created and work is performed in it.

```js
var my_library = (function() {
  // Work to create my_library
  // Even if you use var here, it will not leak to the outside
  return my_library;
})();
```

As a result, a library in the form of "defining a very small number of identifiers globally, about 1 or 2" has appeared, and it has become cleaner than the way JavaScript used to be. A typical example is the library/framework called jQuery, which flourished around 2010-2015. `jquery.js` When you load a global `$` and `jQuery` defines the two values. (These two are the same)

The library that was put together in one file in this way was sometimes copied to the same server as the application and used, and sometimes it was used by referring to the one on the public server such as jsDelivr.

This method had some drawbacks.

- It didn't work at all as a modular system (cannot describe dependencies between multiple files)
- The package system, do not you have to collectively each package in one file, it can not manage an indirect dependency, also that there is a possibility that the name that you want to export to collision 5 , etc. There was a drawback.

## JSmin

Going back further in 2001, Douglas Crockford created JSmin . This is a simple program written in C language, and by removing spaces and comments from the input JavaScript code, it was possible to output an equivalent JavaScript code with a smaller number of characters. It can be said that it is the origin of the current Minifier represented by YUI Compressor , UglifyJS , Closure Compiler , and the idea of ​​"converting JavaScript to JavaScript".

