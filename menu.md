# Menu

FER+ Menu is simplified api based on [Caffeinated Menus](https://github.com/caffeinated/menus) package that handles creating of dynamic menus for [Laravel framework](http://laravel.com).

- [Quick Start](#quick-start)
- [Menu Items](#menu-items)

## Quick Start

Extend Menu class with a simple artisan command:

```bash
php artisan fer:make:menu PostMenuExtender 
```

MenuExtender class is created in the app/MenuExtenders/PostMenuExtender.php path with the following content:

```php
<?php

namespace App\MenuExtenders;

use Fer\Contracts\Menu\Item;
use Fer\Contracts\Menu\Menu;
use Fer\Contracts\Menu\MenuExtender;

class PostMenuExtender implements MenuExtender
{
    /**
     * {@inheritdoc}
     */
    public function extend(Menu $menu)
    {
        // add menu items here...
    }
}
```

Register newly created menu extender in the `app/Http/Middleware/MenuMiddleware` class.

```php
<?php

...
    /**
     * The application mappings.
     *
     * @var array
     */
    protected $application = [
        \App\MenuExtenders\PostMenuExtender::class,
    ];
...
}
```

## Menu Items

Add menu items with the `add` api:

```php
<?php

$menu->add('Posts', '/posts');
```

When a menu item is added, it'll automatically be assigned both a `slug` and `ID` that it can be referenced by in the future if the need arises. The slug is determined by the title that you assign an item, in camel case form.

You can also add a settings items, which are rendered as the settings subitems at the end of the menu.

```php
<?php

$menu->settings('Users', '/settings/users');
```

**Title**

Assign a title to your menu item as the first parameter when adding your menu item.

**Url**

Assign a URL to your menu item by passing the URI, named route or a controller action as the second parameter when adding your menu item.

```php
<?php

// URI 
$menu->add('Posts', '/url');

// named route 
$menu->add('Posts', ['route' => 'route.name']);

// controller action 
$menu->add('Posts', ['action' => 'Controller@method']);
```

**Slug**

Override the default slug.

```php
<?php

$menu->add('Posts', '/posts')->setSlug('slug');
```

**Visibility**

You can hide the menu item.

```php
<?php

$menu->add('Posts', '/posts')->hide();
```

And show it again.

```php
<?php

$menu->add('Posts', '/posts')->show();
```

**Active Link Detection**

FER+ Menu will automatically detect the currently active link for you, based on the current URL. Once an item is detected as being the active link, a new active class will be attributed to the menu item. 

The automatic link detection is only useful for one-on-one URI detection. It does not take into consideration of any dynamic URI segments, such as blog/post/edit/1. For these cases, you may specify that the active state should be activated against other URI segments.

```php
<?php

$menu->add('Posts', '/posts')->setActive('blog/posts/*');
```

The above menu item will be active on any URI that matches blog/posts/*:

- blog/posts
- blog/posts/create
- blog/posts/edit/1
- etc.

**Order**

You can set menu item order. Defaut value is 100.

```php
<?php

$menu->add('Posts', '/posts')->setOrder(20);
```

**Icons**

FER+ Menu supports [Font Awesome 4.7](https://fontawesome.com/v4.7.0/icons/) icons. There is no need to prepand `fa fa-` prefix.

```php
<?php

$menu->add('Posts', '/posts')->setIcon('font');
```

**Attributes**

Assign attributes to the menu item.

```php
<?php

$menu->add('Posts', '/posts')->setAttribute('name', 'value');
```

**Permissions**

comming...

**Sub-Items**

Menu items can have as many child items as you need.

```php
<?php

$menu->add('Posts', '/posts')->add('Subitem', '/item/subitem');
```