---
layout: docs
title: Modules
permalink: /docs/modules/
---

Modules allow Quill's behavior and functionality to customized. Several officially supported modules are available to pick and choose from, some with additional configuration options and APIs. Refer to their respective documentation pages for more details.

To enable a module, simply include it in Quill's configuration.

```javascript
var quill = new Quill('#editor', {
  modules: {
    'history': {          // Enable with custom configurations
      'delay': 2500,
      'userOnly': true
    },
    'syntax': true        // Enable with default configuration
  }
});
```

The [Clipboard](/docs/modules/clipboard/), [Keyboard](/docs/modules/keyboard/), and [History](/docs/modules/history/) modules are required by Quill and do not need to be included explictly, but may be configured like any other module.


## Extending

Modules may also be extended and re-registered, replacing the original module. Even required modules may be re-registered and replaced.

```javascript
var Clipboard = Quill.import('modules/clipboard');
var Delta = Quill.import('delta');

class PlainClipboard extends Clipboard {
  convert(html = null) {
    if (typeof html === 'string') {
      this.container.innerHTML = html;
    }
    return new Delta().insert(this.container.innerText);
  }
}

Quill.register('modules/clipboard', PlainClipboard, true);

var quill = new Quill('#editor');   // Will be created with instance of PlainClipboard
```

*Note: This particular example was selected to show what is possible. It is usually better to just use Clipboard's [addMatcher](/docs/modules/clipboard/#addmatcher) for most paste customization scenarios.*