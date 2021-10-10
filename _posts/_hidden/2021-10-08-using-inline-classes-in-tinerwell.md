---
layout:     post
title:      "Using Inline Classes in Tinkerwell"
subtitle:   "The hidden power in Tinkerwell"
date:       2021-10-08 14:40:00
header-style: img
header-img: "img/post-tinkerwell-bg3.png"
author:     "Sam"
catalog: false
tags:
  - tinkerwell
---

Tinkerwell is an amazing tool by [BeyondCode](https://beyondco.de/). Like many who use it, I find it an invaluable tool in my development toolbox. But, soon after I began using Tinkerwell, I discovered a hidden gem. 

### Inline classes

Tinkerwell allows you to create adhoc "inline" classes directly in the editor. For example:


```php
// Tinkerwell Editor
class StringFormatter
{
    public function format(string $string)
    {
        return strtolower($string);
    }
}

$formatter = new StringFormatter();
$formatter->format('TACOS');


// Tinkerwell Output
=> "tacos"
```


Pretty slick. But, not only can you create these classes in Tinkerwell, you can use them directly with your Laravel applications. For example, you can declare a class in Tinkewell, and then instantiate it and pass it in to your Laravel application.


Let's say you have a service class that performs string formatting, which takes an instance of a `FormatterInterface` object in its contructor. Here's how the interface, the service class, and a concrete implementation of the interface might look like. 


```php
// Service class
class StringFormattingService
{
    /** @var \App\Formatting\Formatter */
    private $formatter;

    public function __construct(Formatter $formatter)
    {
        $this->formatter = $formatter;
    }

    public function format(string $string): string
    {
        return $this->formatter->format($string);
    }
}

// Formatter interface.
interface Formatter
{
    public function format(string $string): string;
}

// Concrete implemenation of Formatter interface.
class LowercaseFormatter implements Formatter
{
    public function format(string $string): string
    {
        return strtolower($string);
    }
}

// AppServiceProvider: Bind the interface to the implementation.
public function register()
{
    $this->app->bind(Formatter::class, App\Formatters\LowercaseFormatter::class);
}
```

To see how this functionality works within our application, we can test it Tinkerwell.

```php
// Tinkerwell Editor
$formatterService = app()->make(App\Formatters\StringFormatterService::class);
$formatterService->format('Tacos');

// Tinkerwell Output
=>'tacos'
```

### Manually pass inline class to Service class

Now, we want to tinker with a new implementation of the Formatter interface within Tinkerkwell. We'll create a new class, instantiate the service class manually, and pass in the new formatter into its constructor. 

<!-- Now, in Tinkerwell, we can instantiate a `StringFormattingService` instance by instantiating a `LowercaseFormatter` instance, and passing it into the contructor of the StringFormattingService object. Then, we can call the `format()` with a string, and inspect the output. -->

```php
// Tinkerwell Editor
class UppercaseFormatter implements App\Formatters\Formatter
{
    public function format(string $string): string
    {
        return strtoupper($string);
    }
}

$formatter = new UppercaseFormatter();
$formatterService = new App\Formatters\StringFormattingService($formatter);
$formatterService->format('Tacos');

// Tinkerwell Output
=>'TACOS'
```

Cool, right? We've just created a class outside of our application, and were then able to use it together with code within our application. 


### Binding to the IOC outside

Now for the really interesting stuff. Not only can we manually inject our new class, but we can also override the existing interface binding in the Service Container, replacing it with the new class.


```php
// Tinkerwell Editor
class UppercaseFormatter implements App\Formatters\Formatter
{
    public function format(string $string): string
    {
        return strtoupper($string);
    }
}

// Bind the Formatter interface to the new class.
app()->bind(App\Formatters\Formatter::class, UppercaseFormatter::class);

$formatterService = app(App\Formatters\StringFormatterService::class);
$formatterService->format('Tacos');

// Tinkerwell Output
=>'TACOS'
```

This gives us the ability to interact with code and data on a remote server, without having to alter the codebase running on it. Imagine what possibiities this affords us? Since we could interact with a remote server, and thereby its associated database by virtue of Eloquent, we have the posibility of accessing and transforming that data, **via code**, but without ever having to make a change to the codebase. Likewise, we can do this wihout having to access the database and run raw SQL queries. 

 Sometimes, the data we have locally just doesn't match all the variety that exists in a production system. Let's say we have an Artisan command that generate a report, and this command has a dependency on our StringFormatterService. If we needed to change how the report was formatted, we could easily create an different implemention of the Formatter implemantion class in Tinkerwell that contains the desired changes, connect to the remote server, and bind this new class to the Service Container. We could then run the Artisan command from within Tinkerwell, and the report would now be using the new service. All wihtout ever having to touch the existing codebase. Now, that's pretty powerful!


 I'd like to say that I'm not advocatig for or against running rogue code on a production server as a best practice, but I'm just pointing out that the possibility exists. Best judgment prevails.