---
layout: default
type: guide
shortname: Docs
title: Behaviors
subtitle: Developer guide
---

{% include toc.html %}

{{site.project_title}} supports extending custom element prototypes with 
shared code modules called _behaviors_.

A behavior is similar to a typical mixin, but it can also define
[lifecycle callbacks](registering-elements.html#basic-callbacks),  [declared
properties](properties.html), [default attributes](registering-elements.html#host-attributes)[dart issue](https://github.com/dart-lang/polymer-dart/issues/561),
[`observers`](properties.html#observing-changes-to-multiple-properties), and [`listeners`](events.html#event-listeners).

To add a behavior to a {{site.project_title}} element definition, include it as
a mixin on your element class.

    @jsProxyReflectable
    @PolymerRegister('super-element')
    class SuperElement extends PolymerElement with SuperBehavior {
      SuperElement.created() : super.created();
    }

Lifecycle callbacks are called on the element class first, then for each
behavior in the order they are mixed in (left to right).

All the regular mixin rules apply to behaviors.

## Defining behaviors

To define a behavior, create a new abstract class, and annotate it with
`@behavior`. The following example defines the `HighlightBehavior`:


`highlight_behavior.dart`:

    @behavior
    abstract class HighlightBehavior {
      @Property(notify: true, observer: 'highlightChanged')
      bool isHighlighted = false;
      
      static created(instance) {
        print('Highlighting for $instance enabled!');
      }

      @Listen('click')
      toggleHighlight(_, __) {
        set('isHighlighted', !isHighlighted);
      },
      
      @eventHandler
      highlightChanged(bool newValue, _) {
        toggleClass('highlighted', newValue);
      }

    };

`my_element.dart`:

    import 'highlight_behavior.dart';

    @jsProxyReflectable
    @PolymerRegister('my-element')
    class MyElement extends PolymerElement with HighlightBehavior {
      MyElement.created() : super.created();
    }

## Extending behaviors {#extending}

To extend a behavior, or create a behavior that includes an existing behavior,
you can add the desired behaviors to the `implements` clause of your class. The
framework will enforce that those classes directly precede your class in the
mixin list of any polymer element.

    import 'oldbehavior.dart';
    
    @behavior
    abstract class NewBehavior implements OldBehavior {}

This would enforce the following ordering:

    @jsProxyReflectable
    @PolymerRegister('my-element')
    class MyElement extends PolymerElement with OldBehavior, NewBehavior {
      MyElement.created() : super.created();
    }
    
If you would instead like to enforce that another behavior is mixed in after
your behavior, you can create an `Impl` class, and add that to a dummy behavior
class which just has an implements clause:

    import 'oldbehavior.dart';
    
    @behavior
    abstract class NewBehaviorImpl {
      // Actual behavior implementation.
    }
    
    @behavior
    abstract class NewBehavior implements NewBehaviorImpl, OldBehavior {
      // Empty, this guy just groups up [NewBehaviorImpl] and [OldBehavior].
    }
    
This would enforce the following ordering:

    @jsProxyReflectable
    @PolymerRegister('my-element')
    class MyElement extends PolymerElement with
        NewBehaviorImpl, OldBehavior, NewBehavior {
      MyElement.created() : super.created();
    }

The `Impl` class must be public, but you can hide it in an import from your
`src` directory (which you do not export).

### Using behaviors written in JS {#interop}

You can also use a behavior written in JS. To do this you will need to create a
new class as before, but add a `@BehaviorProxy` annotation. The argument to
to the annotation is a const list representing the path from the js global
context to the js object for the behavior.

So, to create a dart class which references the js object at `My.Behavior`
you would do the following:

    @BehaviorProxy(const ['My', 'Behavior'])
    abstract class MyBehavior {}
    
If you want to also provide access to properties/methods created by this
behavior, then add `CustomElementProxy`, to the `implements` clause. This
will give you access to the `jsElement` property, which can access all fields
added by the behavior.

Lets say that `My.Behavior` adds a `myString` property, and a `printMyString`
method. The following would provide access to those:

     @BehaviorProxy(const ['My', 'Behavior'])
     abstract class MyBehavior implements CustomElementProxy {
       String get myString => jsElement['myString'];
       void set myString(String value) {
        jsElement['myString'] = value;
       }
       
       void printMyString() => jsElement.callMethod('printMyString');
     }
     
This behavior can now be mixed in just like any other, and can co-exist with
behaviors written in dart.
