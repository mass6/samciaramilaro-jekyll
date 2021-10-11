---
layout:     post
title:      "Create a Helper Function in Laravel for Simpler Readout of Eloquent Dumps"
subtitle:   "Die and dump arrayable: A helper function for dumping Eloquent models attributes."
date:       2021-09-15 12:00:00
header-style: text
author:     "Sam"
catalog: true
tags:
  - laravel
  - helpers
  - tricks
---

Laravel's <a href="https://laravel.com/docs/8.x/collections#method-dd" target="_blank" rel="noopener noreferrer">`dd()`</a> "die and dump" function is so simple, yet incredibly useful. I can't count the number of times I've used `dd('here')` when debugging. It's also great for dumping models and inspecting them in the frontend or console. In this post, we'll look at way to extend this feature for simpler output of Eloquent models.

## `dd()` as it's normally used
If we have a `$user` variable representing an Eloquent model, then `dd($user)` will give an output such as the below.

```php
App\Models\User {#1020 ▼
  #fillable: array:3 [▶]
  #hidden: array:2 [▶]  
  #casts: array:1 [▶]
  #connection: "mysql"
  #table: "users"
  #primaryKey: "id"
  #keyType: "int"
  +incrementing: true
  #with: []
  #withCount: []
  +preventsLazyLoading: false
  #perPage: 15
  +exists: true
  +wasRecentlyCreated: false
  #attributes: array:8 [▶]
  #original: array:8 [▶]
  #changes: []
  #classCastCache: []
  #dates: []
  #dateFormat: null
  #appends: []
  #dispatchesEvents: []
  #observables: []
  #relations: []
  #touches: []
  +timestamps: true
  #visible: []
  #guarded: array:1 [▶]
  #rememberTokenName: "remember_token"
}
```


This is great if you want to inspect the entire Eloquent object. But, sometimes you just want to see the attibute values, not the entire object iteself. In these cases, we can simply use `dd($user->toArray())`, which will only dump the model's attributes.

```php
array:6 [▼
  "id" => 1
  "name" => "Jarret Weber"
  "email" => "hkohler@example.net"
  "email_verified_at" => "2021-10-08T10:58:19.000000Z"
  "created_at" => "2021-10-08T10:58:19.000000Z"
  "updated_at" => "2021-10-08T10:58:19.000000Z"
]
```

But, instead of having to type `dd($user->toArray())` every time, you can create a helper function that does that for you. 

## `dda()` - die and dump arrayable

To create this helper function, start by adding this helper file that will contain 
the new `dda` function. If you already have a customer helper file that's auto-loaded, 
you can place it there instead.

```php
// app/Helpers.php
<?php

use Illuminate\Contracts\Support\Arrayable;
use Symfony\Component\VarDumper\VarDumper;

if (!function_exists('dda')) {
    // If the variable is `arrayable`, call the `toArray()` method
    // on it when dumping.
    function dda(...$vars)
    {
        foreach ($vars as $v) {
            $dump = $v instanceof Arrayable ? $v->toArray() : $v;
            VarDumper::dump($dump);
        }

        exit(1);
    }
}
```

Then, to ensure this file gets auto-loaded, add it to you composer.json file in the `autoload-dev` section.

```json
"autoload-dev": {
    "files": [
        "app/Helpers.php"
    ]
}
```

Afterwards, run:
```sh
composer dump-autoload
```

That's it. You can now use `dda($user)`, and it will work the same as calling 
the `toArray()` method on the $user object. What's more, you can still pass other 
variables to this function, and it will function just like the normal `dd()` method.
