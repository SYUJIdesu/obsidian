---
title: "Laravel VS Code Extension Public Beta"
source: "https://laravel-news.com/laravel-vs-code-extension-public-beta"
author:
  - "[[Laravel News]]"
published: 2024-12-16
created: 2025-05-15
description: "The long awaited public beta of the new Laravel VS Code Extension is finally here."
tags:
  - "web"
  - "Laravel"
---
VS Code has firmly established itself as the go-to editor for developers worldwide. While PHPStorm seems to be the preferred choice for Laravel developers, many, especially those transitioning from other languages, are comfortable and would prefer to do Laravel development in VS Code.

There is some Laravel support in VS Code through various extensions, but a truly seamless and integrated experience was missing. Recognizing this need, at [Laracon US 2024 the Laravel core team announced](https://laravel-news.com/laracon-us-keynote-2024) the ambitious task of creating an official VS Code extension for Laravel. The wait is finally over, as the public beta is now available.

The new Laravel VS Code extension is set to enhance your development workflow by providing a suite of helpful features. It offers improved autocomplete, enhanced navigation with click-to-source functionality, and other tools to streamline your Laravel development experience. It can:

- Auto-complete - for app bindings, config and environment variables, routes, models, views, and more.
- Link directly to definitions for routes, models, and config values.
- Warn if something's missing, like a config value or a blade template, and sometimes suggest fixes.
- Provide more helpful information when you hover over the code.

![](https://picperf.io/https://laravelnews.s3.amazonaws.com/images/GjRNxvRaq0SywqAyxJJyEUkCsAIZlEVAIVnTTpts.gif)

## More Features

Here is a non-exhaustive list of features covered in the extension so far:

### App Bindings

```php
app('auth')
App::make('auth.driver')
app()->make('auth.driver')
App::bound('auth.driver')
App::isShared('auth.driver')
// etc
```
- Auto-completion
- Links directly to binding
- Warns when binding not found
- Hoverable

### Assets

```php
asset('my-amazing-jpeg.png')
```
- Auto-completion
- Links directly to asset
- Warns when asset not found
- Blade
- Syntax highlighting

### Config

```php
config('broadcasting.connections.reverb.app_id');
Config::get('broadcasting.connections.reverb.app_id');
Config::getMany([
    'broadcasting.connections.reverb.app_id',
    'broadcasting.connections.reverb.driver',
]);
config()->string('broadcasting.connections.reverb.app_id');
// etc
```
- Auto-completion
- Links directly to config value
- Warns when config not found
- Hoverable

### Eloquent

- Method auto-completion
- Field auto-completion (e.g. where methods, create/make/object creation)
- Relationship auto-completion (e.g. with method + with with array keys)
- Sub-query auto-completion ( with with array keys + value as closure)

### Env

```php
env('REVERB_APP_ID');
Env::get('REVERB_APP_ID');
```
- Auto-completion
- Links directly to env value
- Warns when env not found, offers quick fixes:
- Add to.env
- Copy value from.env.example
- Hoverable

### Inertia

```php
inertia('Pages/Dashboard');
Inertia::render('Pages/Dashboard');
Route::inertia('/dashboard', 'Pages/Dashboard');
```
- Auto-completion
- Links directly to JS view
- Warns when view not found, offers quick fixes:
- Create view
- Hoverable

### Route

```php
route('dashboard');
signedRoute('dashboard');
Redirect::route('dashboard');
Redirect::signedRoute('dashboard');
URL::route('dashboard');
URL::signedRoute('dashboard');
Route::middleware('auth');
redirect()->route('dashboard');
// etc
```
- Auto-completion
- Links directly to route definition
- Warns when route not found
- Hoverable

### Middleware

```php
Route::middleware('auth');
Route::middleware(['auth', 'web']);
Route::withoutMiddleware('auth');
// etc
```
- Auto-completion
- Links directly to middleware handling
- Warns when middleware not found
- Hoverable

### Translation

```php
trans('auth.failed');
__('auth.failed');
Lang::has('auth.failed');
Lang::get('auth.failed');
// etc
```
- Auto-completion
- Links directly to translation
- Warns when translation not found
- Hoverable
- Parameter auto-completion

### Validation

```php
Validator::validate($input, ['name' => 'required']);
request()->validate(['name' => 'required']);
request()->sometimes(['name' => 'required']);
// etc
```
- Auto-completion for strings/arrays (not "|" just yet)

### View

```php
view('dashboard');
Route::view('/', 'home');
```
- Auto-completion
- Links directly to Blade view
- Warns when view not found, offers quick fixes:
- Create view
- Hoverable

## More on the roadmap

Some things that are on the roadmap are:

- Integration with VS Code test runner
- Livewire support
- Volt support
- Pint support
- Better autocompletion, linking, hovering, and diagnostics in Blade files

The extension is currently in Open Beta Testing. To provide feedback or report issues, follow the [support instructions](https://github.com/laravel/vs-code-extension/) provided by the Laravel core team. Your input will help shape the future of this extension and make it the best-in-class Laravel VS Code extension.

Learn more about this extension on the [marketplace](https://marketplace.visualstudio.com/items?itemName=laravel.vscode-laravel) and view the [source code](https://github.com/laravel/vs-code-extension) on Github.

![Yannick Lyn Fatt photo](https://www.gravatar.com/avatar/d2e1d15c9ca048ffff35a1ba26047471?s=200)

[Yannick Lyn Fatt](https://laravel-news.com/@ylynfatt)

Research Assistant at Laravel News and Full stack web developer.

Filed in:

![Cube](https://picperf.io/https://laravel-news.com/images/cube.svg)

## Laravel Jobs

Explore hundreds of open positions today.

[View all jobs](https://larajobs.com/)

## Partners

[View all →](https://laravel-news.com/partners)

![Bacancy logo](https://picperf.io/https://laravelnews.s3.amazonaws.com/partners/Bacancy_logo_400x100.png)

Supercharge your project with a seasoned Laravel developer with 4-6 years of experience for just $2500/month. Get 160 hours of dedicated expertise & a risk-free 15-day trial. Schedule a call now!

![Cut PHP Code Review Time & Bugs into Half with CodeRabbit logo](https://picperf.io/https://laravelnews.s3.amazonaws.com/partners/logos/coderabbit.png)

CodeRabbit is an AI-powered code review tool that specializes in PHP and Laravel, running PHPStan and offering automated PR analysis, security checks, and custom review features while remaining free for open-source projects.

![Join the Mastering Laravel community logo](https://picperf.io/https://laravelnews.s3.amazonaws.com/partners/logos/mastering-laravel-logo.png)

Connect with experienced developers in a friendly, noise-free environment. Get insights, share ideas, and find support for your coding challenges. Join us today and elevate your Laravel skills!

![SaaSykit: Laravel SaaS Starter Kit logo](https://picperf.io/https://laravelnews.s3.amazonaws.com/partners/logos/qMZ85wCJw4QwTO8pF4Mo2UOUz94WgASnQRKCWB3e.png)

SaaSykit is a Multi-tenant Laravel SaaS Starter Kit that comes with all features required to run a modern SaaS. Payments, Beautiful Checkout, Admin Panel, User dashboard, Auth, Ready Components, Stats, Blog, Docs and more.

## The latest

[View all →](https://laravel-news.com/blog)

![Filter Model Attributes with Laravel's New except() Method image](https://picperf.io/https://laravelnews.s3.amazonaws.com/featured-images/15-05-2025-meadow-excerpt-export.png) ![Arr::from() Method in Laravel 12.14 image](https://picperf.io/https://laravelnews.s3.amazonaws.com/featured-images/laravel-12-new.jpg) ![Streamline API Resources with Laravel's Fluent Methods image](https://picperf.io/https://laravelnews.s3.amazonaws.com/featured-images/14-05-2025-midnights-user-resource-export.png) ![Customize URL Handling with Laravel's Macroable URI Class image](https://picperf.io/https://laravelnews.s3.amazonaws.com/featured-images/13-05-2025-raindrop-uri-host-path-exportt.png) ![Use Passkeys in Your Laravel App image](https://picperf.io/https://laravelnews.s3.amazonaws.com/featured-images/laravel-passkeys.png) ![Laravel Seeder Generator image](https://picperf.io/https://laravelnews.s3.amazonaws.com/featured-images/Laravel-Seeder-Generator-LN.png)