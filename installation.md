# Installation

- [Create project](#create-project)
- [Add package](#add-package)
- [Publish assets](#publish-assets)
- [Middleware](#middleware)
- [User model](#user-model)
- [Permissions](#permissions)
- [Run project](#run-project)

## Create Project

FER + is based on the [Laravel Framework](http://laravel.com), so before you start, you must install it, and also make sure that your computer meets the [system requirements](/requirements).

```bash
composer create-project laravel/laravel project-name "5.5.*" --prefer-dist
```

**Do not forget**
- Set the rights "chmod -R o + w" to the directories `storage` and` bootstrap/cache`
- Edit `.env` file

> If you installed Laravel differently, then you may need to generate a key using the `php artisan key:generate`

## Add package

In the `composer.json` file add the following lines.

```json
...
    "repositories": [{
        "type": "composer",
        "url": "https://packages.fer.plus/"
    }],
    "require": {
        "fer/admin": "0.1.*"
    }
...
```

Go to the project directory and execute the command:

```bash
composer update
```

## Publish assets

Make the styles, javascript and other linkable files available:

```bash
php artisan vendor:publish --tag=admin.public
```

```bash
php artisan storage:link
```

## Middleware

Create a menu middleware in the `app/Http/Middleware` directory.

```php
<?php

namespace App\Http\Middleware;

use Fer\Admin\Http\Middleware\MenuMiddleware as Middleware;

class MenuMiddleware extends Middleware
{
    /**
     * The application mappings.
     *
     * @var array
     */
    protected $application = [
        // register menu extenders here...
    ];
}
```

In the `app/Http/Kernel.php` class create new application's route middleware group called `admin`, and register menu and lang middleware.

```php
<?php

...
    /**
     * The application's route middleware groups.
     *
     * @var array
     */
    protected $middlewareGroups = [
        ...

        'admin' => [
            \Fer\Admin\Http\Middleware\LangMiddleware::class,
            \App\Http\Middleware\MenuMiddleware::class,
        ],
    ];
...
```

## User model

The FER+ comes with a ready set of user functions, for example access rights etc. It is necessary to inherit the base model in `app/User.php` class.

```php
<?php

namespace App;

use Fer\Admin\Models\User as BaseUser;

class User extends BaseUser
{
    
}
```

Or you can register the default user in the `config/auth.php` configuration file.

```php
<?php

...
    'providers' => [
        'users' => [
            'driver' => 'eloquent',
            'model' => Fer\Admin\Models\User::class,
        ],
    ],
...
```

Now you can run migration files:

```bash
php artisan migrate
```

And create a user with the maximum rights by running  the following command:

```bash
php artisan fer:make:admin
```

You will be prompted for the username, email address and password.

## Permissions

Create a permissions service provider in the `app/Providers` directory.

```php
<?php

namespace App\Providers;

use Fer\Admin\Providers\PermissionsServiceProvider as ServiceProvider;

class PermissionsServiceProvider extends ServiceProvider
{
    /**
     * The application mappings.
     *
     * @var array
     */
    protected $application = [
        // register permissions extenders here...
    ];

    /**
     * Register any authentication/authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPermissions();
    }
}
```

Register newly created permissions service provider in the `config/app.php` file.

```php
<?php

...
    'providers' => [
        ...
        /*
         * Application Service Providers...
         */
        App\Providers\PermissionsServiceProvider::class,
        ...
    ],
...
```

## Run project

To start the project, you can use the built-in server:

```bash
php artisan serve
```

Open the browser and go to `http://localhost:8000/admin`. If everything works, you will see the login page.