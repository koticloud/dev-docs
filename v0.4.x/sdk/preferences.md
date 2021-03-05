# Database

<ul class="nav">
    <li><a href="#introduction">Introduction</a></li>
    <li><a href="#get-one">Get One Preference</a></li>
    <li><a href="#get-all">Get All Preferences</a></li>
    <li><a href="#set-one">Set One Preference</a></li>
    <li><a href="#set-bulk">Set Multiple Preferences</a></li>
</ul>

## Introduction <a name="introduction"></a>

The SDK has a `Preferences` class that is useful when you want to store app preferences (settings) that are synced across devices (browsers). It uses the `DB` class internally and syncs automatically when a preference is set. Like with the `DB` class, you shouldn't use `Preferences` directly. Use the instance from the global `App` instance instead: `sdk.prefs`.

## Get One Preference <a name="get-one"></a>

To get a single preference value by key, use the `get(key, defaultValue = null)` method.

```javascript
let theme = sdk.prefs.get('theme', 'light');
```

In this case we're trying to get a preference with the key "theme". If this key doesn't exist, the default value will be returned - "light" in our case.

## Get All Preferences <a name="get-all"></a>

To get all the existing preferences use the `getAll()` method.

```javascript
let prefs = sdk.prefs.getAll();
```

Returns a key-value object, for example:

```javascript
{
    theme: 'dark',
    main_currency: 'USD'
}
```

## Set One Preference <a name="set-one"></a>

To set a preference, provide a key and value to the `set(key, value)` method.

```javascript
sdk.prefs.set('theme', 'monokai');
```

## Set Multiple Preferences <a name="set-bulk"></a>

To set multiple (not necessarily all) preferences at the same time, use the `setBulk(prefs)` method, where prefs is a key-value object.

```javascript
sdk.db.prefs.setBulk({
    theme: 'dark',
    main_currency: 'USD'
});
```