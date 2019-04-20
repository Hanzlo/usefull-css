# DataTables

FER+ DataTables is simplified api with some useful helpers based on [Laravel DataTables](https://yajrabox.com/docs/laravel-datatables/master) package that handles the server-side works of [jQuery DataTables](http://datatables.net) using [Laravel framework](http://laravel.com).

- [Quick Start](#quick-start)
- [Configuration](#configuration)
- [Builder](#builder)
- [View](#view)

## Quick Start

Creating DataTable classes with a simple artisan command:

```bash
php artisan fer:make:datatable PostDataTable --columns="name, updated_at" --actions
```

DataTable class is created in the app/DataTables/PostDataTable.php path with the following content:

```php
<?php

namespace App\DataTables;

use Fer\Admin\DataTables\DataTable;

class PostDataTable extends DataTable
{
    /**
     * {@inheritdoc}
     */
    public function init()
    {
        $this->setDefaultSort('updated_at');
        
        $this->column('name');
        $this->column('updated_at');

        $this->actions(function ($post) {
            return dt_actions(
                route('posts.edit', $post->id),
                route('posts.destroy', $post->id)
            );
        });
    }
}
```

After that instantiate the class in the controller and pass it to the view:

```php
<?php

namespace App\Http\Controllers;

use App\Post;
use App\DataTables\PostDataTable;
use Fer\Contracts\DataTables\DataTableBuilder;

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
        $resources = Post::query()
            ->withoutGlobalScopes()
            ->select('posts.*');
        
        return $builder->create(PostDataTable::class)
            ->of($resources)
            ->build('posts.index');
    }
}
```

And than print the datatable in view:

```html
<!-- resources/views/posts/index.blade.php -->

@extend('layout')

@section('content')
    {!! $html->table() !!}
@endsection

@push('scripts')
    {!! $html->scripts() !!}
@endpush
```

## Configuration

- [ID](#id)
- [Parameters](#parameters)
- [Searching](#searching)
- [Sorting/Ordering](#sortingordering)
- [Columns](#columns)
- [Info](#info)
- [Paging](#paging)
- [Row editing](#row-editing)
- [Ajax](#ajax)

`DataTable` class is the basis of the FER+ DataTables package. It contains all logic that is handled from instantiation to rendering. By default, it should contain `init` method that will be called on the form instantiation through `DataTableBuilder`, and here you can configure request and create columns.

```php
<?php

namespace App\DataTables;

use Fer\Admin\DataTables\DataTable;

class PostDataTable extends DataTable
{
    /**
     * {@inheritdoc}
     */
    public function init()
    {
        // configuration ...
    }
}
```

### ID

You can change a DataTable id on your response by using `setId` api. Default id is `dataTable`, and unless you have more than one DataTable on the view, there is no need to change the id.

```php
<?php

$this->setId('dataTableId');
```

### Parameters

FER+ DataTables package handles only commonly most used options. You can set any missing [options](https://datatables.net/reference/option/) by using the following api.

```php
<?php

$this->setParameters([
    'scrollX' => true,
    'scrollY' => true,
]);

// or set individual parameter.

$this->setParameters('scrollX', true);
$this->setParameters('scrollY', true);
```

### Searching

You can optionally write your own manual filtering functions and thus disabling global search of the package. To achieve this, you can use the `filter` api.

```php
<?php

$this->filter(function ($query) {
    if (request()->has('name')) {
        $query->where('name', 'like', "%" . request('name') . "%");
    }

    if (request()->has('email')) {
        $query->where('email', 'like', "%" . request('email') . "%");
    }
});
```

> To enable global search with filter api, just set the 2nd argument to `true`.

You can optionally enable/disable DataTables smart search feature by using `smart` api by passing `true` or `false` as the argument. Default argument value is `true`.

```php
<?php

$this->smart(false);
```

### Sorting/Ordering

You may disable the default ordering function of the DataTables.

```php
<?php

$this->disableOrdering();
```

To enable it again use the following api.

```php
<?php

$this->enableOrdering();
```

By default DataTables will sort by the first column. To change default sort column and direction use the following api.

```php
<?php

$this->setDefaultSort('updated_at');

// or set descending sort

$this->setDefaultSort('updated_at', 'desc');
```

If you want to override the default ordering function of DataTables and write you own, use `order` api.

```php
<?php

$this->order(function ($query) {
    if (request()->has('name')) {
        $query->orderBy('name', 'asc');
    }

    if (request()->has('email')) {
        $query->orderBy('email', 'desc');
    }
});
```

This api will set DataTables to perform ordering with `NULLS LAST` option.

```php
<?php

$this->orderByNullsLast();
```

### Columns

Column represents both the column to be rendered by your DataTables and optionally you can and or edit a column on your [response](https://yajrabox.com/docs/laravel-datatables/master/response-array).

```php
<?php

$this->column('data', 'Title');
```

A DataTable Column accepts the following attributes:

**Data**

Data attribute will be used when rendering the response to your table. This is the `key` from the  `json` response `data` array.

**Title (Optional)**

Title attribute is used as your `table` column heading.

> If not set, data attribute value will be use as title with title case format.

**Name**

Name attribute represents the `column` name from your data source. DataTables will use this attribute when performing search and ordering functions.

```php
<?php

$this->column('data')->setName('name');
```

> If not set, name attribute will automatically be set to same value as data attribute.

**Searchable**

Searchable attribute will toggle the `searching` ability for the defined column. Default value is `true`.

```php
<?php

$this->column('data')->setSearchable(false);
```

**Filter column**

In some cases, you need to implement a custom search for a specific column. To achieve this, you can use `filterColumn` api.

```php
<?php

$this->column('fullname')->filterColumn(function($query, $keyword) {
    $sql = "CONCAT(users.first_name,'-',users.last_name)  like ?";
    
    $query->whereRaw($sql, ["%" . $keyword . "%"]);
});
```

**Orderable**

Orderable attribute will toggle the `ordering` ability for the defined column. Default value is `true`.

```php
<?php

$this->column('data')->setOrderable(false);
```

**Order column**

In some cases, you may want to use a custom order sql for a specific column. To achieve this, you can use `orderColumn` api.

```php
<?php

$this->column('name')->orderColumn('-name $1');
```

> Order column has a special variable `$1` which is being replace as the order direction value of the request.


**Footer**

Footer attribute will be as your `tables` column's `footer` content.

```php
<?php

$this->column('data')->setFooter(true);
```

> To display the footer using the `builder`, pass `true` as 2nd argument on `$html->table([], true)` api in your view.

**Exportable**

Exportable attribute will flag the column to be included when `exporting` the table. Default value is `true`.

```php
<?php

$this->column('data')->setExportable(false);
```

**Printable**

Printable attribute will flag the column to be included when `printing` the table. Default value is `true`.

```php
<?php

$this->column('data')->setPrintable(false);
```

**Escape**
 
DataTable response is encoded to prevent XSS attack. In case you need to display html on your columns, you can use `setEscape` api. Default value is `false`.

```php
<?php

$this->column('data')->setEscape(true);
```

**Add column to response**

You can add a custom column on your response by using `addColumn` api.

Add Column with Blade Syntax:

```php
<?php

$this->column('name')->addColumn('Hi {{$name}}!');
```

Add Column with Closure:

```php
<?php

$this->column('data')->addColumn(function(User $user) {
    return 'Hi ' . $user->name . '!';
});
```

Add Column with View:

```php
<?php

$this->column('name')->addColumn('users.datatables.intro');
```

Then create your view on `resources/views/users/datatables/intro.blade.php`.

```html
Hi {{ $name }}!
```

Add Column with specific order:

```php
<?php

$this->column('name')->addColumn('Hi {{$name}}!', 2);
```

**Edit column on response**

You can edit a custom column on your response by using `editColumn` api.

Edit Column with Blade Syntax:

```php
<?php

$this->column('name')->editColumn('Hi {{$name}}!');
```

Edit Column with Closure:

```php
<?php

$this->column('data')->editColumn(function(User $user) {
    return 'Hi ' . $user->name . '!';
});
```

Edit Column with View:

```php
<?php

$this->column('name')->editColumn('users.datatables.intro');
```

Then create your view on `resources/views/users/datatables/intro.blade.php`.

```html
Hi {{ $name }}!
```

**Remove column from response**

You can remove a column on your response by using `removeColumn` api.

```php
<?php

$this->removeColumn('name');
```

### Info

You can hide DataTables info.

```php
<?php

$this->hideInfo();
```

To show it again use the following api.

```php
<?php

$this->showInfo();
```

### Paging


To skip paging of DataTables, you can use `disablePaging` api.

```php
<?php

$this->disablePaging();
```

To enable it again use the following api.

```php
<?php

$this->enablePaging();
```

Default page length is 10. To override it use the following api.

```php
<?php

$this->setPageLength(20);
```

In some cases, you need to manually set the total records of our DataTables and skip its internal counting functionality. To achieve this, you can use `setTotalRecords($count)` api.

```php
<?php

$this->setTotalRecords(100);
```

### Row editing

**Row Id**

Setting row id via `column` name.

```php
<?php

$this->setRowId('id');
```

Setting row id via `closure`.

```php
<?php

$this->setRowId(function ($user) {
    return $user->id;
});
```

Setting row id via `blade` string.

```php
<?php

$this->setRowId('{{$id}}');
```

**Row Class**

Setting row class via `closure`.

```php
<?php

$this->setRowClass(function ($user) {
    return $user->id % 2 == 0 ? 'alert-success' : 'alert-warning';
});
```

Setting row class via `blade` string.

```php
<?php

$this->setRowClass('{{ $id % 2 == 0 ? "alert-success" : "alert-warning" }}');
```


**Row Data**

Setting row class via `closure`.

```php
<?php

$this->setRowData([
    'data-id' => function($user) {
    	return 'row-' . $user->id;
    },
    'data-name' => function($user) {
    	return 'row-' . $user->name;
    },
]);
```

Setting row class via `blade` string.

```php
<?php

$this->setRowData([
    'data-id' => 'row-{{$id}}',
    'data-name' => 'row-{{$name}}',
]);
```

**Row Attributes**

Setting row class via `closure`.

```php
<?php

$this->setRowAttr([
    'color' => function($user) {
    	return $user->color;
    },
]);
```

Setting row class via `blade` string.

```php
<?php

$this->setRowAttr([
    'color' => '{{$color}}',
]);
```

### Ajax

Ajax url is an option that you set to tell DataTable where to fetch it's data.

```php
<?php

$this->setAjaxUrl('/url');
```

Script is any vanilla javascript to be included on `data` method of ajax.

```php
<?php

$this->addAjaxScript('data.foo = $("#'.$this->id.'fooBtn").html();');
```

Data is an array of values to be appended on server request.

```php
<?php

$this->addAjaxData('foo', 'bar');
```

You can also set custom javascript template.

```php
<?php

$this->setTemplate('datatables.script');
```

> If published, the file will be installed on `resources/views/vendor/admin/partials/datatables/script.blade.php`.

## Builder

Instantiating `DataTable` class is done with the `DataTableBuilder`.

```php
<?php

$dataTable = $builder->create(PostDataTable::class)
    ->build('posts.index');
```

The `build` method takes view as the first parameter. Optionally, you can send additional data from the controller to the DataTable class.

```php
<?php

$dataTable = $builder->create(PostDataTable::class)
    ->build('posts.index', [
        'some_parameter' => true
    ]);
```

You can also change DataTable `Id` parameter from the builder.

```php
<?php

$dataTable = $builder->create(PostDataTable::class)
    ->setId('posts')
    ->build('posts.index');
```

The build method will automatically detect the type of the request in your controller action, and render DataTable or send a DataTable response accordingly.

But sometimes you will need to separate render from the response (if you have more than one DataTable in the view). You can achive this with `html` and `ajax` api.

```php
<?php

namespace App\Http\Controllers;

use App\Post;
use App\DataTables\PostDataTable;
use Fer\Contracts\DataTables\DataTableBuilder;

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
        $html = $builder->create(PostDataTable::class)
            ->html();

        return view('posts.index', compact('html'));
    }

    /**
     * Process datatables ajax request.
     * 
     * @param \Fer\Contracts\DataTables\DataTableBuilder $builder
     *
     * @return \Illuminate\Http\JsonResponse
     */
    public function data(DataTableBuilder $builder)
    {
        $resources = Post::query()
            ->withoutGlobalScopes()
            ->select('posts.*');
        
        return $builder->create(PostDataTable::class)
            ->of($resources)
            ->ajax();
    }
}
```

Now, you have to tell your DataTable where to fetch the data.

```php
<?php

$this->setAjaxUrl(route('posts.data'));
```

### Data Sources

Laravel's Eloquent Model as data source for your DataTable.

```php
<?php

$model = App\Post::query();

$dataTable = $builder->create(PostDataTable::class)
    ->of($model)
    ->build('posts.index');
```

 Laravel's Query Builder as data source for your DataTable.

 ```php
<?php

$query = DB::table('posts');

$dataTable = $builder->create(PostDataTable::class)
    ->of($query)
    ->build('posts.index');
```

Laravel's Collection as data source for your DataTable.

 ```php
<?php

$collection = collect([
    ['id' => 1, 'name' => 'John'],
    ['id' => 2, 'name' => 'Jane'],
    ['id' => 3, 'name' => 'James'],
]);

$dataTable = $builder->create(PostDataTable::class)
    ->of($collection)
    ->build('posts.index');
```

## View

Render a dataTable in the view:

```html
<!-- resources/views/posts/index.blade.php -->

@extend('layout')

@section('content')
    {!! $html->table() !!}
@endsection

@push('scripts')
    {!! $html->scripts() !!}
@endpush
```