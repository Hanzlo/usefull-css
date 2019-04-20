# Forms

FER+ form builder creates forms that can be easily modified and reused.

- [Quick start](#quick-start)
- [Form](#form)
- [Form builder](#form-builder)
- [View](#view)

## Quick start

Creating Form classes with a simple artisan command:

```bash
php artisan fer:make:form PostForm --fields="title:text, description:texarea"
```

Form is created in app/Forms/PostForm.php path with the following content:

```php
<?php

namespace App\Forms;

use Fer\Admin\Form\Form;
use Fer\Contracts\Form\Field;

class PostForm extends Form
{
    /**
     * {@inheritdoc}
     */
    public function init(array $options = [])
    {
        $this->text('title');
        $this->texarea('description');
    }
}
```

After that instantiate the class in the controller and pass it to the view:

```php
<?php

namespace App\Http\Controllers;

use App\Forms\PostForm;
use Fer\Contracts\Form\FormBuilder;

class PostController extends Controller
{
    /**
     * Show the form for creating a new resource.
     * 
     * @param \Fer\Contracts\Form\FormBuilder $builder
     * 
     * @return \Illuminate\Http\Response
     */
    public function create(FormBuilder $builder)
    {        
        $form = $builder->post(PostForm::class)
            ->setAction(route('posts.store'))
            ->build();

        return view('posts.create', compact('form'));
    }
}
```

And than print the form in view:

```html
<!-- resources/views/posts/create.blade.php -->

@extend('layout')

@section('content')
    {!! $form->render() !!}
@endsection
```

## Form

`Form` class is the basis of the FER+ form package. It contains all logic that is handled from instantiation to rendering. By default, it should contain `init` method that will be called on the form instantiation through `FormBuilder`, and here you can create form fields and set cancel button url.

```php
<?php

namespace App\Forms;

use Fer\Admin\Form\Form;
use Fer\Contracts\Form\Field;

class PostForm extends Form
{
    /**
     * {@inheritdoc}
     */
    public function init(array $options = [])
    {
        $this->text('title')
            ->setGroupClass('from-group')
            ->setClass('from-control')
            ->hideLabel();

        $this->showCancelButton('/url');
    }
}
```

Fields all share these default options:

- **setGroupAttributes(array $attributes)** - Sets the HTML attributes on this field group.
- **setGroupAttribute(string $name, string $value = null)** - Sets the HTML attribute on this field group.
- **setGroupClass(string $class)** - Sets the HTML class attribute on this field group.
- **addGroupClass(string $class)** - Adds the HTML class attribute on this field group.
- **setGroupData(string $name, string $value = null)** - Sets the HTML data attribute on this field group.
- **setLabelAttributes(array $attributes)** - Sets the HTML attributes on this field label.
- **setLabelAttribute(string $name, string $value = null)** - Sets the HTML attribute on this field label.
- **setLabelClass(string $class)** - Sets the HTML class attribute on this field label.
- **addLabelClass(string $class)** - Adds the HTML class attribute on this field label.
- **showLabel()** - Show label on this field.
- **hideLabel()** - Hide label on this field.
- **setLabelText(string $text)** - Sets the label text on this field.
- **setRequired(bool $required = true)** - Set required on this field.
- **setAttributes(array $attributes)** - Sets the HTML attributes on this field.
- **setAttribute(string $name, string $value = null)**- Sets the HTML attribute on this field.
- **setClass(string $class)** - Sets the HTML class attribute on this field.
- **addClass(string $class)** - Adds the HTML class attribute on this field.
- **setData(string $name, string $value = null)** - Sets the HTML data attribute on this field.
- **setHelpAttributes(array $attributes)** - Sets the HTML attributes on this field help.
- **setHelpAttribute(string $name, string $value = null)** - Sets the HTML attribute on this field help.
- **setHelpClass(string $class)** - Sets the HTML class attribute on this field help.
- **addHelpClass(string $class)** - Adds the HTML class attribute on this field help.
- **setHelpText(string $text)** - Sets the help text on this field.
- **setErrorAttributes(array $attributes)** - Sets the HTML attributes on this field error.
- **setErrorAttribute(string $name, string $value = null)** - Sets the HTML attribute on this field error.
- **setErrorClass(string $class)** - Sets the HTML class attribute on this field error.
- **addErrorClass(string $class)** - Adds the HTML class attribute on this field error.
- **showError()** - Show error on this field.
- **hideError()** - Hide error on this field.
- **hasError()** - Check if there is any error on this field.
- **getError()** - Return the error message on this field.
- **setOptions(array $options)** - Sets the value options for this field, typically for select and radios.
- **fill($value)** - Fills this field with the value.
- **getValue(bool $force = false)** - Returns the value for this field.

All FER+ custom form fields has default options already set, so there is almost no need to use most of them.

### Custom fields

- [Checkboxes field](#checkboxes-field)
- [Checkbox field](#checkbox-field)
- [Date field](#date-field)
- [Dropdown field](#dropdown-field)
- [File field](#file-field)
- [Files field](#files-field)
- [Hidden field](#hidden-field)
- [Image field](#image-field)
- [Images field](#images-field)
- [Number field](#number-field)
- [Password field](#password-field)
- [Radios field](#radios-field)
- [RichEditor field](#richEditor-field)
- [Text field](#text-field)
- [Textarea field](#textarea-field)

#### Checkboxes field

Renders a list of checkboxes.

- **name** - Sets the field name. Required.
- **label** - Sets the field label text. Default is null.
- **options** - Sets the field options. Default is empty array.
- **selected** - Sets the field selected option. Default is null.

```php
<?php

$options = [
    1 => 'Foo',
    2 => 'Bar',
    3 => 'Bazz',
];

$this->checkboxes('name', 'Label', $options, 2);
```

In addition to the default options, checkboxes field has the following options:

- **setInline(bool $inline = true)** - Are items aligned vertically or horizontally.

#### Checkbox field

Renders a single checkbox.

- **name** - Sets the field name. Required.
- **label** - Sets the field label text. Default is null.
- **value** - Sets the field value option. Default is 1.
- **checked** - Sets the field ckecked option. Default is null.

```php
<?php

$this->checkbox('name', 'Label', 1, false);
```

#### Date field

Renders a text input used for selecting date.

- **name** - Sets the field name. Required.
- **label** - Sets the field label text. Default is null.

```php
<?php

$this->date('name', 'Label');
```

In addition to the default options, date field has the following options:

- **setFormat(string $format)** - Sets the date format.

#### Dropdown field

Renders a dropdown with specified options.

- **name** - Sets the field name. Required.
- **label** - Sets the field label text. Default is null.
- **options** - Sets the field options. Default is empty array.
- **selected** - Sets the field selected option. Default is null.

```php
<?php

$options = [
    1 => 'Foo',
    2 => 'Bar',
    3 => 'Bazz',
];

$this->dropdown('name', 'Label', $options, 2);
```

In addition to the default options, dropdown field has the following options:

- **setEmptyText(string $text)** - If provided, added to the start of select as empty value.

#### File field

Render a control to upload a single file.

- **name** - Sets the field name. Required.
- **label** - Sets the field label text. Default is null.

```php
<?php

$this->file('name', 'Label');
```

In addition to the default options, file field has the following options:

- **setAllowedExtensions(array $extensions)** - Sets the field allowed extensions.

#### Files field

Render a control to upload a multiple files.

- **name** - Sets the field name. Required.
- **label** - Sets the field label text. Default is null.

```php
<?php

$this->files('name', 'Label');
```

In addition to the default options, files field has the following options:

- **setAllowedExtensions(array $extensions)** - Sets the field allowed extensions.

#### Hidden field

Render a hidden text input.

- **name** - Sets the field name. Required.
- **value** - Sets the field value. Default is null.

```php
<?php

$this->hidden('name', 'value');
```

#### Image field

Render a control to upload a single image file.

- **name** - Sets the field name. Required.
- **label** - Sets the field label text. Default is null.

```php
<?php

$this->image('name', 'Label');
```

In addition to the default options, image field has the following options:

- **setAllowedExtensions(array $extensions)** - Sets the field allowed extensions.
- **setThumbnailWidth(int $width)** - Sets the field thumbnail width. Default is 400.
- **setThumbnailHeight(int $height)** - Sets the field thumbnail height. Default is 0.
- **setThumbnailMode(string $mode)** - Sets the field thumbnail mode. Default is crop.

#### Images field

Render a control to upload a multiple image files.

- **name** - Sets the field name. Required.
- **label** - Sets the field label text. Default is null.

```php
<?php

$this->images('name', 'Label');
```

In addition to the default options, images field has the following options:

- **setAllowedExtensions(array $extensions)** - Sets the field allowed extensions.
- **setFullWidth(bool $fullWidth = true)** - Sets the field width mode.
- **setThumbnailWidth(int $width)** - Sets the field thumbnail width. Default is 100.
- **setThumbnailHeight(int $height)** - Sets the field thumbnail height. Default is 100.
- **setThumbnailMode(string $mode)** - Sets the field thumbnail mode. Default is crop.

#### Number field

Renders a text input that takes numbers only.

- **name** - Sets the field name. Required.
- **label** - Sets the field label text. Default is null.

```php
<?php

$this->number('name', 'Label');
```

In addition to the default options, number field has the following options:

- **setMin($min)** - Sets the min attribute.
- **setMax($max)** - Sets the max attribute.
- **setStep($step = 1)** - Sets the max attribute. Defaut is 1.

#### Password field

Renders a password input.

- **name** - Sets the field name. Required.
- **label** - Sets the field label text. Default is null.

```php
<?php

$this->password('name', 'Label');
```

#### Radios field

Renders a list of radio options, where only one item can be selected at a time.

- **name** - Sets the field name. Required.
- **label** - Sets the field label text. Default is null.
- **options** - Sets the field options. Default is empty array.
- **selected** - Sets the field selected option. Default is null.

```php
<?php

$options = [
    1 => 'Foo',
    2 => 'Bar',
    3 => 'Bazz',
];

$this->radios('name', 'Label', $options, 2);
```

In addition to the default options, radios field has the following options:

- **setInline(bool $inline = true)** - Are items aligned vertically or horizontally.

#### Richeditor field

Renders a visual editor for rich formatted text, also known as a WYSIWYG editor.

- **name** - Sets the field name. Required.
- **label** - Sets the field label text. Default is null.

```php
<?php

$this->richeditor('name', 'Label');
```

#### Text field

Renders a text input.

- **name** - Sets the field name. Required.
- **label** - Sets the field label text. Default is null.

```php
<?php

$this->text('name', 'Label');
```

In addition to the default options, text field has the following options:

- **setPlaceholder(string $placeholder)** - Sets the placeholder.

#### Textarea field

Renders a multiline text box.

- **name** - Sets the field name. Required.
- **label** - Sets the field label text. Default is null.

```php
<?php

$this->textarea('name', 'Label');
```

In addition to the default options, textarea field has the following options:

- **setRows(int $rows = 20)** - Set the rows attribute. Default is 20.
- **setCols(int $cols = 50)** - Set the cols attribute. Default is 50.
- **setPlaceholder(string $placeholder)** - Sets the placeholder.

## Form builder

Instantiating `Form` class is done with the `FormBuilder`.

### Create form

Create GET form:

```php
<?php

$form = $builder->get(PostForm::class)->build();
```

Create POST form:

```php
<?php

$form = $builder->post(PostForm::class)->build();
```

Create PUT form:

```php
<?php

$form = $builder->put(PostForm::class)->build();
```

Create PATCH form:

```php
<?php

$form = $builder->patch(PostForm::class)->build();
```

Create DELETE form:

```php
<?php

$form = $builder->delete(PostForm::class)->build();
```

In addition of creating the form, you can send an optional `data` parameter:

```php
<?php

$post = Post::findOrFail(1);

$form = $builder->post(PostForm::class, $post)->build();
```

Now, when you generate a form field, like a text input, the model's value matching the field's name will automatically be set as the field value. So, for example, for a text input named `title`, the post model's `title` attribute would be set as the value. However, if there is an item in the Session flash data matching the input name, that will take precedence over the model's value. So, the priority looks like this:

1. Session Flash Data ([Old Input](https://laravel.com/docs/5.5/requests#old-input))
2. Data From Current [Request](https://laravel.com/docs/5.5/requests) (via `Request::input` method)
3. Explicitly Passed Value
4. Model Attribute Data

This allows you to quickly build forms that not only bind to model values, but easily re-populate if there is a validation error on the server. Data binding is done by implementing LaravelCollective [HTML and Form Builders](https://github.com/LaravelCollective/html).

### Form action

You may also create forms that point to URI, named routes or a controller actions:

```php
<?php

// URI 
$form = $builder->post(PostForm::class)
    ->setAction('/url')
    ->build();

// named route 
$form = $builder->post(PostForm::class)
    ->setAction(['route' => 'route.name'])
    ->build();

// controller action
$form = $builder->post(PostForm::class)
    ->setAction(['action' => 'Controller@method'])
    ->build();
```

### Form files

If your form is going to accept file uploads, add a files option to your form:

```php
<?php

$form = $builder->post(PostForm::class)
    ->setFiles()
    ->build();
```

FER+ form `upload fields` use ajax to send data to the server, so there is no need to accept file uploads in the form opening tag.

### Form additional options

Build method has an optional `options` parameter, where you can send additional data from the controller to the `Form` class.

```php
<?php

$form = $builder->post(PostForm::class)->build([
    'some_parameter' => true 
]);
```

## View

The are a few ways you can render a form in the views.

Render a form:

```html
@extend('layout')

@section('content')
    {!! $form->render() !!}
@endsection
```

Render a fields:

```html
@extend('layout')

@section('content')
    {!! $form->open() !!}
    {!! $form->fields() !!}
    {!! $form->submit() !!}
    {!! $form->close() !!}
@endsection
```

Render an individual field:

```html
@extend('layout')

@section('content')
    {!! $form->open() !!}
        <div class="row">
            <div class="col-md-6">{!! $form->field('title') !!}</div>
            <div class="col-md-6">{!! $form->field('description') !!}</div>
        </div>
        <div class="row">
            {!! $form->submit() !!}
        </div>
    {!! $form->close() !!}
@endsection
```