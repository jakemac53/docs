---
layout: default
type: guide
shortname: Docs
title: Registration and lifecycle
subtitle: Developer guide
---

{% include toc.html %}


## Register a custom element {#register-element}


To register a custom element, use the `PolymerRegister` annotation on a class,
and pass in the tag name for the new element. The class must extend either
`PolymerElement` or a native html element class. See
[type-extension](#type-extension) for more info on extending native elements.

By specification, the custom element's name **must contain a dash (-)**. 

You must also call `initPolymer` at some point in your program, which will
find all the annotations and do the actual registration.

Example:

    @jsProxyReflectable
    @PolymerRegister('my-element')
    class MyElement extends PolymerElement {
      MyElement.created() : super.created();
      
      void ready() {
        text = `My element!`;
      }
    }
    
    main() async {
      // Actually registers the elements.
      await initPolymer();

      // create an instance with createElement:
      var el1 = document.createElement('my-element');
    }

Extending other custom elements is not directly supported today, although it may
work in some cases. If you want to share behavior between elements it is
recommended that you encapsulate the shared logic in a [behavior](#behaviors),
which is a special type of mixin class.

### Define a custom constructor {#custom-constructor}

If you want users of your element to be able to do `new MyElement()`, you can
do that by simply making a factory constructor which calls
`document.createElement('my-element')`. You can also set up additional state
if desired.

Example:

    @jsProxyReflectable
    @PolymerRegister('my-element')
    class MyElement extends PolymerElement {
      MyElement.created() : super.created();
      
      factory MyElement() => document.createElement('my-element');
    }

    main() async {
      await initPolymer();

      MyElement el1 = new MyElement();
    }

### Extend native HTML elements {#type-extension}

Polymer currently only supports extending native HTML elements (for example,
`input`, or `button`, as opposed to extending other custom elements, which will
be supported in a future release). These native element extensions are called
_type extension custom elements_.

To extend a native HTML element, extend the native class instead of
`PolymerElement`, and then mix in all of `PolymerMixin`, `PolymerBase`, and
`JsProxy`. Then, call `polymerCreated()` inside the `created` constructor.
You must also supply the tag name in the `extendsTag` named parameter
of the `PolymerRegister` annotation, but this restriction may go away in the
future.

Example:

    @jsProxyReflectable
    @PolymerRegister('my-input', extendsTag: 'input')
    class MyInput extends InputElement with
        PolymerMixin, PolymerBase, JsProxy {
      MyInput.created() : super.created() {
        polymerCreated();
      }
      factory MyInput() => document.createElement('input', 'my-input');
      
      void ready() {
        style.border = '1px solid red';
      }
    }

    main() async {
      await initPolymer();

      MyInput el1 = new MyInput();
    }

To use a type-extension element in markup, use the _native_ tag and add an
`is` attribute that specifies the extension type name:

    <input is="my-input">

<!-- legacy anchor -->
<a id="basic-callbacks"></a>

## Lifecycle callbacks {#lifecycle-callbacks}

Polymer elements can contain all the standard Custom Element lifecycle
callbacks:

- `created`
- `attached`
- `detached`
- `attributeChanged`

Polymer adds an extra callback, `ready`, which is invoked when Polymer has
finished creating and initializing the element's local DOM. I

Example:

    @jsProxyReflectable
    @PolymerRegister('my-element')
    class MyElement extends PolymerElement {
      MyElement.created() : super.created() {
        print('${localName}#$id was created');
      }

      void attached() {
        print('${localName}#$id was attached');
      }

      void detached() {
        print('${localName}#$id was detached');
      }

      void attributeChanged(name, type) {
        print('${localName}#$id attribute $name was changed to '
            '${attributes[name]}');
      }
    }

### Ready callback and local DOM initialization {#ready-method} 

The `ready` callback is called when an element's local DOM is ready.

It is called after the element's template has been stamped and all elements
**inside the element's local DOM** have been configured (with values bound from
parents, deserialized attributes, or else default values) and had their `ready`
method called. 

Implement `ready` when it's necessary to manipulate an element's
local DOM when the element is constructed.

    void ready() {
      // access a local DOM element by ID using `$`
      $['header'].text = 'Hello!'
    }

**Note:** This example uses [Automatic node finding](local-dom.html#node-finding) to
access a local DOM element. 
{: .alert .alert-info }

Within a given tree, `ready` is generally called in _document order_, but you should not
rely on the ordering of initialization callbacks between sibling elements, or between 
a host element and its light DOM children.

### Initialization order {#initialization-order}

The element's basic initialization order is:

- PolymerElement `created` constructor (the `super.created()` call).  
- local DOM initialized 
- `ready` callback
- Your custom element `created` constructor.
- `attached` callback

Note that the **initialization order may vary** depending on whether or not the
browser includes native support for web components. In particular, there are no
guarantees with regard to initialization timing between **sibling elements** or
between **parents and light DOM children**. You should not rely on observed
timing to be identical across browsers, except as noted below.

For a given element:

*   The `ready` callback is always called before `created`.
*   The `created` callback is always called before `attached`.
*   The `ready` callback is called on any **local DOM children** before it's
    called on the host element.

This means that an element's **light DOM children** may be initialized **before or after** 
the parent element, and an element's **siblings may become `ready` in any order**.

For accessing sibling elements when an element initializes you can call `async` from inside
the `attached` callback:

    void attached() {
      async(() {
        // access sibling or parent elements here
      });
    }

**Dart note:**: In Polymer JS, the `ready` method will actually be called before
the `created` constructor, but in dart it is the other way around. This is
because it gets invokes as part of the `polymerCreated()` call inside of the
`PolymerElement.created()` constructor, which runs before your custom element
constructor.

## Static attributes on host {#host-attributes}

If a custom elements needs HTML attributes set on it at create-time, these may
be declared in a `hostAttributes` argument of the `PolymerRegister` annotation.
This takes a const map where keys are the attribute name and values are the
values to be assigned.  Values should typically be provided as strings, as HTML
attributes can only be strings; however, the standard `serialize` method is used
to convert values to strings, so `true` will serialize to an empty attribute,
and `false` will result in no attribute set, and so forth (see
[Attribute serialization](properties.html#attribute-serialization) for more
details).

Example:

    @jsProxyReflectable
    @PolymerRegister('x-custom', hostAttributes: const {
      'string-attribute': 'Value',
      'boolean-attribute': true,
      'tabindex': 0,
    });
    class XCustom extends PolymerElement {
      XCustom.created() : super.created()
    }

Results in:

    <x-custom string-attribute="Value" boolean-attribute tabindex="0"></x-custom>

**Note:** The `class` attribute can't be configured using `hostAttributes`.
{: .alert .alert-error }

## Behaviors {#behaviors}

Elements can share code in the form of _behaviors_, which can define 
properties, lifecycle callbacks, event listeners, and other features.

For more information, see [Behaviors](behaviors.html).
