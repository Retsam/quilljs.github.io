---
layout: docs
title: Keyboard Module
permalink: /docs/modules/keyboard/
---

The Keyboard module enables custom behavior for keyboard events in particular contexts. Quill uses this to bind formatting hotkeys and prevent undesirable browser side effects.


### Key Bindings

Keyboard handlers are bound to a particular key and key modifiers, such as `metaKey`, `ctrlKey`, `shiftKey` and `altKey`. In addition `shortKey` is a platform specific modifier equivalent to `metaKey` on a Mac and `ctrlKey` otherwise.

Handlers will called with `this` bound to the keyboard instance and be passed the current selection range.

```js
quill.keyboard.addBinding({ key: 'B', shortKey: true }, function(range, context) {
  this.quill.formatText(range, 'bold', true);
});
```

If a modifier key is not, it is assumed to mean that modifier is not active. You may also pass `null` to mean any value for the modifier.

```js
// Only b with no modifier will trigger
quill.keyboard.addBinding({ key: 'B' }, handler);

// Only shift+b will trigger
quill.keyboard.addBinding({ key: 'B', shiftKey: true }, handler);

// Either b or shift+b will trigger
quill.keyboard.addBinding({ key: 'B', shiftKey: null }, handler);

```

Multiple handlers may be bound to the same key, in which case the handlers are called synchronously, in the order they were bound. By default, a handler stops propagating to the next handler, unless it explicitly returns `true`.

```js
quill.keyboard.addBinding({ key: Keyboard.keys.ENTER }, function(range) {
  // I will normally prevent handlers of the enter key
  // Return true to let later handlers be called
  return true;
});
```


### Context

Contexts enable further specification for handlers to be called only in particular scenarios. Regardless if context is specified, a context object is provided as a second parameter for all handlers.

```js
// If the user hits backspace at the beginning of list or blockquote,
// remove the format instead delete any text
quill.keyboard.addBinding({ key: Keyboard.keys.BACKSPACE }, {
  collapsed: true,
  format: ['blockquote', 'list'],
  offset: 0
}, function(range, context) {
  if (context.format.list) {
    this.quill.format('list', false);
  } else {
    this.quill.format('blockquote', false);
  }
});
```

#### collapsed

If `true`, handler is called only if the user's selection is collapsed, i.e. in cursor form. If `false`, the users's selection must be non-zero length, such as when the user has highlighted text.


#### empty

If `true`, called only if user's selection is on an empty line, `false` for a non-empty line. Note setting empty to be true implies collapsed is also true and offset is 0&mdash;otherwise the user's selection would not be on an empty line.

```js
// If the user hits enter on an empty list, remove the list instead
quill.keyboard.addBinding({ key: Keyboard.keys.ENTER }, {
  empty: true,    // implies collapsed: true and offset: 0
  format: ['list']
}, function(range, context) {
  this.quill.format('list', false);
});
```


#### format

When an Array, handler will be called if *any* of the specified formats are active. When an Object, *all* specified formats conditions must be met. In either case, the context parameter will be an Object of all current active formats, the same returned by `quill.getFormat()`.

```js
var context = {
  format: {
    list: true,       // must be on a list, but can be any value
    script: 'super',  // must be exactly superscript, subscript will not suffice
    link: false       // cannot be in any link
  }
};
```


#### offset

Handler will be only called when the user's selection starts `offset` characters from the beginning of the line. Note this is before printable keys have been applied. This is useful in combination with other context specifications.


#### prefix

Regex that must match the text immediately preceding the user's selection's start position. The text will not match cross format boundaries. The supplied `context.prefix` value will be the entire immediately preceding text, not just the regex match.

```js
// When the user types space...
quill.keyboard.addBinding({ key: ' ' }, {
  collapsed: true,
  format: { list: false },  // ...on an line that's not already a list
  prefix: /^-$/,            // ...following a '-' character
  offset: 1,                // ...at the 1st position of the line,
                            // otherwise handler would trigger if the user typed
                            // hyphen space in the middle of a sentence
}, function(range, context) {
  // the space character is consumed by this handler
  // so we only need to delete the hyphen
  this.quill.deleteText(range.index - 1, 1);
  // apply bullet formatting to the line
  this.quill.formatLine(range.index, 1, 'list', 'bullet');
  // restore selection
  this.quill.setSelection(range.index - 1);

  // console.log(context.prefix) would print '-'
});
```


#### suffix

The same as [`prefix`](#prefix) except matching text immediately following the user's selection's end position.


### Peformance

Like DOM events, Quill key bindings are blocking calls on every match, so it is a bad idea to have a very expensive handler for a very common key binding. Apply the same performance best practices as you would when attaching to common blocking DOM events, like `scroll` or `mousemove`.