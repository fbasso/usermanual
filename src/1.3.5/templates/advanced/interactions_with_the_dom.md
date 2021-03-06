Title: Interactions with the DOM


Aria Templates relies heavily on data binding to refresh parts of the interface that need to be updated, which is why you should never feel the need to directly modify the DOM when writing an AT application.
There are however very specific cases where manual update of the UI make sense from the code and this article describes them.
It explains how to deal with DOM element wrappers and update CSS classes, how to handle focus and scrolling in your application and how to use loading indicators.

## DOM elements wrappers

While writing your template, you can access and manipulate the DOM elements that are generated by the template markup.
This can be done in the template script (but also in the template itself) by calling the method [$getElementById](http://ariatemplates.com/api/#aria.templates.ITemplate:$getElementById:method).

The returned object is not the DOM element itself.
Indeed, direct access to the DOM element is strongly discouraged by Aria Templates.
Every DOM element available in templates is wrapped within an instance of the [DomElementWrapper](http://ariatemplates.com/api/#aria.templates.DomElementWrapper) class.
This allows

* to limit the interaction to a certain set of safe operations,
* to provide cross-browser implementations of commonly used methods,
* enhance the set of actions that can be performed on dom elements.

The argument to provide to the `$getElementById` method is an id that you have assigned

* to an element by using the [id statement](writing_templates#id).
* To a section in the [section statement](writing_templates#section). In this case the returned object is of class [SectionWrapper](http://ariatemplates.com/api/#aria.templates.SectionWrapper), which extends class `DomElementWrapper` by adding section-specific methods.
* To a [widget](widgets). In this case the returned DomElementWrapper is associated to the dom element that contains all the markup generated by the widget.

Consider the following code extracted from a HTML template.

<script src='%SNIPPETS_SERVER_URL%/snippets/github.com/ariatemplates/documentation-code/snippets/templates/domInteractions/DomInteractionTemplate.tpl?tag=domInteractions&lang=at&outdent=true'></script>

Its associated script could contain the following statements:

<script src='%SNIPPETS_SERVER_URL%/snippets/github.com/ariatemplates/documentation-code/snippets/templates/domInteractions/DomInteractionTemplateScript.js?tag=domInteractions&lang=at&outdent=true'></script>


## Focus handling

It is possible to programmatically set focus on a specific widget or generic dom element from the template script.
Note that it is only possible to do this from the template script of the same template in which the widget/element is defined.
The template script is endowed with the [`$focus`](http://ariatemplates.com/api/#aria.templates.ITemplate:$focus:method) that receives an id as parameter and sets the focus on the widget/element with that id.

As far as widgets are concerned, each of them can implement a `focus` method that will be called whenever you try to set the focus on it.
Suppose you are using a `TextField` widget from the `AriaLib` widget library in your HTML template:

<script src='%SNIPPETS_SERVER_URL%/snippets/github.com/ariatemplates/documentation-code/snippets/templates/domInteractions/DomInteractionTemplate.tpl?tag=focusExample&lang=at&outdent=true'></script>

In the associated template script you can use

<script src='%SNIPPETS_SERVER_URL%/snippets/github.com/ariatemplates/documentation-code/snippets/templates/domInteractions/DomInteractionTemplateScript.js?tag=focusExample&lang=at&outdent=true'></script>

which will set the input focus on the text field.
A side effect of this is that the browser will scroll to make the focused widget visible to the user, so this feature can also be used to programatically scroll to different parts of the template.


### Focus on sub-templates

Sub-templates are a very useful widget (`Template` widget of the standard `AriaLib` widget library).
When you include a template inside another one, you can provide an id and focus on it, as you would do with any other widget.

Suppose you have the following template:

<script src='%SNIPPETS_SERVER_URL%/snippets/github.com/ariatemplates/documentation-code/snippets/templates/domInteractions/TemplateA.tpl?lang=at&outdent=true'></script>

`TemplateA` includes two other templates (`TemplateB` and `TemplateC`) as sub-templates.

Suppose that, as soon as the template is ready to be displayed, you want to give focus to the sub-template C.
You can implement this behaviour in the script associated to template A.

<script src='%SNIPPETS_SERVER_URL%/snippets/github.com/ariatemplates/documentation-code/snippets/templates/domInteractions/TemplateAScript.js?lang=at&tag=first'></script>

As soon as template A and its sub-templates are ready to be displayed, the `$focus` method is called (for more information about the `$displayReady` go to [this article](template_scripts#intercepting-template-lifecycle-phases)).

Suppose that template C is defined in the following way:

<script src='%SNIPPETS_SERVER_URL%/snippets/github.com/ariatemplates/documentation-code/snippets/templates/domInteractions/TemplateC.tpl?lang=at&outdent=true'></script>

Basically it has two text fields with id's `textFieldA` and `textFieldB`.
What happens when it receives the focus from template A?

You can specify the actions to take after a sub-template is given focus by defining the method **`$focusFromParent`** inside the sub-template script.
For example, suppose that you want to give focus to the textfield with id `textFieldB`.
You can do that in the template script

<script src='%SNIPPETS_SERVER_URL%/snippets/github.com/ariatemplates/documentation-code/snippets/templates/domInteractions/TemplateCScript.js?lang=at&outdent=true'></script>

If the method `$focusFromParent` is not implemented on Template C, then the focus received from the parent template would be cast onto the first focusable element (`TextField` with id `"textFieldA"` in the above example).

Sometimes it is desirable to set the focus on an element inside the template after the latter is refreshed.
In these cases it is possible to define the method `$afterRefresh` in the template script.
Whenever the `$refresh` method is called on the template, the `$afterRefresh` method is invoked (when it has been defined).
For example, in the `$prototype` property of the template A script definition, it is possible to add the method

<script src='%SNIPPETS_SERVER_URL%/snippets/github.com/ariatemplates/documentation-code/snippets/templates/domInteractions/TemplateAScript.js?tag=afterRefresh&lang=at&outdent=true'></script>

so that the focus is given to the sub-template C whenever template A is refreshed.
For more information about the `$afterRefresh` method go to [this article](template_scripts#intercepting-template-lifecycle-phases)).

#### Example

<iframe class='samples' src='%SNIPPETS_SERVER_URL%/samples/github.com/ariatemplates/documentation-code/samples/templates/domInteractions/focushandling/' ></iframe>

In this sample you can see how sub-template A gives focus to the second checkbox after receiving it from the containing template (its script implements the `$focusFromParent` method).
Sub-template B, instead, does not have any script at all.
This is why the focus is automatically passed over to the first focusable element.


## Scrolling

### Templates

It is possible to get/set the scroll positions of the DOM element that contains the template by using the `getContainerScroll` and `setContainerScroll` methods from a template script.
This element can be either:

* the `span` wrapping a template widget
* the `div` container specified in the `Aria.loadTemplate()` method


### DomElementWrapper

The `DomElementWrapper` class provides methods to control the scroll positions of elements inside a template that can be retrieved using `$getElementId()` inside the template script. These methods are:

* `getScroll` which returns an object containing the scrollLeft and scrollTop values
* `setScroll` which allows to set them
* `scrollIntoView` whose purpose is to scroll the element until a certain element contained inside it becomes visible

For more information, check the [`DomElementWrapper`](http://ariatemplates.com/api/#aria.templates.DomElementWrapper) class.


## Processing indicators

Aria Templates allows you to display processing indicators overlays on top of DOM elements.
In particular, you can trigger the processing indicator overlay on

* DOM element wrappers
* Sections
* Generic DOM elements


### On DOM element wrappers

Once you retrieve a DOM element wrapper through the `$getElementById` method, you can use the method [setProcessingIndicator](http://ariatemplates.com/api/#aria.templates.DomElementWrapper:setProcessingIndicator:method) method.

Suppose you create a span in your template:

<script src='%SNIPPETS_SERVER_URL%/snippets/github.com/ariatemplates/documentation-code/snippets/templates/domInteractions/ProcIndTemplate.tpl?tag=procIndOne&lang=at&outdent=true'></script>

You can then retrieve its associated DOM element wrapper and trigger the display of a processing indicator on top of it:

<script src='%SNIPPETS_SERVER_URL%/snippets/github.com/ariatemplates/documentation-code/snippets/templates/domInteractions/ProcIndTemplateScript.js?tag=procIndOne&lang=at&outdent=true'></script>

You can see that it is also possible to provide a message to be displayed on the overlay that contains the processing indicator image.

**Remark:** the processing indicator is removed when a template refresh occurs.


### On Sections

The processing indicator above a section can be displayed either through a Section Wrapper or by automatically binding it to the data-model.

#### Section Wrapper

When you give the id of a section as a parameter of the `$getElementById` method, it returns the instance of [SectionWrapper](http://ariatemplates.com/api/#aria.templates.SectionWrapper) associated to the section with the specified id.
Since the `SectionWrapper` class extends the `DomElementWrapper` class, you can trigger a processing indicator on top of the section dom just like you would on any DOM element wrapper.

In the template:

<script src='%SNIPPETS_SERVER_URL%/snippets/github.com/ariatemplates/documentation-code/snippets/templates/domInteractions/ProcIndTemplate.tpl?tag=procIndTwo&lang=at&outdent=true'></script>

In the template script:

<script src='%SNIPPETS_SERVER_URL%/snippets/github.com/ariatemplates/documentation-code/snippets/templates/domInteractions/ProcIndTemplateScript.js?tag=procIndTwo&lang=at&outdent=true'></script>

**Remark:** the processing indicator is removed when a template refresh occurs.
If you want to keep the processing indicator through refreshes, you have to bind its presence to the data-model, as explained in next section.


#### Bound to the data-model

It is possible to use the **`bindProcessingTo`** property of a section configuration in order to specify a piece of data-model to which the presence of a processing indicator should be bound.
Moreover, you can also set a message to be displayed on the overlay in the **`processingLabel`** property.

In the template:

<script src='%SNIPPETS_SERVER_URL%/snippets/github.com/ariatemplates/documentation-code/snippets/templates/domInteractions/ProcIndTemplate.tpl?tag=procIndThree&lang=at&outdent=true'></script>

In the template script:

<script src='%SNIPPETS_SERVER_URL%/snippets/github.com/ariatemplates/documentation-code/snippets/templates/domInteractions/ProcIndTemplateScript.js?tag=procIndThree&lang=at&outdent=true'></script>

The information on whether the processing indicator is displayed or not becomes persistent through template refreshes and can be set also from a [module controller](controllers) or a [flow controller](flow_controllers).


### DOM overlay utility

Aria Templates has a [DOM overlay utility](http://ariatemplates.com/api/#aria.utils.DomOverlay) that allows to set a processing indicator on top of any DOM element.
This could be useful if you want the overlay on the whole page or on any element that is defined outside of Aria Templates (for example in the `index.html` file of your application).

However, make sure you use the API described [above](#on-dom-element-wrappers) if you want to set a processing indicator on top of elements defined within your templates.
**The utility function should only be used for elements outside Aria Templates.**

Suppose you have the following <`body`> in your `index.html` file:

<script src='%SNIPPETS_SERVER_URL%/snippets/github.com/ariatemplates/documentation-code/snippets/templates/domInteractions/index.html?lang=html5'></script>

You can use the following methods (inside any class, module controller, flow controller, template script, ...)

<script src='%SNIPPETS_SERVER_URL%/snippets/github.com/ariatemplates/documentation-code/snippets/templates/domInteractions/ProcIndTemplateScript.js?tag=procIndFour&lang=at&outdent=true'></script>


### Example

<iframe class='samples' src='%SNIPPETS_SERVER_URL%/samples/github.com/ariatemplates/documentation-code/samples/templates/domInteractions/processingIndicator/' ></iframe>

You can see that, after refreshing the template, the loading indicator on the the section with id `"mySecondSection"` persists because it is bound to the data-model.
All the other loading indicators are removed.


## Drag and Drop

Aria Templates allows to set an element as draggable and to specify how the element should be dragged.
`aria.utils.dragdrop.Drag` is the class that defines the element that can be dragged, called _draggable element_ from now on.

<script src='%SNIPPETS_SERVER_URL%/snippets/github.com/ariatemplates/documentation-code/snippets/utils/dragdrop/DragScript.js?tag=dragSyntax&lang=javascript&outdent=true&noheader=true'></script>

The constructor accepts the following arguments

* **`element`** is either an id or a DOM element.
* **`params`** is an optional configuration object (also described [here](http://ariatemplates.com/api/#aria.utils.dragdrop.DragDropBean:DragCfg)) containing:

	* **`handle`**
    either an id or a DOM element, this is the element from which the user can start dragging.
    If not specified it defaults to the draggable element.
    In the case of a dialog, draggable element is the whole dialog window, while handle is just the title bar.

	* **`cursor`**
    css cursor property to be added on the draggable element or the handle.
    If not specified no default value will be added.

	* **`constrainTo`**
    either an id or a DOM element, whose boundaries will be used to constrain the movement of the dragged element.
    It can also be the special value `aria.utils.Dom.VIEWPORT` if you want to constrain the movement to the viewport.

	* **`axis`** can be either "x" or "y" in order to force one-dimensional movement.

	* **`proxy`**
    configuration object for the element that moves with the mouse.
    This can contain:

		* **`type`** Type of proxy.
      Possible values are the classes in package `aria.utils.overlay` (in particular `CloneOverlay` or `Overlay`).
		* **`cfg`** Argument passed to the constructor of the overlay described by _type_.

`aria.utils.dragdrop.Drag` is observable through the events

* **`dragstart`**
  raised when the user starts moving the draggable element.
  More in details this is not raised on mousedown but on the first mousemove after a mouse down, in other words when the position actually changes.

* **`dragend`**
  raised when the user stops moving the draggable element.
  This corresponds to a mouseup after a mousemove. This event will also contain the current position of the draggable element.


### Examples

#### HTML element that moves while dragged

![Draggable Element](../images/draggable_element.png)

<script src='%SNIPPETS_SERVER_URL%/snippets/github.com/ariatemplates/documentation-code/snippets/utils/dragdrop/Drag.tpl?tag=drag1html&lang=html5&outdent=true&noheader=true'></script>

<script src='%SNIPPETS_SERVER_URL%/snippets/github.com/ariatemplates/documentation-code/snippets/utils/dragdrop/DragScript.js?tag=drag1js&lang=javascript&outdent=true&noheader=true'></script>


#### HTML element that can be dragged only from an handle

![Dialog Handle](../images/dialog_handle.png)

<script src='%SNIPPETS_SERVER_URL%/snippets/github.com/ariatemplates/documentation-code/snippets/utils/dragdrop/Drag.tpl?tag=drag2html&lang=html5&outdent=true&noheader=true'></script>

<script src='%SNIPPETS_SERVER_URL%/snippets/github.com/ariatemplates/documentation-code/snippets/utils/dragdrop/DragScript.js?tag=drag2js&lang=javascript&outdent=true&noheader=true'></script>


#### Proxy element, box with borders

![Clone Element](../images/box_clone.png)

<script src='%SNIPPETS_SERVER_URL%/snippets/github.com/ariatemplates/documentation-code/snippets/utils/dragdrop/Drag.tpl?tag=drag1html&lang=javascript&outdent=true&noheader=true'></script>

<script src='%SNIPPETS_SERVER_URL%/snippets/github.com/ariatemplates/documentation-code/snippets/utils/dragdrop/DragScript.js?tag=drag3js&lang=javascript&outdent=true&noheader=true'></script>

The default classname of the overlay element is xOverlay.
Its style can be changed throught the skinning system by setting the desired background color, opacity and border corresponding to the following skin properties:

    aria.templates.general.overlay.backgroundColor=#ddd
    aria.templates.general.overlay.opacity=40
    aria.templates.general.overlay.border=1px solid black


#### Proxy, clone of the draggable element

![Transparent Clone](../images/transparent_clone.png)

<script src='%SNIPPETS_SERVER_URL%/snippets/github.com/ariatemplates/documentation-code/snippets/utils/dragdrop/Drag.tpl?tag=drag1html&lang=javascript&outdent=true&noheader=true'></script>

<script src='%SNIPPETS_SERVER_URL%/snippets/github.com/ariatemplates/documentation-code/snippets/utils/dragdrop/DragScript.js?tag=drag4js&lang=javascript&outdent=true&noheader=true'></script>

The default opacity is 0.5/1.


### Example

<iframe class='samples' src='%SNIPPETS_SERVER_URL%/samples/github.com/ariatemplates/documentation-code/samples/utils/dragdrop/' ></iframe>
