---
layout: default
type: guide
shortname: Docs
title: Utility functions
subtitle: Developer guide
---

{% include toc.html %}


## Utility Functions {#utility-functions}

All {{site.project_title}} elements inherit from `{{site.project_title}}.Base`, which 
provides a set of useful convenience functions for instances to use.

*   `$$(String selector)`. Returns the first node in this element's local DOM 
    that matches `selector`.

*   `toggleClass(String name, [bool value, Element node])`. Toggles the named 
    boolean class on the host element, adding the class if `value` is true and
    removing it if `value` is false or null. If `node` is specified, sets the 
    class on `node` instead of the host element.

*   `toggleAttribute(String name, [bool value, Element node])`. Like
    `toggleClass`, but toggles the named boolean attribute.

*   `attributeFollows(String name, Element newNode, Element oldNode)`. Moves a
    boolean attribute from `oldNode` to `newNode`, unsetting the attribute (if
    set) on `oldNode` and setting it on `newNode`.

*   `classFollows(String name, Element newNode, Element oldNode)`. Moves a class
    from `oldNode` to `newNode`, removing the class (if present) on `oldNode`
    and adding it to `newNode`.

*   `fire(String type, {detail, bool canBubble, bool cancelable, Node node})`.
    Fires a custom event with the desired properties.

*   `int async(void callback(), {int waitTime})`. Calls `method` asynchronously.
    If no wait time is specified, runs tasks with microtask timing (after the 
    current method finishes, but before the next event from the event queue is 
    processed). Returns a handle that can be used to cancel the task.

*   `cancelAsync(handle)`. Cancels the identified async task.

*   `debounce(String jobName, void callback(), {int waitTime})`. Call `debounce`
    to collapse multiple requests for a named task into one invocation, which is
    made after the wait time has elapsed with no new request.  If no wait time
    is given, the callback is called at microtask timing (guaranteed to be
    before paint).

*   `cancelDebouncer(String jobName)`. Cancels an active debouncer without
    calling the callback.

*   `flushDebouncer(String jobName)`. Calls the debounced callback immediately
    and cancels the debouncer.

*   `bool isDebouncerActive(String jobName)`. Returns true if the named debounce
    task is waiting to run.

*   `transform(String transform, [Element node])`. Applies a CSS transform to 
    the specified node, or host element if no node is specified. `transform` is
    specified as a string. For example:

         transform('rotateX(90deg)', this.$['myDiv']);

*   `translate3d(String x, String y, String z, [Element node])`. Transforms the
    specified node, or host element if no node is specified. For example:

        this.translate3d('100px', '100px', '100px');

*   `importHref(String href, {void onLoad(e), void onError(e)})`. Dynamically
    imports an HTML document.

        this.importHref('path/to/page.html', (e) {
            // e.target.import is the import document.
        }, (e) {
            // loading error
        });

    **Note:** To call `importHref` from outside a Polymer element, use `Polymer.Base.importHref`.
    {: .alert .alert-info }
    
    **Dart Note:** Html files imported this way cannot contain dart script tags.
