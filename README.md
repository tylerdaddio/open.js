# only.js
```html
<script src="only.js"></script>
```

- [About](#About)
- [Getting Started](#Getting-Started)
- [Problem: External Links](#External-Links)
- [Directives](#Directives)
- [Modules](#Modules)
- [Documentation](#Documentation)
- [License](#License)
- [Footnotes](#Footnotes)



## About

only.js is a simple JS library for making sitemap-driven websites. It is written entirely using native JavaScript. No dependencies, and no requirement to use any particular framework. Only you; Only JS™. †

To use only.js, just include the core module.
```html
<script src="only.js"></script>
```


## Getting Started

Getting started with only.js is easy!

As we want to use only JS, we'll have a bare-bones `index.html`:
```html
<DOCTYPE html>
<html>
<head>
  <script src="only.js"></script>
  <script src="myscript.js"></script>
</head>
<body></body>
</html>
```

In `myscript.js`, write the following:
```js
only.init();

window.onload = () =>
{
  only.load("");
};
```

The call to `only.init` initializes the website with a default sitemap. The `only.load` function loads the page at the root of the sitemap. The call to `only.load` is made inside of `window.load` to give time for the DOM to initialize. After all, loading a page will probably modify the DOM!

That's it! Our simple site is working! Granted, this seems like more work than just throwing some text in the `<body>` of our `index.html`. The real power of only.js comes when we start building our own sitemap!

To do so, we make a simple object and pass it into `only.init`:
```js
/* This is our sitemap. */
var mysitemap = {
  "..": () => {
    document.body.innerHTML = "This is my root page!";
  }
};

/* Initialize only.js with our sitemap. */
only.init(mysitemap);

window.onload = () =>
{
  /* Load our website's root page. */
  only.load("");
};
```

The `..` property of the `mysitemap` object specifies the function to run when the containing page is loaded. In this case, it is the function that loads the root page of the site.

Let's create a new page! To do so, we'll need to modify the `mysitemap` object.
```js
var mysitemap = {
  "..": () => {
    document.body.innerHTML = "This is my root page!";
  },
  "about": {
    "..": () => {
      document.body.innerHTML = "This is my about page!";
    }
  }
};
```

Now just call `only.load("about")`, and our new page will load! Notice that all we did was nest another object, `about`, inside of `mysitemap` then added a new `..` property to it, again mapping to a function. When the `about` page is loaded, that new function is called.

However, we probably want to provide some way to swtich between our root page and our `about` page. We'll create a simple page search box for this. After our call to `only.init`, let's add a text element to the DOM:

```js
var searchbar = document.createElement("input");
searchbar.oninput = (input) => {
  only.load(input.text);
};
var pagecontent = document.createElement("div");

document.body.appendChild(searchbar);
document.body.appendChild(pagecontent);
```

(It should be tempting to desire a DOM API less bulky than that offered with native JS. [d3.js](https://d3js.org) is one popular, feature-rich alternative. Of course, you could also put all the HTML stuff in your HTML files, but that wouldn't be Only JS™. †)

Now just change those calls to `document.body` to `pagecontent`, and give it a try! Type in `about`, and... `404 page not found`? What? Where did my searchbar go?

This is our first encounter with the dreaded 404 response! In general, a 404 response occurs when a site attempts to load a page that doesn't exist. When we typed `a`, it tried to `only.load` the page `a`, which doesn't exist, so it ran the default 404 function. Unfortunately for us, the default 404 function writes text directly into `document.body`.

We want to change the default 404 function so it doesn't break everything. To do so, add a new `.404` property to `mysitemap`.
```js
var mysitemap = {
  "..": () => {
    pagecontent.innerHTML = "This is my root page!";
  },
  ".404": (path, err) => {
    pagecontent.innerHTML = "No page at /" + path.join("/");
  },
  "about": {
    "..": () => {
      pagecontent.innerHTML = "This is my about page!";
    }
  }
};
```

Notice that the `.404` property looks similar to the `..` property: it has a function mapped directly to it. only.js refers to these properties starting with `.` and mapping to a function *directives*. So the `..` directive directs only.js to run that function when the containing page is the target. Similarly, the `.404` directive says what to do when a page cannot be loaded.

Also notice that the `.404` directive can receive up to two arguments: a path and an error. The path is the path that was tried. The error is that generated when a page fails to load. Here we are using the path to update `pagecontent` with the page we are trying (unsuccessfully) to load.

With the addition of the `.404` directive, try to load the `about` page again. Type `about` in the `searchbar`, and the `about` page should come up.

Let's add two more pages nested under the `about` page. Here's the new `mysitemap`:
```js
var mysitemap = {
  "..": () => {
    pagecontent.innerHTML = "This is my root page!";
  },
  ".404": (path, err) => {
    pagecontent.innerHTML = "No page at /" + path.join("/");
  },
  "about": {
    "..": () => {
      pagecontent.innerHTML = "This is my about page!";
    },
    "me": {
      "..": () => {
        pagecontent.innerHTML = "Hi. I'm a super duper coder guru. UwU";
      }
    },
    "you": {
      "..": () => {
        pagecontent.innerHTML = "You? I have no idea who you are."
      }
    }
  }
};
```

Now try typing both `about/me` and `about/you` into the `searchbar`. Each page should load once its full path is typed.

Wait, before you go! One more thing: notice that the URL in the browser is changing when we type in our searchbar. This occurs because only.js modifies the browser's history to simulate what normally happens when following hyperlinks. Yes, the website is fully navigable using the back and forward arrows.

However, notice that we cannot yet go directly to any page just using the URL. Try it: refresh the page after typing `about` into the `searchbar`. Things will probably get weird.

To fix this problem, we need to ensure we open the correct page when the window loads. Let's modify the `window.onload` function:
```js
window.onload = () =>
{
  only.load(window.location.pathname);
};
```

Unfortunately, some things are out of the control of only.js. The websever may redirect to a special 404 page when the path isn't recognized. The way to fix this is to ensure the served page is always `index.html` and that the URL is preserved. For examples of how to do this, see [External Links](#External-Links).

Here is the final demo code:
```js
/* This is our sitemap. */
var mysitemap = {
  "..": () => {
    pagecontent.innerHTML = "This is my root page!";
  },
  ".404": (path, err) => {
    pagecontent.innerHTML = "No page at /" + path.join("/");
  },
  "about": {
    "..": () => {
      pagecontent.innerHTML = "This is my about page!";
    },
    "me": {
      "..": () => {
        pagecontent.innerHTML = "Hi. I'm a super duper coder guru. UwU";
      }
    },
    "you": {
      "..": () => {
        pagecontent.innerHTML = "You? I have no idea who you are."
      }
    }
  }
};

/* Initialize only.js with our sitemap. */
only.init(mysitemap);

/* Add some elements. */
var searchbar = document.createElement("input");
searchbar.oninput = (input) => only.load(searchbar.value);
var pagecontent = document.createElement("div");
document.body.appendChild(searchbar);
document.body.appendChild(pagecontent);

window.onload = () =>
{
  only.load(window.location.search);
};
```

## External links

Loading `yourdomain.com/about/me` from an external site may not load the page you expect. Most likely, your websever is to blame. For example, it may serve up a special 404 page when the path doesn't exist in the file system of the website. The way to fix this is to ensure the served page is always `index.html` and that the path is preserved in the URL.

This section details how to make this happen using a variety of web servers/hosts.

- [FastMail](#FastMail)
- [Others?](#Others?)

### FastMail

FastMail lets you place a `404.html` file in the root directory of the website folder. This page is served whenever a path is not recognized. In `404.html`, add the following script:
```js
window.location.replace("yourdomain.com?page=" + window.location.pathname);
```
Then when loading the first page do the following:
```js
var path = "";
if(window.location.search !== "")
  path = window.location.search.split("?page=")[1];
only.load(path);
```

### Others?

If you use different web server/host and know how to solve this problem, please contribute!



## Modules

only.js has optional modules available to make the task of creating your sitemap-driven website even easier. Just include the relevant `only-*.js` in your HTML, and enjoy!

- [core](#core) - For only.js!
- [tabs](#tabs) - For easily managing links to root pages!
- [journals](#journals) - For using JSON to power streams of objects!
- [utils](#utils) - For some useful functions!



## Documentation

Documentation is always a work-in-progress. Provided here are the public functions and variables provided by only.js and its modules.

### core

##### `only`
The object containing everything only.js needs and provides.

##### `only.init(sitemap)`
Initializes only.js with a sitemap. This **must** be invoked before using `only.load` and before initializing any other modules.

##### `only.maketitle(function(path))`
Provide a function to use when changing the window title following a page load.

##### `only.load(path)`
Loads the given path.


### tabs

##### `only.tabs`
The object containing everything in the only.tabs module.

##### `only.tabs.init()`
Initializes the only.tabs module. This **must** be invoked before using any functionality of only.tabs and after invoking `only.init`.

##### `only.tabs.select(tabName)`
Selects the tab with the given name.

##### `only.tabs.onselect = function(tabObj)`
The function to invoke when a tab is selected.

##### `only.tabs.ondeselect = function(tabObj)`
The function to invoked when a tab is deselected.


### journals

##### `only.journals`
The object containing everything in the only.journals module.

##### `only.journals.init()`
Initializes the only.journals module. This **must** be invoked before using any functionality of only.journals and after invoking `only.init`.

##### `only.journals.create(name, url)`
Creates a new journal with the given name. The URL specifies the location of the JSON representing the journal. This function does not fetch from the URL. To do so, see [`only.journals.update`](#`only.journals.update(name)`).

##### `only.journals.update(name)`
Fetches the JSON living at the URL provided upon journal creation. This is used to update the internal representation of the journal.

##### `only.journals.search(name, searchTerm, toSearch, caseSensitive=false)`
Searches all entries in the journal's current JSON for the search term. Only those properties provided in the `toSearch` array are searched within each entry, and only `string`s and arrays of `string`s are searched. The search can be made case-sensitive by setting `caseSensitive` to `true`.


### utils

##### `only.utils`
The object containing everything in the only.utils module.

##### `only.utils.fetch(url, useCache=true)`
Fetches from a URL optionally caching the response received. Upon invokation with a novel URL, an actual fetch is performed, but upon subsequent invokations with the same URL, a clone of the original `Response` is returned as long as `useCache` is not set to `false`.

##### `only.utils.fetchAndFill(url, node, useCache=true)`
Fetches from a URL, gets the text from the body of the response, then sets the inner HTML of the node. This function uses [`only.utils.fetch`](#`only.utils.fetch(url,-useCache=true)`) under the hood and so also can cache the received `Response`.

##### `only.utils.emptyNode(node)`
Removes all children from a DOM node, if any.



## License
See the [License](/LICENSE).



## Footnotes

†: Not actually trademarked.