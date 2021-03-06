# Field Type Basics

Building a field type for use with streams is simple if you are familiar with PHP and PSR-0 autoloading. Each field type is a folder with a php file containing a class that has information about your field type as well as methods for things like the form output. This page outlines the basic structure of a field type file.

* {{ docs:id_link title="Basics" }}
* {{ docs:id_link title="Field Parameters" }}
* {{ docs:id_link title="Languages in Field Types" }}
* {{ docs:id_link title="Validation" }}
* {{ docs:id_link title="Working With File Uploads" }}
* {{ docs:id_link title="CSS/JS Files" }}
* {{ docs:id_link title="Using View Files" }}
* {{ docs:id_link title="Field AJAX Functions" }}

</div>
<div class="doc_content">

## Basics

To get started, create a folder and PSR-0 autoloaded class with your chosen slug. For this example, we are going to use the Email field type, which has a slug called "email".
 
	email/src/Pyro/FieldType/Email.php
 
<p>You can put this file in  <strong>addons/shared\_addons/field\_types</strong> or <strong>addons/[site-slug]/field\_types</strong> and streams will recognize and use it automatically.</p> 
 
<p>Each field type has a basic structure with data that needs to be there for it work correctly. Here is an example of a very basic field type:</p> 

    <?php namespace Pyro\FieldType;

    use Pyro\Module\Streams\FieldType\FieldTypeAbstract;

    class Email extends FieldTypeAbstract
    {
        public $field_type_slug				= 'email';

        public $db_col_type					= 'string';

        /**
        * Output form input
        *
        * @return string
        */
        public function formInput()
        {
            $options['name'] 	= $this->getFormSlug();
            $options['id']		= $this->getFormSlug();
            $options['value']	= $this->value;

            return form_input($options);
        }
    }

<p>As you can see, we have a class name of <strong>Yourslug</strong>. Inside we have some basic class variables:</p>
 
<table cellpadding="0" cellspacing="0" class="docs_table"> 
 <thead> 
  <tr> 
   <th width="200"> 
    Variable</th> 
   <th> 
    Description</th> 
  </tr> 
 </thead> 
 <tbody> 
  <tr> 
   <td>field\_type\_slug</td> 
   <td>The field slug (the same one you are using for the class name and folder/file names).</td> 
  </tr> 
  <tr> 
   <td>db\_col\_type </td> 
   <td>Type of MySQL column that Streams should create to store the data for this field. Any type can be used, but the most common are varchar, text, and int.</td> 
  </tr> 
</tbody> 
</table> 
 
<p>Aside from the class variables, one method is necessary &ndash; <strong>formInput</strong>. This method is called when building the form and allows you to include logic and customized input for your fields. In this case, we are just returning a basic input. You can find more about this method on the {{ link title="Methods" uri="developers/addons/developing-field-types/methods" }} page.</p>
 
<p class="note">Streams uses <a href="http://codeigniter.com/user_guide/helpers/form_helper.html">CodeIgniter's Form Helper</a> when creating inputs, and it is available in third party field types as well.</p> 

### Tapping Into CodeIgniter

If you want to use the CodeIgniter super object in your field types, you can access it via the <strong>CI</strong> class variable. This is added to your field type automatically so you do <strong>not</strong> need to use the <strong>get_instance()</strong> syntax.

	$this->CI->load->library('Typography');

### Optional Class Properties

<p>You can also add some other class properties to change how Streams interacts with the field type.</p>

<table cellpadding="0" cellspacing="0" class="docs_table"> 
 <thead> 
  <tr> 
   <th width="120">Variable</th>
   <th width="120">Value</th> 
   <th>Description</th> 
  </tr> 
 </thead> 
 <tbody> 
  <tr> 
   <td>extra_validation</td> 
   <td>string</td>
   <td>Validation for required, unique, etc. is added automatically with Streams. However, you might need to add some extra validation from time to time. You can do that by adding a class variable named extra_validation.</td> 
  </tr> 
  <tr> 
   <td>input_is_file </td>
   <td>bool</td>
   <td>Some field types work with files (such as the image field type). When creating a field type that uses a file, make sure to add a class variable called input_is_file and set it to TRUE.</td> 
  </tr>
  <tr>
  	<td>alt_process</td>
  	<td>bool</td>
  	<td>Sometimes you don't want to actually have a column created for your field type (such as in the case of having a related rows table). Setting this to true will tell Streams to ignore the column in the stream when it needs to because there is none.</td>
  </tr>
   <tr>
  	<td>version</td>
  	<td>number</td>
  	<td>The current version number of the field type.</td>
  </tr>
   <tr>
  	<td>author</td>
  	<td>array</td>
  	<td>An array with two keys: <strong>name</strong> (name of the author) and <strong>url</strong> URL to the author's website.</td>
  </tr>

</tbody> 
</table>

## Field Parameters

<p>Each field can have custom or prefab field parameters. For instance, many fields, such as the text field take advantage of the max_length field parameter. However, you can make them yourself to give your users added functionality.</p> 
 
<p>Custom fields (preset and ones you make) are activated by putting a class variable into your field type class named <strong>custom_parameters</strong>.</p> 
 
	public $custom_parameters = array('max_length', 'my_custom_one'); 
 
<h3>Preset Field Parameters</h3> 
 
<table cellpadding="0" cellspacing="0" class="docs_table"> 
 <thead> 
  <tr> 
   <th width="200"> 
    Parameter</th> 
   <th> 
    Description</th> 
  </tr> 
 </thead> 
 <tbody> 
  <tr> 
   <td>max\_length</td> 
   <td>Collects data on the max length of a field.</td> 
  </tr> 
  <tr> 
   <td>default\_value</td> 
   <td>Collects data on the default value of a field. This is only a simple text input, so if you need a special default input, you should create your own.</td> 
  </tr> 
</tbody> 
</table> 
 
<h3 id="creating-custom-field-parameters">Creating Custom Field Parameters</h3> 
 
<p>To create custom field parameters, you need to</p>

1. add them to the <strong>custom_parameters</strong> class variable (detailed above)
2. specify a parameter name
3. specify a parameter input.
 
To specify the label name of the parameter, just add a new language key in the format of:

    $lang['streams:{your_field_type_slug}.{your_param_slug}'] = 'Label';

So if you have a parameter default_setting for your Setting field type, your language variable would look like this:

    $lang['streams:setting.default_setting'] = 'Default Setting';
 
<p>To specify an input, create a method in your field class with the prefix <strong>param_</strong></p> 
 
	public function paramMyCustomOne()
	{
	  // Return the form input
	}
 
<p>It takes one parameter &ndash; a string that is the current value of the custom field parameter.</p>

### Adding Instructions to Your Field Parameter

If you want to take it one step further, you can return an array (instead of a string) of the input and the instructions for that input:

    public function paramChoiceData()
    {
      return array(
          'input'        => form_textarea('choice_data', $value),
          'instructions' => $this->CI->lang->line('streams.choice.instructions')
        );
    }

## Languages in Field Types

Streams fully supports language files in field types. The canonical language for Streams is English, so we'll use English in the following examples.

Inside your field type folder, you'll need a folder called **language**. Inside this, you can add the different language folders. In this case, English.

Streams will recognize and load a language file with the same slug as your field type. So if we are creating a language file for the Email field type, the structure would look like this:

	email
		- language
			- english
				--- email_lang.php

Inside the <strong>email_lang.php</strong> file, you'll need an array like this:
	
	$lang['streams:email.name'] = 'Email';

Each language item should follow the convention <strong>streams.yourslug.lang_slug</strong>. The only one that is required is <strong>streams.yourslug.name</strong> which will be used as the name of the field type.

### Custom Parameter Titles

If you are using {{ link title="Custom Parameters" uri="developers/addons/developing-field-types/structure#creating-custom-field-parameters" }} for your field type, you need to put the names of your fields in this language file. Simply use the slugs of your parameters as the language slugs. They will automatically be used.

In your field type:

	public $custom_parameters = array('choice_data', 'choice_type');

In your language file:

	$lang['streams:choice.choice_data'] = 'Choice Data';
	$lang['streams:choice.choice_type'] = 'Choice Type';

## Validation
 
Validation takes several forms for PyroCMS field types.

* **Assignment-added validation**. This takes the form of **required** and **unique**, which are determined by the field assignment.
* **Standard field type validation**. This is the standard validation rules in the actual field type (see below).
* **Custom field type validation**. This is the custom validation functionality in the **valiation()** function (see below).

With these three ways to incorporate validation, you have all the tools you need to validate pretty much anything!

### Standard Field Type Validation

Standard field type validation is added as a class variable named **extra_validation**: 
 
    public $extra_validation = 'numeric|integer'; 
 
Any of [CodeIgniter's Form Validation](http://codeigniter.com/user_guide/libraries/form_validation.html) rules can be used here, separated by a pipe character.

### Custom Field Type Validation

Sometimes, the standard validation rules aren't enough for your input, and you need some custom validation logic. To add some custom validation logic, just a public **validate** function to your field type. It takes three parameters:

<table cellpadding="0" cellspacing="0" class="docs_table"> 
	<thead> 
  <tr> 
   <th width="100"> 
    Parameter</th> 
   <th> 
    Description</th> 
  </tr> 
 </thead> 
 <tbody> 
  <tr>
      <td>value</td>
      <td>The value submitted to the form for your field.</td>
  </tr>
  <tr>
      <td>mode</td>
      <td>Either 'new' or 'edit', depending on if the form is editing an entry or creating a new one.</td>
  </tr>
  <tr>
      <td>field</td>
      <td>The field instance object.</td>
  </tr>
 </tbody>
</table>

If the data fails validation, you can return an error string. A return of **null** or **true** indicates that your validation has passed the test. Here's a simple example: 

    public function validate()
    {
        if ($this->value != 'the value we want')
        {
            return 'The '.$this->field->field_name.' field needs to be the value we want!';
        }

        return true;
    }

Remember, you can still access all the <var>$_POST</var> variables. If you need the entry ID simply access the entry object.

    $this->entry->getKey(); // Returns the ID or null
 
## Working With File Uploads

<p>Some field types work with files (such as the image field type). When creating a field type that uses a file, make sure to add a class variable called <strong>input_is_file</strong> and set it to true.</p> 
 
    public $input_is_file = true;
 
<p>This will make sure things like required fields works correctly, since it needs to check the `$_FILE` variable, not `$_POST`.</p> 

## CSS/JS Files

Often times you need to use additional assets in your field type. This could be a CSS file or a view. Streams is set up to allow you to pull in these files without having to figure out where your field type is in the file system.

If you need to add CSS or Javascript on the back end of PyroCMS, you can put them into a <strong>css</strong> or <strong>js</strong> folder in the field type, and add them by adding a function called <strong>event()</strong>:

    public function event()
    {
        $this->css('example.css', 'email'));
        $this->js('example.js', 'email'));
    }

The above code will add the _example.css_ and _example.js_ files to the admin area from the email field type.

## Using View Files

If you'd like to load a view file from your field type, create a <strong>views</strong> folder in your field type folder and place your view file in there. You can call your view file like this:

    $this->view('view_file', $data, $field_type_slug);

The first two parameters are the same as CodeIgniter's `$this->load->view()` function. The third parameter is the field\_type\_slug of the field type the view belongs to if different than current field type.

## Field AJAX Functions

If you need to have your field type access an ajax function, you can create a function in your field type prefixed with <strong>ajax</strong>.

    public function ajaxMyFunction()
    {
        // AJAX functionality here.
    }

You can then access that function via the following URL:
    
    http://example.com/streams_core/public_ajax/field/[field_type_slug]/my_function

Remember, this function is publicly accessible, so if you need data checks you need to do those yourself. (Ex: checking for a logged in user). The reason these ajax functions are kept public since we can't anticipate where the entry form using the field type will be used. It could be public or private.