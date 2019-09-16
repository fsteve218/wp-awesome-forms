WP Awesome Forms
Steve Fraser
fsteve218@gmail.com
https://github.com/fsteve218/wp-awesome-forms

WordPress Awesome form is a forms rendering engine built in both php & javascript, that can render both server side and browser side forms.

It can be used independantly but is also packaged with wordpress additions.

Architecture
The basic concept is to use a json form defintion combined with handlebar templates. The rendering engine is written in both php & js, as close to the same as possible.

Rendering

The rendering begins by creating a “form” object, which uses the json config to create an array of fields. The “render” method renders a list of fields, then includes them in the form container.

The “field” object can be extended – fieldset & duplicator are included, fieldeditor is in development. The object calls a “render” method, which combines a “container” template (defaults to “field” or the input name) with an “input” template defining the specific input.

The field object contains an “input” object which controls the specific template for the input, as well as things like sanitizing options.

Javascript functions are provided to handle form rendering so that the form & field objects are associated with a DOM element, and to provide access to the “$.getField()” and “$.getForm()” methods.

Submission

The forms are submitted via ajax by default. There is both an advanced and a simple form handler.

The advanced handler uses the created “form” object, in either formData or JSON mode. This is mainly useful if there is a challenge in formatting the data – the “field” object can contain a value different from what is visible, or if you wish to destroy the DOM, like in an admin interface using localStorage or similiar. it is also useful as it can be used with a JSON api if the form’s “format” is set to JSON.

For the most part, the simple handler is adequate. This uses $.serialize & $.ajax.

Both submission methods trigger the “complete” event on the form.

Completion

After the form recieves a response, the “complete” action is triggered (or failed). This is handled by $.processJSON by default, which handles things like messages & dom addtions/removals, as well as other things.

Engine

The engine uses as Form, Field, and Input object, with Duplicator & Fieldset extending the Field object.

The “Form” is as so:

Progress Variables get previous (field), get percent, get next (field). these are used if you are “walking” a form.
jQuery Variables: get $form, get $messages, get $bd
Field Navigation: allFields, nextField, findField (using path), findByName
Rendering: getContext, getTemplate, render, renderFields, walkFields (render one at a time)
Validation: get invalid (checks .invalid on each field), renderValidation
Form Data: getFormData (get a form data object using field path & value), getFormJSON (gets values in json format, useful with a REST API)
submission: presubmit (triggers a jquery “presubmit” event on form & all fields), submit (submits forms using axios)
loading: addLoading, removeLoading, appendLoading, getLoading (color & type from config)
form messages: message (display a message with a bootstrap color)
values: get value, set(fieldName, value) – set form values or a field value
JSON: loadFromString (load a config from a form’s id string), save (build a json config for the current form. could then be saved to a file or in localStorage)
Field Object

path: the “path” is a json object style variable path for the current element, if it is embedded in fieldsets, eg. “step1.details.first_name” would be the first_name field in the details fieldset of the step1 fieldset. These paths are created using the fields “name”.

get jsonPath (find the path in json notation, eg. step1.details.first_name), get path (return the full path), getPath(skipFieldsets=false) (get the path, optionally including fieldsets in the path or skipping it if applicable, which is used for getting the path submitted to the server)
value: the field object can hold a value independant of the dom.
set value (sets the field’s value internally), get realValue (get the value from dom), get value (get the field’s value. will always use internal value first if possible, otherwise return the internal value. since it is potentially confusing, the internal value is not used by default, it is )
templates: get inputSrc, get container, set/get containerName, render, isRendered
keyboard: keyboardNav
submit: submit (submits the field, uses data-route attributes on fields or similiar)
loading: removeLoading, addLoading
variables: sel, $append, $f, $field, $errors, $first, $input, $last, $lastRow, $text, $textarea, $other, form
validation: requiredLength, validate, renderValidation get invalid
Typical Usage
typically you would use the form or field object on a change event or similar.

To retrieve the object, use “$.getForm()” or “$.getField()” on any element within the form or the form element itself.

A typical usage would be to set a value for a field discretely. Typically this would be done with a hidden input. Instead, the value can be stored directly on the field object. eg. $(e.target).getField().value = ‘hello world’.

Another usage would be to re-render the field after changing it’s configuration, for example if you changed the “options” of a select field.

The field has a “context” property which contains the initial json definition of the field, that is used as the context for the handlebars template when rendering. so for example, to change a select field:

$(‘.trigger’).on(‘click’, function(e) {
const field = $(e.target).getField();
const options = [{name:’one’,value:’One’},{name:’two’,value:’Two’}];
field.context.options = options;
field.render( field.$field);
field.$field.trigger(render);
})

The “field.render” method takes a html/jquery element or selector. In this example, i used the link to the field’s dom element, which will re-render the same field. I then trigger the “render” action on the DOM for any scripts that use it.

In this way, the advanced handler becomes a useful tool when forms need to be altered in dynamic ways.

WordPress Integrations
WordPress integrations as provided with template tags, shortcodes, and filters.

Rendering a Form

use <?php the_form( ‘form_id’); ?> or [the_form id=”form_id”], where form_id is the filename without the extension.

Storing Forms & Templates

Form json files can be stored in “/waf-json” in the theme or child theme. Templates go in “/waf-templates”.

Use the “waf_json_path” and “waf_template_paths” filters to use your own location, like in a plugin.

Filtering the Form Config

The forms can be filtered with the following hooks:

waf_form_data (all forms)
waf_form_args (all forms)
waf_{form_id}_form_data
waf_{form_id}_form_args
waf_field_data (filters inputs)
waf_{input_type}_field_data
