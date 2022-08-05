## Laravel Shopify Plugin

[![Latest Stable Version](https://poser.pugx.org/robwittman/laravel-shopify-plugin/v/stable)](https://packagist.org/packages/robwittman/laravel-shopify-plugin)
[![Total Downloads](https://poser.pugx.org/robwittman/laravel-shopify-plugin/downloads)](https://packagist.org/packages/robwittman/laravel-shopify-plugin)
[![License](https://poser.pugx.org/robwittman/laravel-shopify-plugin/license)](https://packagist.org/packages/OK10101/laravel-shopify-plugin)

This library integrates the Shopify API with your Laravel application.

### Installation

```
composer require robwittman/laravel-shopify-plugin
php artisan vendor:publish
```

### Usage

#### Authentication

By default, the package exposes `/shopify/install` for app installation, and `shopify/uninstall` to listen for the app uninstalled webhook.

#### Configuration

The config file for this library is located at `/config/shopify.php`. This must be filled out using the details from your app in the Shopify Partners dashboard.

##### api_key / api_secret

The credential Shopify gives your application for access

##### embedded

Wether or not your app is embedded. Embedded apps load inside Shopify's admin panel

##### force_redirect

If you initialize the embedded app, and are not in Shopify's admin panel IFrame, should
the app automatically redirect

##### redirect_uri

The URL Shopify should send stores to after they authenticate your application

##### scopes

The scopes we want to ask the store owner for

##### webhooks

An array of webhooks we should automatically install. If the array key is just the webhook, we automatically register `domain.com/shopify/<topic>` as the webhook destination. If the array is associative, we'll use the specified route. If you supply an array of routes, we'll install each one. **NOTE** Relative URLs will use the current app domain. Also, we recommend at least the `app/uninstalled` webhook being set, so the app is notified when someone uninstalls

```php
<?php
# domain = 'https://app.com';
'webhooks' => array(
    'shop/update',
    'app/uninstalled' => 'my/custom/webhook',
    'products/create' => array(
        'endpoint1',
        'https://custom-api.com/products/create'
    )
);
?>
```
This will install the following.

| Topic | Webhook Destination |
| --- | --- |
| `shop/update` | `https://app.com/shopify/shop/update` |
| `app/uninstalled` | `https://app.com/my/custom/webhook` |
| `products/create` | `https://app.com/endpoint1` |
| `products/create` | `https://custom-api.com/products/create` |

##### script_tags

Script tags are URLs that Shopify should automatically load when an installed stores storefront is opened. These run on the customer facing store. **NOTE** Relative URLs will
use the current app domain.
```php
<?php
#domain => 'https://app.com';
'script_tags' => array(
    '/js/script.js',                # installs https://app.com/js/script.js
    'https://cdn.com/js/script2.js' # installs https://cdn.com/js/script2.js'
);
?>
```

#### Events

##### `ShopInstalled`

Triggered anytime a new store authenticates with your application. This plugin includes 2 listeners that install required webhooks and script tags. You can also use this to send E-mails, persist the store to database, start tenant requirements, etc.

To register the bundled webhooks, you can start with this
###### EventServiceProvider.php
```php
<?php

protected $listen = [
    'LaravelShopifyPlugin\Events\ShopInstalled' => [
        'LaravelShopifyPlugin\Listeners\InstallWebhooks',
        'LaravelShopifyPlugin\Listeners\InstallScriptTags'
    ],
];
```

##### `AppUninstalled`

Triggered anytime a store uninstalls your app. Use it to do any last cleanup before saying goodbye.
