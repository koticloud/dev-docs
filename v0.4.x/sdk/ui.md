# Database

<ul class="nav">
    <li><a href="#introduction">Introduction</a></li>
    <li><a href="#dialogs">Dialogs</a></li>
    <li><a href="#notifications">Notifications</a></li>
    <li><a href="#misc-methods">Misc Methods</a></li>
</ul>

## Introduction <a name="introduction"></a>

The `UI` module provides you with a few useful methods for things like dialogs and notifications. All the components have default Koti Cloud styling. You shouldn't use the 'UI' class directly. Instead, use the instance from the global `App` instance: `sdk.ui`.

## Dialogs <a name="dialogs"></a>

There are pre-defined types of dialogs for you to use. You can also use a generic `dialog()` method to show a custom dialog (modal).

### Alert Dialog

A typical alert dialog with a single "OK" button. Providing an ID as the second parameter ensures the same dialog cannot be opened twice at the same time.

```javascript
sdk.ui.alert('You have been alerted!', 'unique-dialog-id')
    .then(() => {
        // User pressed "OK"
    });
```

### Confirmation Dialog

A confirmation dialog with two buttons: "Yes" and "No". Providing an ID as the second parameter ensures the same dialog cannot be opened twice at the same time.

```javascript
sdk.ui.confirm('Are you sure you want to delete all your data?', 'unique-dialog-id')
    .then(() => {
        // User pressed "Yes"
    }).catch(() => {
        // User pressed "No"
    });
```

### Select Dialog

Select dialog is a mobile-friendly dialog that let's the user choose one of the provided options. It can be used instead of a standard form select element too.

```javascript
const options = {
    id: 'unique-dialog-id',
    title: 'Transaction Type',
    selected: 'expense',    // Default (pre-selected) value
    options: {
        expense: 'Expense',
        income: 'Income',
    },
};

sdk.ui.select(options)
    .then((selectedKey) => {
        // Perform appropriate action
    });
```

### Custom Dialog

You can create custom dialogs too. Please examine the example below.

```javascript
// An optional list of buttons
const buttons = {
    saveBtn: {
        text: 'Save',
        // Custom button handler
        handler: () => {
            saveSettings();
        },
        closeDialog: false, // Don't close the dialog on this button press
    },
    cancelBtn: {
        text: 'Cancel',
        // reject() the Promise on press (otherwise resolve()'d by default)
        reject: true,
    },
    closeBtn: {
        text: 'Close',
        // reject() the Promise on press (otherwise resolve()'d by default)
        reject: true,
    },
};

const options = {
    id: 'settings-dialog',    // Optional but recommended
    title: 'Settings',  // Optional. You can have a custom title inside the body
    // Optional, but what's the point if the dialog has no body?
    body: 'Provide plain text, HTML, or a DOM object.',
    // Optional. You can have your custom buttons inside the body or no buttons
    // at all
    buttons: buttons,
};

sdk.ui.dialog(options)
    .then(() => {
        // Save pressed. In this case though, the "Save" button has a custom
        // handler, so you don't have to do anything here. It all depends on
        // whatever you want.
    })
    .catch(() => {
        // Cancel or Close pressed
        resetSettings();
    });
```

## Notifications <a name="notifications"></a>

You can show your users a notification that will appear in the bottom-right corner of the screen. The notifications stack one above another and automatically disappear after a few seconds.

```javascript
// 'info' it the default notification type, so you don't have to specify it
sdk.ui.notify('Just letting you know!', 'info');
sdk.ui.notify('Everything went fine!', 'success');
sdk.ui.notify('Something went wrong!', 'error');
```

## Misc Methods <a name="misc-methods"></a>

### `hasOpenDialogs()`

Returns `true` if there are any open dialogs.

### `closeDialog(id)`

Close a single dialog by its ID.

### `closeAllDialogs(id)`

Close all open dialogs.