# App Module

<ul class="nav">
    <li><a href="#introduction">Introduction</a></li>
    <li><a href="#defining-pages">Defining Pages</a></li>
    <li><a href="#initializing-navigator">Initializing Navigator</a></li>
    <li><a href="#rendering-pages">Rendering Pages</a></li>
    <li><a href="#navigation">Navigation</a></li>
    <li><a href="#page-params">Page Params</a></li>
    <li><a href="#current-page">Current Page</a></li>
    <li><a href="#events">Events</a></li>
</ul>

## Introduction <a name="introduction"></a>

`Navigator` is a page navigator, commonly known as "router" in many JS frameworks. Unlike the traditional routers, Koti Cloud Navigator follows a tree page structure. When you press the back button, you go to the parent page, or one level up the page tree.

This kind of navigation is what you'd expect from an app. The traditional routers are really meant for web pages and don't handle back navigation well when used in web apps. 

If you are converting an existing app to a Koti Cloud app and already use a traditional router - you can keep using that, but we really encourage you to at least give Navigator a try for better user experience.

## Defining Pages <a name="defining-pages"></a>

You need to create a `.js` file to define the list of your app's pages. We'll call it `pages.js`. Below is a very basic example.

```javascript
import HomePage from '../svelte/pages/HomePage.svelte';
import ChildPage from '../svelte/pages/ChildPage.svelte';

export default {
    home: {
        component: HomePage,

        children: {
            'child-page': {
                component: ChildPage,
                title: 'Child Page',
            },
        }
    },
};
```

For each page you need to specify a key, a component file, an optional title and an optional list of child pages. The depth of the tree is not limited. If you press back button while on the `child-page` - you'll be sent to the `home` page. If you press back button while on the `home` page - the app will close.

### Svelte.js Pages

A Svelte page is simply a default Svelte component. You don't need to add any specific additional code inside.

### Vanilla JS Pages

A Vanilla JS page is a ES6 module that simply exports the page's HTML template. Example:

```javascript
export default `
    <h1>Home Page</h1>

    <sdk-page-link to="child-page">
        <button>Child Page</button>
    </sdk-page-link>
`;
```

## Initializing Navigator <a name="initializing-navigator"></a>

To initialize navigator you need to tell it what pages your app has, then navigate to the home/index page of the app.

```javascript
import { Navigator } from '../../../sdk/v0.4.x/sdk.js';
import pages from './pages.js'; // The file we've created in previous step

Navigator.setPages(pages);
Navigator.goTo('home');
```

## Rendering Pages <a name="rendering-pages"></a>

### Using Svelte Component

The SDK provides a Svelte component for easy page rendering. First import it inside the component where you want to render your pages:

```javascript
import { PageRenderer } from '../../../sdk/v0.4.x/svelte-components.js';
```

Then add it to the HTML template of the same component:

```html
<PageRenderer />
```

The component will automatically render the current Navigator's page.

### Using Vanilla JS Component

The SDK provides a Vanilla JS component for easy page rendering. You'll need to import it anywhere in your code, the sooner the better. Then you'll need to define a global custom DOM element.

```javascript
import { PageRenderer } from '../../../sdk/v0.4.x/js-components.js';

window.customElements.define('sdk-page-renderer', PageRenderer);
```

Finally you can use the component in your HTML wherever you want your pages to be rendered:

```html
<sdk-page-renderer></sdk-page-renderer>
```

The component will automatically render the current Navigator's page.

### Custom Solution

You can hook into the Navigator's <a href="#events">events</a> and define your own rendering logic. This way you'll also be able to define your own page component format. 

## Navigation <a name="navigation-methods"></a>

### Page Links Svelte Component

The SDK provides a Svelte component to easily add a link to any page anywhere you like. First import it inside your Svelte component:

```javascript
import { PageLink } from '../../../../../sdk/v0.4.x/svelte-components.js';
```

Then add it to the HTML template of the same component:

```html
<PageLink to="create-note" params={ { category } }>
    <button class="btn">write a new one</button>
</PageLink>
```

Let's take a look at the example above. The `to` attribute should specify a page key from your `pages.js` file. The optional `params` attribute let's you pass an object with custom data with the navigation event (see <a href="#page-params">Page Params</a>).

You need to provide the HTML of the button/link between the opening and closing tag (the component's default slot).

### Page Links Vanilla JS Component

The SDK provides a Vanilla JS component to easily add a link to any page anywhere you like. You'll need to import it anywhere in your code, the sooner the better. Then you'll need to define a global custom DOM element.

```javascript
import { PageLink } from '../../../sdk/v0.4.x/js-components.js';

window.customElements.define('sdk-page-link', PageLink);
```

Now you can use the component in your HTML wherever you need:

```html
<sdk-page-link to="create-note" params="{ category }">
    <button class="btn">write a new one</button>
</sdk-page-link>
```

Let's take a look at the example above. The `to` attribute should specify a page key from your `pages.js` file. The optional `params` attribute let's you pass an object with custom data with the navigation event (see <a href="#page-params">Page Params</a>).

You need to provide the HTML of the button/link between the opening and closing tag (the component's default slot).

### Programmatic Navigation

#### `Navigator.goTo(page, params = {})`

Go to a page anywhere in the page tree.

`page` is either a page key or a page object.

`params` is an object with custom data that will be passed with the navigation event.

#### `Navigator.goBack()`

Go one page up the navigation tree. Navigates to the parent page of the current page. If there's no parent page - Navigator will attempt to close the app.

## Page Params <a name="page-params"></a>

As you've seen from the previous section, you can pass params with each navigation event. A params object is just a simple JS object with whatever data you want to pass. You'll be able to access this data in multiple places. Read the following section for details.

Params are useful. For example, you might want to display different data in a single page component depending on a DB id, category, tag, or anything like that.

### Default Page Params

You can specify default page params when defining the list of pages in your `pages.js` file. These params will be attached to any navigation event even if you don't specify any custom params when navigating. Imagine we have the following pages for a notes app:

```javascript
import HomePage from '../svelte/pages/HomePage.svelte';
import NotesListPage from '../svelte/pages/NotesListPage.svelte';

export default {
    home: {
        component: HomePage,

        children: {
            'notes-by-category': {
                component: NotesListPage,
                title: 'JavaScript',
                params: {
                    type: 'category',
                },
            },
            'notes-by-tag': {
                component: NotesListPage,
                title: 'JavaScript',
                params: {
                    type: 'tag',
                },
            },
        }
    },
};
```

In the example above we have two different pages to display a list of notes grouped by either categories or tags. There would be two page links leading to each of the pages respectively. Both pages use the same page component and will be able to display different (but same in essence) data based on the `type` param.

Now, if you wanted to display the list of notes in a specific single category or tag, you would send it's name as a dynamic page param when navigating, for example:

```javascript
Navigator.goTo('notes-by-tag', { tag: 'javascript' });
```

If you didn't specify default params and had a single page called `notes`, your navigation would look like this:

```javascript
Navigator.goTo('notes', { type: 'tag', tag: 'javascript' });
```

## Current Page <a name="current-page"></a>

You can get the current page object anywhere in your code like this:

```javascript
const page = Navigator.getCurrentPage();
```

After that you can get page params (both default and dynamic) by accessing the `params` property. Considering the example above, after navigating to a page like this...

```javascript
Navigator.goTo('notes-by-tag', { tag: 'javascript' });
```

... you could then access page params like this in the `notes-by-tag` page's component `NotesListPage`:

```javascript
const page = Navigator.getCurrentPage();

console.log(page.params.type);  // = 'tag'. A default param. Always present.
console.log(page.params.tag);  // = 'javascript'. A dynamic param. Could be empty (`undefined`).
```

## Events <a name="events"></a>

### Dedicated Page Events

Each page can have a single dedicated event of each type attached to it. An event can be attached directly to the page declaration in your `pages.js` file, for example:

```javascript
import MyPage from '../svelte/pages/MyPage.svelte';

async function beforeMyPageEntering(from, to) {
    if (!to.params.category) {
        console.error('You need to specify a category');

        return false;
    }

    return true;
}

export default {
    'my-page': {
        component: MyPage,
        title: 'My Page',
        beforeEntering: (from, to) => beforeMyPageEntering(from, to),
    },
};
```

You can also attach these events to the current page in the page's component file. For example:

```javascript
let changesMade = false;

const page = Navigator.getCurrentPage();

page.beforeLeaving((currentPage, nextPage) => {
    if (changesMade) {
        console.log('Please save your work first.');

        return false;
    }

    return true;
});
```

#### `bool beforeLeaving(from, to)`

This event is fired before leaving the `from` page. Your callback should return `false` to prevent the operation and `true` to continue.

#### `bool beforeEntering(from, to)`

This event is fired before entering the `to` page. Your callback should return `false` to prevent the operation and `true` to continue.

#### `void afterEntering(from, to)`

This event is fired after entering the `to` page.

### Global Navigator Events

There are also global Navigator events which are more flexible than the dedicated page events. Global event listeners are attached directly to the Navigator like this:

```javascript
let changesMade = false;

Navigator.beforeLeavingEach((from, to) => {
    if (from.name = 'my-page' && changesMade) {
        console.log('Please save your work first.');

        return false;
    }

    return true;
});
```

#### `bool beforeLeavingEach(from, to)`

This event is fired before leaving any page (`from` page). Your callback should return `false` to prevent the operation and `true` to continue.

#### `bool beforeEnteringEach(from, to)`

This event is fired before entering any page (`to` page). Your callback should return `false` to prevent the operation and `true` to continue.

#### `void afterEnteringEach(from, to)`

This event is fired after entering any page (`to` page).