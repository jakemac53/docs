---
layout: default
type: guide
shortname: Docs
title: Events
subtitle: Developer guide
---

{% include toc.html %}

## Event listener setup {#event-listeners}

Add event listeners to the host element by providing a `@Listen('event-name`)`
annotation to any method.

You can also add an event listener to any element in the `$` collection 
using the syntax <code><var>nodeId</var>.<var>eventName</var>.

Example:

`x_custom.html`:

    <dom-module id="x-custom">

      <template>
        <div>I will respond</div>
        <div>to a tap on</div>
        <div>any of my children!</div>
        
        <div id="special">I am special!</div>
      </template>

    </dom-module>

`x_custom.dart`:

    @jsProxyReflectable
    @PolymerRegister('x-custom')
    class XCustom extends PolymerElement {
      XCustom.created() : super.created();
      
      @Listen('tap')
      void regularTap(event, target) {
        window.alert('Thank you for tapping');
      }
      
      @Listen('special.tap')
      void specialTap(event, target) {
        window.alert('It was special tapping');
      }
    }



## Annotated event listener setup {#annotated-listeners}

To add event listeners to local-DOM children, use
<code>on-<var>event</var></code>  annotations in your template. This often
eliminates the need to give an element an `id` solely for  the purpose of
binding an event listener. For now you must also annotate the method with
`@eventHandler` for technical reasons.

Example:

`x_custom.html`:

    <dom-module id="x-custom">

      <template>
        <button on-click="handleClick">Kick Me</button>
      </template>

    </dom-module>
    
`x_custom.dart`:
   
    @jsProxyReflectable
    @PolymerRegister('x-custom')
    class XCustom extends PolymerElement {
      XCustom.created() : super.created();
      
      @eventHandler
      handleClick(event, target) {
        window.alert('Ow!');
      }
    }

Because the event name is specified using an HTML attribute, **the event name is always
converted to lowercase**. This is because HTML attribute names are case 
insensitive. So specifying `on-myEvent` adds a listener for `myevent`. The event handler 
_name_ (for example, `handleClick`) **is** case sensitive.

**Compatibility note:** The syntax differs from 0.5, which required curly brackets ({%raw%}{{}}{%endraw%})
around the event handler name.

**Lowercase event names.** When you use a declarative handler, the event name 
is converted to lowercase, because attributes are case-insensitive.
So the attribute `on-core-signal-newData` sets up a listener for `core-signal-newdata`, 
_not_ `core-signal-newData`. To avoid confusion, always use lowercase event names.
{: .alert .alert-info } 


## Gesture events {#gestures}

Polymer fires a custom "gesture" events for certain user
interactions automatically when a declarative listener is added for the event
type.  These events fire consistently on both touch and mouse environments,
so we recommend using these events instead of their mouse- or
touch-specific event counterparts. This provides better interoperability with both touch and
mouse devices.  For example, `tap` should be used instead of
`click` for the most reliable cross-platform results.

Listening for certain gestures controls the scrolling direction for touch input.
For example, nodes with a listener for the `track` event will prevent scrolling
by default. Elements can override scroll direction with
`this.setScrollDirection(direction, node)`, where `direction` is one of `'x'`,
`'y'`, `'none'`, or `'all'`, and `node` defaults to `this`.

The following are the gesture event types supported, with a short description
and list of detail properties available on `event.detail` for each type:

* **down** - finger/button went down
  * `x` - clientX coordinate for event
  * `y` - clientY coordinate for event
  * `sourceEvent` - the original DOM event that caused the `down` action
* **up** - finger/button went up
  * `x` - clientX coordinate for event
  * `y` - clientY coordinate for event
  * `sourceEvent` - the original DOM event that caused the `up` action
* **tap** - down & up occurred
  * `x` - clientX coordinate for event
  * `y` - clientY coordinate for event
  * `sourceEvent` - the original DOM event that caused the `tap` action
* **track** - moving while finger/button is down
  * `state` - a string indicating the tracking state:
      * `start` - fired when tracking is first detected (finger/button down and moved past a pre-set distance threshold)
      * `track` - fired while tracking
      * `end` - fired when tracking ends
  * `x` - clientX coordinate for event
  * `y` - clientY coordinate for event
  * `dx` - change in pixels horizontally since the first track event
  * `dy` - change in pixels vertically since the first track event
  * `ddx` - change in pixels horizontally since last track event
  * `ddy` - change in pixels vertically since last track event
  * `hover()` - a function that may be called to determine the element currently being hovered

Example:

`drag_me.html`:

    <dom-module id="drag-me">

      <style>
        #dragme {
          width: 500px;
          height: 500px;
          background: gray;
        }
      </style>

      <template>
        <div id="dragme" on-track="handleTrack">{{message}}</div>
      </template>

    </dom-module>

`drag_me.dart`:

    @jsProxyReflectable
    @PolymerRegister('drag-me')
    class DragMe extends PolymerElement {
      DragMe.created() : super.created();
      
      @eventHandler
      void handleTrack(e, _) {
        switch(e.detail['state']) {
          case 'start':
            message = 'Tracking started!';
            break;
          case 'track':
            message = 'Tracking in progress... ${e.detail.x}, ${e.detail.y}';
            break;
          case 'end':
            message = 'Tracking ended!';
            break;
        }
        notifyPath('message', message);
      }
    }

Example with `@Listen`:

`drag_me.html`:

    <dom-module id="drag-me">

      <style>
        #dragme {
          width: 500px;
          height: 500px;
          background: gray;
        }
      </style>

      <template>
        <div id="dragme">{{message}}</div>
      </template>

    </dom-module>

`drag_me.dart`:

    @jsProxyReflectable
    @PolymerRegister('drag-me')
    class DragMe extends PolymerElement {
      DragMe.created() : super.created();
      
      @Listen('dragme.track')
      void handleTrack(e, _) {
        switch(e.detail['state']) {
          case 'start':
            message = 'Tracking started!';
            break;
          case 'track':
            message = 'Tracking in progress... ${e.detail.x}, ${e.detail.y}';
            break;
          case 'end':
            message = 'Tracking ended!';
            break;
        }
        notifyPath('message', message);
      }
    }


## Event retargeting {#retargeting}

Shadow DOM has a feature called "event retargeting" which changes an event's
target as it bubbles up, such that target is always in the receiving element's
light DOM. Shady DOM does not do event retargeting, so events may behave differently
depending on which local DOM system is in use.

Use `Polymer.dom(event)` to get a normalized event object that provides
equivalent target data on both shady DOM and shadow DOM. Specifically, the
normalized event has the following properties:

*   `rootTarget`: The original or root target before shadow retargeting
    (equivalent to `event.path[0]` under shadow DOM or `event.target` under
    shady DOM).

*   `localTarget`: Retargeted event target (equivalent to `event.target` under
    shadow DOM)

*   `path`: Array of nodes through which event will pass 
    (equivalent to `event.path` under shadow DOM).


