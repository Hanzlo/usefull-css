# Scaffold

This FER+ admin package provides a variety of generators to speed up your development process. Rather than opening the models or controllers directory, creating a new file, saving it, and adding the class, you can simply run a single command.

- [fer:make:migration](#migration)
- [fer:make:model](#model)
- [fer:make:route](#route)
- [fer:make:menu](#menu)
- [fer:make:datatable](#datatable)
- [fer:make:form](#form)
- [fer:make:request](#request)
- [fer:make:controller](#controller)
- [fer:make:views](#views)
- [fer:make:langs](#langs)
- [fer:make:permissions](#permissions)
- [fer:make:resource](#resource)

## Migration

Laravel offers a migration generator, but it stops just short of creating the schema (the fields for the table).

### Usage

```bash
php artisan fer:make:migration posts
```

This will generate the following boilerplate at `database/migrations/YYYY_MM_DD_xxxxxx_create_posts_table.php`:

```php
<?php

use Illuminate\Support\Facades\Schema;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class CreatePostsTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('posts', function (Blueprint $table) {
            $table->increments('id');
            // add schema here...
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::dropIfExists('posts');
    }
}
```

### Options

- -s, --schema[=SCHEMA]              Optional schema to be attached to the migration
- -t, --translatable[=TRANSLATABLE]  Optional translatable schema that creates new translatable migration
- -pk, --pk[=PK]                     The name of the primary key [default: "id"]
- -ts, --timestamps                  Optional timesteps to be attached to the migration

To declare database schema, use a comma+space-separated list of key:value:option sets, where `key` is the name of the field, `value` is the [column type](https://laravel.com/docs/5.5/migrations#creating-columns), and `option` is a way to specify column "modifiers", like `unique` or `nullable`. For the `value` and `options` sets you can also use parameters in the brackets.

Here are some examples:

- `--schema="first:string, last:string"`
- `--schema="age:integer, yob:date"`
- `--schema="username:string:unique, age:integer:nullable"`
- `--schema="name:string:default('John Doe'), bio:text:nullable"`
- `--schema="username:string(30):unique, age:integer:nullable:default(18)"`
- `--schema="user_id:integer:unsigned:foreign"`

## Model

Creates a new model class.

### Usage

```bash
php artisan fer:make:model PostModel
```

This will generate the model at `app/Models/PostModel.php`:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class PostModel extends Model
{
    /**
     * The attributes that are mass assignable.
     *
     * @var array
     */
    protected $fillable = [
        //
    ];
}
```

### Options

- -t, --translatable[=TRANSLATABLE]  Optional translatable fields to be attached to the model and creates new translatable model
- -fi, --fillable[=FILLABLE]         Optional fillable fields to be attached to the model

To declare model fillable or translatable, use a comma+space-separated list of key sets, where `key` is the name of the field. Translatable option will create additional translatable model.

Here are some examples:

- `--fillable="title, description"`
- `--fillable="title, description, published_at" --translatable="title, description"`

## Route

Creates laravel resource routing that assigns the typical "CRUD" routes to a controller.

### Usage

```bash
php artisan fer:make:route posts
```

This will add the routes with the name and the `can` middleware for handling the permissions to the `routes/admin.php` file.

```php
<?php

$router->group([
    'prefix' => 'posts',
    'as' => 'posts.'
], function ($router) {
    $router->get('', 'PostController@index')->name('index')->middleware('can:admin.posts.view');
    $router->get('create', 'PostController@create')->name('create')->middleware('can:admin.posts.create');
    $router->post('', 'PostController@store')->name('store')->middleware('can:admin.posts.create');
    $router->get('{post}/edit', 'PostController@edit')->name('edit')->middleware('can:admin.posts.update');
    $router->put('{post}', 'PostController@update')->name('update')->middleware('can:admin.posts.update');
    $router->delete('{post}', 'PostController@destroy')->name('destroy')->middleware('can:admin.posts.delete');
});
```

### Options

- -s, --simple          Option for creating simple route

With the `--simple` option, you can create routes for the simple "CRUD" screens.

```php
<?php
$router->group([
    'prefix' => 'posts',
    'as' => 'posts.'
], function ($router) {
    $router->get('', 'PostController@index')->name('index')->middleware('can:posts.view');
    $router->post('', 'PostController@store')->name('store')->middleware('can:posts.create');
    $router->get('{post}/edit', 'PostController@edit')->name('edit')->middleware('can:posts.update');
    $router->put('{post}', 'PostController@update')->name('update')->middleware('can:posts.update');
    $router->delete('{post}', 'PostController@destroy')->name('destroy')->middleware('can:posts.delete');
    $router->get('data', 'PostController@data')->name('data')->middleware('can:posts.view');
});
```

## Menu

Creates a new menu class, and assigns the menu item with the resource index route.

### Usage

```bash
php artisan fer:make:menu PostMenu
```

This will generate menu with the title, route, active url, icon and permission at the `app/Menu/PostMenu.php`.

```php
<?php

namespace App\Menus;

use Fer\Contracts\Menu\Item;
use Fer\Contracts\Menu\Menu;
use Fer\Contracts\Menu\MenuExtender as MenuExtenderInterface;

class PostMenu implements MenuExtenderInterface
{
    /**
     * {@inheritdoc}
     */
    public function extend(Menu $menu)
    {
        $menu->add(__('admin/post.posts'), ['route' => 'admin.posts.index'])
            ->setActive('posts/*')
            ->setIcon('list')
            ->setPermission('admin.posts.view');
    }
}
```

After the menu is generated, you need to register it at `app/Http/Middleware/MenuMiddleware.php`:

```php
...
    /**
     * The application mappings.
     *
     * @var array
     */
    protected $application = [
        \App\Menus\PostMenu::class,
    ];
...
```

## Datatable

Creates a new datatable class.

### Usage

```bash
php artisan fer:make:datatable PostDataTable
```

This will generate the datatable at `app/DataTables/PostDataTable.php`:

```php
<?php

namespace App\DataTables;

use Gate;
use Fer\Admin\DataTables\DataTable;

class PostDataTable extends DataTable
{
    /**
     * {@inheritdoc}
     */
    public function init()
    {
        $this->setDefaultSort('updated_at');
        
        // add columns here...

        // add actions here...
    }
}
```

### Options

- -c, --columns[=COLUMNS]  Optional columns to be attached to the datatable
- -l, --titles             Add titles to the columns
- -s, --sort[=SORT]        Default datatable sort column [default: "updated_at"]
- -a, --actions            Datatable actions column

To declare datatables columns, use a comma+space-separated list of key:value sets, where `key` is the name of the column, `value` is the [column attribute](/datatables?id=columns). For the `value` sets you can also use parameters in the brackets.

Here are some examples:

- `--columns="title, description" --titles --actions`
- `--columns="title, description:setSearchable(false):setOrderable(false)"`

## Form

Creates a new form class.

### Usage

```bash
php artisan fer:make:form PostForm
```

This will generate the form at `app/Forms/PostForm.php`:

```php
<?php

namespace App\Forms;

use Fer\Admin\Form\Form;

class PostForm extends Form
{
    /**
     * {@inheritdoc}
     */
    public function init(array $options = [])
    {
        // add form fields here...
    }
}
```

### Options

- -l, --labels            Add labels to the fields
- -c, --cancel            Show cancel button
- -fi, --fields[=FIELDS]  Optional fields to be attached to the form

To declare form fields, use a comma+space-separated list of key:value:option sets, where `key` is the name of the field, `value` is the [field type](/forms?id=custom-fields), and `option` is a way to specify field options. For the `value` and `options` sets you can also use parameters in the brackets.

Here are some examples:

- `--fields="title:text, description:textarea" --labels`
- `--fields="title:text:setRequired:setPlaceholder('Enter the title...'), description:textarea"`

## Request

Creates a new request class.

### Usage

```bash
php artisan fer:make:request PostRequest
```

This will generate the request at `app/Http/Requests/PostRequest.php`:

```php
<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class PostRequest extends FormRequest
{
    /**
     * Determine if the user is authorized to make this request.
     *
     * @return bool
     */
    public function authorize()
    {
        return true;
    }

    /**
     * Get the validation rules that apply to the request.
     *
     * @return array
     */
    public function rules()
    {
        return [
            // add rules here...      
        ];
    }

    /**
     * Get custom attributes for validator errors.
     *
     * @return array
     */
    public function attributes()
    {
        return [
            // add attributes here...  
        ];
    }
}
```

### Options

- -r, --rules[=RULES]                Optional rules to be attached to the request
- -t, --translatable[=TRANSLATABLE]  Optional translatable rules to be attached to the request

To declare request rules or translatable rules, use a comma+space-separated list of key:value:option sets, where `key` is the name of the field, `value` is the [validation rule](https://laravel.com/docs/5.5/validation#available-validation-rules). For the `value` sets you can also use parameters in the brackets.

Both options will also add the list of the attribute translations.

Here are some examples:

- `--rules="title:required, description:required"`
- `--rules="title:required, description:required:between(50,500)"`

## Controller

Creates a new controller class.

### Usage

```bash
php artisan fer:make:controller PostController
```

This will generate the controller at `app/Http/Controllers/Admin/PostController.php`:

```php
<?php

namespace App\Http\Controllers\Admin;

use App\DataTables\PostDataTable;
use App\Forms\PostForm;
use App\Http\Requests\PostRequest;
use App\Models\Post;
use Fer\Contracts\DataTables\DataTableBuilder;
use Fer\Contracts\Form\FormBuilder;
use Fer\Admin\Http\Controllers\Controller;
use Illuminate\Http\Request;
use Illuminate\Http\Response;

class PostController extends Controller
{
    /**
     * Display a listing of the resource.
     * 
     * @param \Fer\Contracts\DataTables\DataTableBuilder $builder
     *
     * @return \Illuminate\Http\Response
     */
    public function index(DataTableBuilder $builder)
    {
        $posts = Post::query()->select('posts.*');
        
        return $builder->create(PostDataTable::class)
            ->of($posts)
            ->build('admin.posts.index');
    }

    /**
     * Show the form for creating a new resource.
     * 
     * @param \Fer\Contracts\Form\FormBuilder $builder
     * 
     * @return \Illuminate\Http\Response
     */
    public function create(FormBuilder $builder)
    {
        $form = $builder->post(PostForm::class, new Post)
            ->setAction(route('admin.posts.store'))
            ->build();

        return view('admin.posts.create', compact('form'));
    }

    /**
     * Store a newly created resource in storage.
     *
     * @param \App\Http\Requests\PostRequest $request
     * 
     * @return \Illuminate\Http\Response
     */
    public function store(PostRequest $request)
    {
        $post = Post::create($request->all());

        return redirect()
            ->to(route('admin.posts.edit', $post))
            ->with('success', __('admin::crud.save_success'));
    }

    /**
     * Show the form for editing the specified resource.
     *
     * @param \App\Models\Post $post
     * @param \Fer\Contracts\Form\FormBuilder $builder
     * 
     * @return \Illuminate\Http\Response
     */
    public function edit(Post $post, FormBuilder $builder)
    {
        $form = $builder->put(PostForm::class, $post)
            ->setAction(route('admin.posts.update', $post))
            ->build();

        return view('admin.posts.edit', compact('form', 'post'));
    }

    /**
     * Update the specified resource in storage.
     *
     * @param \App\Models\Post $post
     * @param \App\Http\Requests\PostRequest  $request
     * 
     * @return \Illuminate\Http\Response
     */
    public function update(Post $post, PostRequest $request)
    {
        $post->update($request->all());

        return redirect()
            ->to(route('admin.posts.edit', $post))
            ->with('success', __('admin::crud.save_success'));
    }

    /**
     * Remove the specified resource from storage.
     *
     * @param \App\Models\Post $post
     * 
     * @return \Illuminate\Http\Response
     */
    public function destroy(Post $post)
    {
        $post->delete();
        
        return redirect()
            ->route('admin.posts.index')
            ->with('success', __('admin::crud.delete_success'));
    }
}
```

### Options

- -s, --simple          Option for creating simple controller

With the `--simple` option, you can create controller for the simple "CRUD" screens.

```php
<?php

namespace App\Http\Controllers\Admin;

use App\DataTables\PostDataTable;
use App\Forms\PostForm;
use App\Http\Requests\PostRequest;
use App\Models\Post;
use Fer\Admin\Http\Controllers\Controller;
use Fer\Contracts\DataTables\DataTableBuilder;
use Fer\Contracts\Form\FormBuilder;
use Illuminate\Http\Request;
use Illuminate\Http\Response;

class PostController extends Controller
{
    /**
     * Display a listing of the resource, and show the form for creating a new resource.
     * 
     * @param \Fer\Contracts\DataTables\DataTableBuilder $dataTableBuilder
     * @param \Fer\Contracts\Form\FormBuilder $formBuilder
     * 
     * @return \Illuminate\Http\Response
     */
    public function index(DataTableBuilder $dataTableBuilder, FormBuilder $formBuilder)
    {
        $html = $dataTableBuilder->create(PostDataTable::class)
            ->setUrl(route('admin.posts.data'))
            ->html();

        $form = $formBuilder->post(PostForm::class, new Post)
            ->setAction(route('admin.posts.store'))
            ->build();

        return view('admin.posts.index', compact('html', 'form'));
    }

    /**
     * Return the DataTable response.
     * 
     * @param \Illuminate\Http\Request $request
     * @param \Fer\Contracts\DataTables\DataTableBuilder $builder
     *
     * @return \Illuminate\Http\JsonResponse
     */
    public function data(Request $request, DataTableBuilder $builder)
    {
        $posts = Post::query()->select('posts.*');

        return $builder->create(PostDataTable::class)
            ->of($posts)
            ->ajax();
    }

    /**
     * Store a newly created resource in storage.
     * 
     * @param \App\Http\Requests\PostRequest $request
     * 
     * @return \Illuminate\Http\Response
     */
    public function store(PostRequest $request)
    {
        $post = Post::create($request->all());

        if ($request->get('action') === 'save_and_new') {
            $url = route('admin.posts.index');
        } else {
            $url = route('admin.posts.edit', $post);
        }

        return redirect()
            ->to($url)
            ->with('success', __('admin::crud.save_success'));
    }

    /**
     * Show the form for editing the specified resource.
     *
     * @param \App\Models\Post $post
     * @param \Fer\Contracts\Form\FormBuilder $formBuilder
     * 
     * @return \Illuminate\Http\Response
     */
    public function edit(Post $post, FormBuilder $formBuilder)
    {
        $form = $formBuilder->put(PostForm::class, $post)
            ->setAction(route('admin.posts.update', $post))
            ->build();

        return view('admin.posts.edit', compact('form', 'post'));
    }

    /**
     * Update the specified resource in storage.
     *
     * @param \App\Models\Post $post
     * @param \App\Http\Requests\PostRequest $request
     * 
     * @return \Illuminate\Http\Response
     */
    public function update(Post $post, PostRequest $request)
    {
        $post->update($request->all());

        if ($request->get('action') === 'save_and_new') {
            $url = route('admin.posts.index');
        } else {
            $url = route('admin.posts.edit', $post);
        }

        return redirect()
            ->to($url)
            ->with('success', __('admin::crud.save_success'));
    }

    /**
     * Remove the specified resource from storage.
     *
     * @param \App\Models\Post $post
     * 
     * @return \Illuminate\Http\Response
     */
    public function destroy(Post $post)
    {
        $post->delete();
        
        return redirect()
            ->route('admin.posts.index')
            ->with('success', __('admin::crud.delete_success'));
    }
}
```

## Views

Creates a new index, create, edit and partial form views.

### Usage

```bash
php artisan fer:make:views posts
```

This will add the views to the `resources/views/admin/posts` folder.

```html
<!-- resources/views/admin/posts/index.blade.php -->

@extends('admin::layouts.plain')

@section('breadcrumb')
    <li class="breadcrumb-item active">
        {{ __('admin/post.posts') }}
    </li>
@endsection

@section('content')

    @component('admin::components.container')
        @component('admin::components.datatable')
            @slot ('header')
                @can ('admin.posts.create')
                    <a class="btn btn-primary btn-cons" href="{{ route('admin.posts.create') }}">
                        <span class="fa fa-plus"></span>
                        {{__('admin::crud.create')}}
                    </a>
                @endcan
            @endslot
            @slot ('filters')
                {!! $html->search() !!}
            @endslot
            {!! $html->table() !!}
        @endcomponent
    @endcomponent

@endsection

@push('scripts')
    {!! $html->scripts() !!}
@endpush
```

```html
<!-- resources/views/admin/posts/create.blade.php -->

@extends('admin::layouts.plain')

@section('breadcrumb')
    <li class="breadcrumb-item">
        <a href="{{ route('admin.posts.index') }}">
            {{ __('admin/post.posts') }}
        </a>
    </li>
    <li class="breadcrumb-item active">
        {{ __('admin/post.form.create') }}
    </li>
@endsection

@section('content')
    @component('admin::components.container')
        @include('admin.posts._form')
    @endcomponent
@endsection
```

```html
<!-- resources/views/admin/posts/edit.blade.php -->

@extends('admin::layouts.plain')

@section('breadcrumb')
    <li class="breadcrumb-item">
        <a href="{{ route('admin.posts.index') }}">
            {{ __('admin/post.posts') }}
        </a>
    </li>
    <li class="breadcrumb-item active">
        {{ __('admin/post.form.edit') }} #{{$post->id}}
    </li>
@endsection

@section('content')
    @component('admin::components.container')
        @include('admin.posts._form')
    @endcomponent
@endsection
```

```html
<!-- resources/views/admin/posts/_form.blade.php -->

{!! $form->open() !!}
    @component('admin::components.card')
        {!! $form->inputs() !!}
    @endcomponent
    @component('admin::components.card', ['class' => 'card-default sticky'])
        {!! $form->submit() !!}
    @endcomponent
{!! $form->close() !!}
```

Every view can be also generated separately:

Index view:
```bash
php artisan fer:make:views:index posts
```

Create view:
```bash
php artisan fer:make:views:create posts
```

Edit view:
```bash
php artisan fer:make:views:edit posts
```

Form view:
```bash
php artisan fer:make:views:form posts
```

### Options

- -s, --simple          Option for creating simple views

With the `--simple` option, you can create views for the simple "CRUD" screens.

## Langs

Creates a new lang files.

### Usage

```bash
php artisan fer:make:langs post
```

This will add the lang files to the `resources/lang/{lang}/admin/post` folders.

```php
<?php

return [
    'post' => 'Post',
    'posts' => 'Posts',

    'fields' => [
        // add fields here...
    ],

    'form' => [
        'create' => 'Create new post',
        'edit' => 'Edit post',
    ],
];
```

Every lang file can be also generated separately:

English lang file:
```bash
php artisan fer:make:lang posts --language="en"
```

### Options

- -s, --fields[=FIELDS]  Optional fields to be attached to the lang files

To declare fields, use a comma+space-separated list of key sets, where `key` is the name of the field.

Here are some examples:

- `--fields="title, description, published_at"`

## Permissions

Creates a new Permissions class.

### Usage

```bash
php artisan fer:make:permissions PostPermissions
```

This will generate the permissions class at `app/Auth/PostPermissions.php`:

```php
<?php

namespace App\Auth;

use Fer\Contracts\Auth\Permissions;
use Fer\Contracts\Auth\PermissionsExtender as PermissionsExtenderInterface;

class PostPermissions implements PermissionsExtenderInterface
{
    /**
     * {@inheritdoc}
     */
    public function extend(Permissions $permissions)
    {
        $permissions->group(__('admin/post.posts'), function () use ($permissions) {
            $permissions->resource('admin.posts');
        });
    }
}
```

After the permissions class is generated, you need to register it at `app/Providers/PermissionsServiceProvider.php`:

```php
...
    /**
     * The application mappings.
     *
     * @var array
     */
    protected $application = [
        \App\Auth\PostPermissions::class,
    ];
...
```

## Resource