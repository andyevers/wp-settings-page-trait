# WP_Settings_Page_Trait

Easily create a WordPress settings page to manage options for your theme or plugin on the admin dashboard. Just use the WP_Settings_Page_Trait on your page class and call `register_settings_fields()` on the 'admin_menu' action.

This settings page works by creating an option inside the __wp_options__ table and saving each field as serialized data in the option value. 

So if you set your option as 'my_info' and you have a field called 'first_name', that value will be accessible using `get_option('my_info')['first_name']`


## Usage
Here's a basic usage example in a page class:
```PHP
class Example_Settings_Page
{
    use WP_Settings_Page_Trait;
     
    public static function get_option()
    {
        return 'my_option';
    }
    public static function get_slug()
    {
        return 'my-slug';
    }
    protected function register_fields()
    {
        $this->start_fields_section('personal_info', 'Personal Info');
        $this->field_input('first_name', 'First Name', [
            'type' => 'text'
        ]);
        $this->end_fields_section()
    }
    public function render()
    {?>
        <form <?php echo $this->get_form_attribute_string(); ?>>
            <?php $this->do_fields_section('personal_info'); ?>
        </form>
    <?php
    } 
}
```
## Required Methods
* __public static function get_option()__: Returns a string that is the option_name that gets stored in wp_options.
* __public static function get_slug()__: Returns a string that is the URL slug for the settings page.
* __protected function register_fields()__: Registers each setting field that gets stored inside the option_value in wp_options

## Registering Settings

### Register Sections
Each field must be registered inside a fields section which is defined using `start_fields_section()` and `end_fields_section()`

```PHP 
protected function register_fields()
{
    $this->start_fields_section('section_id', 'Section Label');
    // register fields here...
    $this->end_fields_section();
}
```
### Register Fields
Use the field methods listed in the __Field Methods__ section to register fields.
```PHP 
protected function register_fields()
{
    $this->start_fields_section('section_id', 'Section Label');

    // Creates <input type="text" name="my_option[first_name]">
    $this->field_input('first_name', 'First Name', [ 
        'type' => 'text'
        'description' => 'Type your first name here',
    ])

    // Creates <select name="my_option[favorite_animal]">[option els]</select>
    $this->field_select('favorite_animal', 'Favorite Animal', [
        'options' => [
            'cat'   => 'Cat',
            'dog'   => 'Dog',
            'mouse' => 'Mouse'
        ],
        'description' => 'Pick your favorite animal.',
    ]);
    $this->end_fields_section();
}
```

## Field Section Methods
* __start_fields_section($section_id, $label, $section_options)__: Used to specify the beginning of a group of fields that can later be accessed using `do_fields_section`. Used inside `register_fields()` function.
    * __$section_id__: {String} the ID of the section that will be used to produce the fields using `$this->do_fields_section($section_id)`.
    * __$label__: {String} the heading that will appear as an h2 element above the section fields
    * __$section_options__: {Array} accepted keys: 
        * __'field_options'__: {Array} options added to all fields in the section
        * __'description'__: {String} Description that appears between the section heading and the fields
        * __'callback'__: {Function} Callback that is fired after the heading is echoed
* __end_fields_section()__: Used to specify the end of a group of fields. Used inside `register_fields()` function.
* __do_fields_section($section_id)__: Echos the field elements for each field registered for the $section_id provided in a form table. Use inside a function that renders the page HTML.


## Field Methods
Use these methods to register settings and create fields. These must be used inside `start_fields_section()` and `end_fields_section()`. All fields accept the args __$id__, __$label__, & __$options__. values can be accessed using `get_option($option_name)[$id]` where $option_name is the string returned by __self::get_option()__ and $id is the field id.
* __field_input($id, $label, $options)__: Creates an input element field. *Required Options: 'type'*
    * __$id__: {String} id of the field that is stored in the wp_option array
    * __$label__: {String} HTML Label for the input element.
    * __$options__: {Array} See __Field Options__ section for available options.
* __field_inputs()__: Creates multiple input elements. Can be type 'radio' or 'checkbox'. __Note__: checkbox types contain multiple values and are stored as an array. *Required Options: 'type', 'options'*
* __field_select()__: Creates select dropdown field. *Required Options: 'options'*
* __field_check()__: Creates a single 'checkbox' type input
* __field_textarea()__: Creates a textarea element
* __field_wysiwyg()__: Creates a WYSIWYG editor. $options for this field are the settings for the wp_editor() function. see options at https://developer.wordpress.org/reference/classes/_wp_editors/parse_settings/


## Field Options
The __$options__ argument of each field is used to customize the field by adding attributes, descriptions, etc... Here are a few examples of accepted args:
* __'description'__: The description HTML that appears next to the field.
* __'default'__: The default value of a field if no value is already set.
* __'type'__: Required for field_input and field_inputs. specifies the input type attribute.
* __'class'__: Class attribute added to the form element field.
* __'options'__: Required for field_inputs and field_select. Assoc array that specifies the value => label for each option.
* __'row_class'__: Class attribute added to the tr element that holds the setting.
* __'label_for'__: Sets the for attribute on the input's label. defaults to the ID of the field. use null to remove the attribute.

Other options will be used as attributes if it is an allowed attribute based on the field type. For a list of allowed attributes for field types, see function `extract_atts_by_type()`

## Other Methods
* __get_settings($field_id = null)__: Returns current value of the $field_id provided. If $field_id === null, returns assoc array containing all field values.
* __get_form_attribute_string($addon_atts = [])__: Returns a string containing the required 'method' and 'action' atts for an options form. $addon_atts is an assoc array for any additional attributes.
* __register_settings_fields()__: Registers option specified in get_option() along with all field settings. This should be called from your page class instance in the 'admin_menu' action.

## Example
```PHP
class Example_Settings_Page
{
    use WP_Settings_Page_Trait;

    public static function get_option()
    {
        return 'my_option';
    }
    public static function get_slug()
    {
        return 'my-slug';
    }
    protected function register_fields()
    {

        // SECTION GENERAL FIELDS
        //-----------------------------------------------

        $this->start_fields_section('general_fields', 'General Fields');

        $this->field_input('first_name', 'First Name', [
            'type' => 'text',
        ]);

        $this->field_check('cool_or_not', 'Are You Cool?', [
            'description' => 'Check = Cool'
        ]);

        $this->end_fields_section();


        // SECTION FAVORITE THINGS
        //-----------------------------------------------

        $this->start_fields_section('favorite_things', 'Favorite Things');

        $this->field_inputs('favorite_color', 'Favorite Color', [
            'type'        => 'radio',
            'description' => 'Pick your favorite color.',
            'options'     => [
                'red'   => 'Red',
                'blue'  => 'Blue',
                'green' => 'Green'
            ]
        ]);

        $this->field_select('favorite_animal', 'Favorite Animal', [
            'description' => 'Pick your favorite animal.',
            'options'     => [
                'cat'   => 'Cat',
                'dog'   => 'Dog',
                'mouse' => 'Mouse'
            ]
        ]);

        $this->end_fields_section();
    }

    public function render()
    { ?>
        <div class="wrap">

            <h1>My Settings Page</h1>

            <form <?php echo $this->get_form_attribute_string(); ?>>

                <?php $this->do_fields_section('general_fields'); ?>
                <hr>
                <?php $this->do_fields_section('favorite_things'); ?>

                <div>This is such an awesome settings page!</div>

                <?php submit_button(); ?>
            </form>
        </div>
    <?php
    }
}


add_action('admin_menu', function () {

    $settings_page = new Example_Settings_Page();
    $settings_page->register_settings_fields();

    add_menu_page(
        'Some Page Title',
        'Some Menu Title',
        'manage_options',
        $settings_page::get_slug(),
        [$settings_page, 'render']
    );
});
```
