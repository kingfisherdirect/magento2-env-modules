# Magento 2 env.php Modules

> Patch prevents Magento from duplicating `env.php` defined modules in `config.php` makint it easier to manage module status per environment

## Installation

> [!CAUTION]
> This is not yet fully tested under every circumstances. It works for me.

### Install patch source files

```sh
composer require kingfisherdirect/magento2-env-modules
```

### Apply patch

Install composer patches plugin, if you don't have it already.

```sh
composer require cweagans/composer-patches
```

> [!NOTE]
> Details about applying patches can be found on [Magento documentation](https://experienceleague.adobe.com/en/docs/commerce-operations/upgrade-guide/patches/apply).

In the `composer.json` under `extra.patches` add following
```
"magento/magento2-base": {
    "Skip config.php modules if defined in env.php": "vendor/kingfisherdirect/magento2-env-modules/magento2_setup_install.patch"
}
```

Now run `composer install` which will do the patching.  
For git users, after successfull installation you should have observe an untracked file in `git status` (`src/setup/src/Magento/Setup/Model/Installer.php`). This file should be commited.

## Problem and solution

Pretty much explained in following discussions:

- https://github.com/magento/magento2/issues/38016
- https://magento.stackexchange.com/questions/369810/using-env-php-to-override-module-status-in-config-php

Magento installation saves module status in `config.php` file that should be tracked in git repository. This file is overwritten every time `bin/magento setup:upgrade` is run. Whether module status changed, new module is installed everything is overwritten in `config.php`.

By default Magento also reads `env.php` file for module statuses. However, because of mentioned overwrites to `config.php`, it's not convinient to use `env.php`.

**This module amends the default magento behaviour by excluding modules from beeing written to `config.php` if they're already in `env.php`.**

> [!IMPORTANT]
> Module status can be still changed in by `module:enable` or `module:disable` which will write to `config.php`.
> In cases when module is defined in both `config.php` and `env.php`, status from latter file will be considered.

## Example usage scenarious

Since composer allows to define `require` and `require-dev` packages, dev tools can be added to `require-dev` and only manually enabled through `env.php`. On production `require-dev` packages doesn't have to be installed, therefore nothing should be in `config.php`.

On the other hand modules may be in `require` making it always installed. Because of modules `registration.php` file, it's automatically loaded and status will (and should) be written to `config.php`. Each environment may then use `env.php` to disable module.

## Why not a PR to magento/magento2

I don't believe this is ready to be shipped in Magento by default.

This patch makes `module:enable` and `module:disable` ineffective for `env.php` defined modules. I needed a solution that works, not neccessary handles every scenario.
