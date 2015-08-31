---
layout: default
type: guide
shortname: Docs
title: Declared properties
subtitle: Developer guide
---

{% include toc.html %}


You can declare properties on your custom element by adding the `@property`
annotation to any public field of the class. This allows a user to configure the
property from markup (see 
[attribute deserialization](#attribute-deserialization) for details).
**Any property that's part of your element's public API should be annotated with
@property**

In addition, the `Property` annotation can be used to specify:

* Property change observer. Calls a method whenever the property value changes.
* Two-way data binding support. Fires an event whenever the property value changes.
* Computed property. Dynamically calculates a value based on other properties.
* Property reflection to attribute. Updates the corresponding attribute value when the property value changes.

Example:

    @jsProxyReflectable
    @PolymerRegister('x-custom')
    class XCustom extends PolymerElement {
      XCustom.created() : super.created():
      
      @property
      String user;
      
      @property
      bool isHappy;
      
      @Property(notify: true)
      num get count;
    }

The `Property` object supports the following named arguments for each property:

<table>
<tr>
<th>Key</th><th>Details</th>
</tr>
<tr>
<td><code>reflectToAttribute</code></td>
<td>Type: <code>bool</code><br> 

Set to `true` to cause the corresponding attribute to be set on the host node
when the property value changes. If the property type is `bool`, the attribute
is created as a standard HTML boolean attribute (set if true, not set if false).
For other property types, the attribute value is a string representation of the
property value. Equivalent to `reflect` in {{site.project_title}} 0.5.
See <a href="#attribute-reflection">Reflecting properties to attributes</a> for
more information.
</td>
</tr>
<tr>
<td><code>notify</code></td>
<td>Type: <code>bool</code><br> 

If `true`, the property is available for two-way data binding. In addition, an
event, <code><var>propertyName</var>-changed</code> is fired whenever the
property changes. See <a href="#notify">Property change notification events (notify)</a>
for more information.
</td>
</tr>
<tr>
<td><code>computed</code></td>
<td>Type: <code>String</code><br>

The value is interpreted as a method name and argument list. The method is invoked
to calculate the value whenever any of the argument values changes. Computed
properties should never be written to directly. See
<a href="#computed-properties">Computed properties</a> for more information.
</td>
</tr>
<tr>
<td><code>observer</code></td>
<td>Type: <code>String</code><br>

The value is interpreted as a method name to be invoked when the property value 
changes. Note that unlike in 0.5, <strong>property change handlers must be registered 
explicitly.</strong> The <code><var>propertyName</var>-changed</code> method will not be 
invoked automatically. See <a href="#change-callbacks">Property change callbacks (observers)</a> 
for more information.
</td>
</tr>
</table>

## Property name to attribute name mapping {#property-name-mapping}

For data binding, deserializing properties from attributes, and reflecting
properties back to attributes, {{site.project_title}} maps attribute names to property
names and the reverse. 

When mapping attribute names to property names:

*   Attribute names are converted to lowercase property names. For example,
    the attribute `firstName` maps to `firstname`.

*   Attribute names with _dashes_ are converted to _camelCase_ property names 
    by capitalizing the character following each dash, then removing the dashes. 
    For example, the attribute `first-name` maps to `firstName`.

The same mappings happen in reverse when converting property names to attribute
names (for example, if a property is defined using `reflectToAttribute: true`.)

**Compatibility note:** In 0.5, Polymer attempted to map attribute names to corresponding properties.
For example, the attribute `foobar` would map to the property `fooBar` if it was
defined on the element. This **does not happen in 0.8** &mdash; attribute to property
mappings are set up on the element at registration time based on the rules
described above.
{: .alert .alert-info }

## Attribute deserialization {#attribute-deserialization}

If a field is annotated with @property, an attribute on the instance matching
the property name will be deserialized according to the type specified and
assigned to a field of the same name on the element instance.

The type system includes support for Map and List values expressed as JSON,
or Date objects expressed as any Date-parsable string representation. Boolean
properties set based on the existence of the attribute: if the attribute exists
at all, its value is true, regardless of its string-value (and the value is only
false if the attribute does not exist).

Example:

`x_custom.dart`:

    @jsProxyReflectable
    @PolymerRegister('x-custom')
    class XCustom extends PolymerElement {
      XCustom.created() : super.created();
    
      @property
      String user;
    
      @Property(notify: true)
      bool manager = false;
    
      attached() {
        text = 'Hello World, my user is ${user == null ? 'nobody' : user}.\n'
            'This user is ${manager ? '' : 'not'} a manager.';
      }
    }
    
`index.html`:

    <x-custom user="Scott" manager></x-custom>
    <!--
    <x-custom>'s text content becomes:
    Hello World, my user is Scott.
    This user is a manager.
    -->

In order to configure camel-case properties of elements using attributes, dash-
case should be used in the attribute name.  Example:

`x_custom.dart`:

    @jsProxyReflectable
    @PolymerRegister('x-custom')
    class XCustom extends PolymerElement {
      XCustom.created() : super.created();
      
      @property
      String userName;
    }
    
`index.dart`:

    <x-custom user-name="Scott"></x-custom>
    <!-- Sets <x-custom>.userName = 'Scott';  -->


**Note:** Deserialization occurs both at create time, and at runtime (for
example, when the attribute is changed using `setAttribute`).  However, it is
encouraged that attributes only be used for configuring properties in static
markup, and instead that properties are set directly for changes at runtime. 
{: .alert .alert-info }


## Property change observers {#change-callbacks}

Custom element properties may be observed for changes by specifying `observer`
argument of the `Property` annotation for the field with the name of a function
to call.  When the property changes, the change handler will be called with the
new and old values as arguments.

Example:

    @jsProxyReflectable
    @PolymerRegister('x-custom')
    class XCustom extends PolymerElement {
      XCustom.created() : super.created();
    
      @Property(observer: 'disabledChanged')
      bool disabled;
    
      @Property(observer: 'highlightChanged')
      bool highlight;
    
      @eventHandler
      void disabledChanged(newValue, oldValue) {
        toggleClass('disabled', newValue);
        set('highlight', true);
      }
    
      @eventHandler
      void highlightChanged([_, __]) {
        classes.add('highlight');
        async(() {
          classes.remove('highlight');
        }, waitTime: 300);
      }
    }

    
**Dart note:** You must also annotate each of the methods with @eventHandler in
order to make them available via reflection.

**Compatibility note:** The argument order for change handlers is currently the
**opposite** of the order used in 0.5. 
{: .alert .alert-info }

**Dart note:** Property change observation is currently achieved in Polymer Dart
by using the `set` function to modify any properties on your class. This is
different from Polymer JS because we cannot dynamically create fields on your
class. We will investigate alternatives for Polymer Dart 1.1.0.

### Observing changes to multiple properties {#multi-property-observers}

To observe changes to a set of properties, use the `@Observe(...)` annotation.

These observers differ from single-property observers in a few ways:

*   Observers are not invoked until all dependent properties are defined (`!= null`).  
    So each dependent properties should have a default value assigned (or otherwise 
    be initialized to a non-`null` value) to ensure the observer is called.
*   Observers do not receive `old` values as arguments, only new values.  Only single-property 
    observers defined in the `Property` object receive both `old` and `new` values.

Example:

    @jsProxyReflectable
    @PolymerRegister('x-custom')
    class XCustom extends PolymerElement {
      XCustom.created() : super.created();
      
      @property
      bool preload;
      
      @property
      String src;
      
      @property
      String size;
      
      @Observe('preload, src, size')
      void updateImage(bool newPreload, String newSrc, String newSize) {
        // ... do work using dependent values
      }
    }

In addition to properties, observers can also observe [paths to sub-properties](#observing-path-changes),
[paths with wildcards](#deep-observation), or [array changes](#array-observation).

### Observing path changes {#observing-path-changes}

You can also observe changes to object sub-properties using the 
`Observe` annotation, by specifying a full path (`user.manager.name`)
as a function argument.

Example:

    @jsProxyReflectable
    @PolymerRegister('x-custom')
    class XCustom extends PolymerElement {
      XCustom.created() : super.created();
      
      @property
      User user;
      
      @Observe('user.manager')
      void userManagerChanged(User newUser) {
        print('new manager name is ' + newUser.name);
      }
    }

To observe a change to a path (object sub-property) the value **must be changed in
one of the following ways**:

*   Using a Polymer [property binding](data-binding.html#property-binding) to another element.
*   Using the [`set`](data-binding.html#set-path) API, which provides the
    required notification to elements with registered interest.
    
**Dart note:** The above rules also apply to top level properties in dart, as
noted in the Property Observers section.

### Deep path observation {#deep-observation}

To call an observer when any (deep) sub-property of an
object changes, specify a path with a wildcard (`*`).

When you specify a path with a wildcard, the argument passed to your
observer is a change record object with the following properties:

*   `path`. Path to the property that changed. 
*   `value`. New value of the path that changed.
*   `base`. The object matching the non-wildcard portion of the path. 

Example:

    @jsProxyReflectable
    @PolymerRegister('x-custom')
    class XCustom extends PolymerElement {
      XCustom.created() : super.created();
    
      @property
      User user;
    
      @Observe('user.manager.*')
      void userManagerChanged(Map changeRecord) {
        if (changeRecord['path'] == 'user.manager') {
          // user.manager object itself changed
          print('new manager name is ' + changeRecord['value'].name);
        } else {
          // sub-property of user.manager changed
          print('${changeRecord['path']} changed to ${changeRecord['value']}');
        }
      }
    }

### List observation {#array-observation}

Finally, to observe mutations to lists (changes resulting from calls to `add`,
`addAll`, `clear`, `fillRange`, `insert`, `insertAll`, `removeItem`, `removeAt`,
`removeLast`, `removeRange`, `removeWhere`, `replaceRange`, `retainWhere`,
`setAll`, and `setRange`), specify a path to an array followed by `.splices` as
an argument to the observer function.  

The value received by the observer for the `splices` path of a list is a
change records with the following properties:

*   `indexSplices`. Lists the set of changes that occurred to the list, in 
     terms of list indicies. Each `indexSplices` record contains the following 
     properties:

     -   `index`. Position where the splice started.
     -   `removed`. List of `removed` items.
     -   `addedCount`. Number of new items inserted at `index`. 

*   `keySplices`. Lists the set of changes that occurred to the array in terms
    of "keys" used by Polymer for identifying list elements. Each `keySplices` 
    record contains the following properties: 

    -   `added`. Array of added keys.
    -   `removed`. Array of removed keys. 

Example:

    @jsProxyReflectable
    @PolymerRegister('x-custom')
    class XCustom extends PolymerElement {
      XCustom.created() : super.created();
    
      @property
      List<User> users = [];
      @Observe('users.splices')
      void usersAddedOrRemoved(Map changeRecord) {
        if (changeRecord == null) return;
        changeRecord['indexSplices'].forEach((s) {
          s['removed'].forEach((user) {
            print('${user.name} was removed');
          });
          print('${s['addedCount']} users were added');
        });
      }
    
      void addUser() {
        add('users', new User("Jack Aubrey"));
      }
    }

**List mutation methods.** Observing changes to arrays is dependent on the change to the list
being made through one of the [list mutation methods](#list-mutation) provided
on Polymer elements, which provides the required notification to elements with
registered interest.
{: .alert .alert-info }

When you specify a wildcard path on an array, the observer is for both splices as
well as array element sub-property changes.  So the  observer in the
following example will be called for all additions, removals, and deep changes
that occur in the array:

    @jsProxyReflectable
    @PolymerRegister('x-custom')
    class XCustom extends PolymerElement {
      XCustom.created() : super.created();
      
      @property
      List<User> users = [];
      
      @Observe('users.*')
      void usersChanged(Map changeRecord) {
        if (changeRecord['path'] == 'users.splices') {
          // a user was added or removed
        } else {
          // an individual user or its sub-properties changed
          // check "changeRecord.path" to determine what changed
        }
      }
    }

### List mutation methods {#list-mutation}

When modifying lists, a set of list mutation methods are provided by the
`PolymerMixin` class which mimic the `List` api, with the exception that
they take a `path` string as the first argument and a few other minor
differences.  The `path` argument identifies a list on the element to mutate,
with the following arguments matching those of the native `List` methods.

These methods perform the mutation action on the list, and then notify other
elements that may be bound to the same list of the changes.  You must use these
methods when mutating a list to ensure that any elements watching the list
(via observers, computed properties, or data bindings) are kept in sync.

Every Polymer element has the following list mutation methods available:

* `add(String path, item)`
* `addAll(String path, Iterable items)`
* `clear(String path)`
* `fillRange(String path, int start, int end, [fillValue])`
* `insert(String path, int index, element)`
* `insertAll(String path, int index, Iterable elements)`
* `removeItem(String path, value)`
  * `remove` was renamed to `removeItem` because HtmlElement already has a
    method by that name.
* `removeAt(String path, int index)`
* `removeLast(String path)`
* `removeRange(String path, int start, int end)`
* `removeWhere(String path, bool test(element))`
* `replaceRange(String path, int start, int end, Iterable replacement)`
* `retainWhere(String path, bool test(element))`
* `setAll(String path, int index, Iterable iterable)`
* `setRange(String path, int start, int end, Iterable iterable, [int skipCount = 0])`

Example:

`custom_element.html`:

    <dom-module id="custom-element">
      <template>
        <template is="dom-repeat">{{users}}</template>
      </template>
    </dom-module>

`custom_element.dart`:

    @jsProxyReflectable
    @PolymerRegister('custom-element')
    class CustomElement extends PolymerElement {
      CustomElement.created() : super.created();
      
      void addUser(User user) {
        add('users', user);
      }
      
      void removeUser(User user) {
        removeItem('users', user);
      }
    }

## Property change notification events (notify) {#notify}

When a property is set to `notify: true`, an event,
<code><var>propertyName</var>-changed</code>, is fired whenever the property
value changes. These events are used by the two-way data binding system, and can
also notify external scripts and frameworks to respond to changes in the element.

For more on property change notifications and data binding, see  [Property
change notification and two-way binding](data-binding.html#property-notification).


## Read-only properties {#read-only}

When a property only "produces" data and never consumes data, this can be made
explicit to avoid accidental changes from the host by defining only a getter
for the field.  In order for the element to actually change the value of the
property, it must use a private variable which holds the actual value,
following normal dart semantics. Then, to notify the system that the value has
changed, you must call `notifyPath('propertyName', value);`.

    @jsProxyReflectable
    @PolymerRegister('my-element')
    class MyElement extends PolymerElement {
      MyElement.created() : super.created();
      
      @property
      String get myValue => _myValue;
      String _myValue;
      
      @eventHandler
      someEventHandler() {
        _myValue = 'hello!';
        notifyPath('myValue', _myValue);
      }
    }
    
**Dart Note:** In Polymer Js they create magic `_set{{PropertyName}}` functions,
which set the value and call notifyPath. In dart we can't create these on the
fly, so you have to manually call `notifyPath`.

For more on read-only properties and data binding, see 
[Property change notification and two-way binding](data-binding.html#property-notification).

## Computed properties {#computed-properties}

Polymer supports virtual properties whose values are calculated from other
properties.

To define a computed property, pass `computed: 'someFunction(...)` to the
`@Property` annotation:

    @Property(computed: 'computeFullName(first, last)');
    String fullName;

The function is provided as a string with dependent properties as arguments 
in parenthesis. The function will be called once for any change to 
the dependent properties.

The computing function is not invoked until **all** dependent properties 
are defined (`!= null`). So each dependent properties should have a 
default value (or otherwise be initialized to a non-`null` value) to ensure the
property is computed.

**Note:** The definition of a computing function looks like the 
definition of a [multi-property observer](#multi-property-observers),
and the two act almost identically. The only difference is that the 
computed property function returns a value that's exposed as a virtual property.
{: .alert .alert-info }

`x_custom.html`:

    <dom-module id="x-custom">

      <template>
        My name is <span>{%raw%}{{fullName}}{%endraw%}</span>
      </template>

    </dom-module>

`x_custom.dart`:

    @jsProxyReflectable
    @PolymerRegister('x-custom')
    class XCustom extends PolymerElement {
      XCustom.created() : super.created();
      
      @property
      String first;
      
      @property
      String last;
      
      @Property(computed: 'computeFullName(first, last)')
      String fullName;
      
      @eventHandler
      String computeFullName(String first, String last) {
        return '$first $last';
      }
    }
  

Arguments to computing functions may be simple properties on the element, as 
well as any of the arguments types supported by `observers`, including [paths](#observing-path-changes), 
[paths with wildcards](#deep-observation), and [paths to array splices](#array-observation).  
The arguments received by the computing function match those described in the sections referenced above.

**Note:** If you only need a computed property for a data binding, you
can use a computed binding instead. See 
[Computed bindings](data-binding.html#annotated-computed).
{: .alert .alert-info }

## Reflecting properties to attributes {#attribute-reflection}

In specific cases, it may be useful to keep an HTML attribute value in sync with
a property value.  This may be achieved by setting `reflectToAttribute: true` on
a property in the `Property` annotation object.  This will cause any change
to the property to be serialized out to an attribute of the same name.

    @jsProxyReflectable
    @PolymerRegister('x-custom')
    class XCustom extends PolymerElement {
      XCustom.created() : super.created();
     
      @Property(reflectToAttribute: true)
      String myString;
      
      ready() {
        set('myString', 'ready');
        // results in attributes['my-string'] = 'ready';
      }
    }

### Attribute serialization {#attribute-serialization}

When reflecting a property to an attribute or 
[binding a property to an attribute](data-binding.html#attribute-binding),
the property value is _serialized_ to the attribute.

By default, values are serialized according to value's  _current_ type.

*   `String`. No serialization required.
*   `DateTime` or `num`. Serialized using  `toString`.  
*   `bool`. Results in a non-valued attribute to be either set (`true`) or removed (`false`).
*   `List` or `Map`. Serialized using `JSON.stringify`. 

To supply custom serialization for a custom element, override your element's `serialize` method.
