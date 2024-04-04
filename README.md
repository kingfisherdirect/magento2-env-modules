# Magento 2 env.php Modules

> Patch prevents Magento from duplicating `env.php` defined modules in `config.php` makint it easier to manage module status per environment

## Installation

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
    "Skip config.php modules if defined in env.php": "patches/composer/mage_env_modules.diff"
}
```

Now run `composer install` which will do the patching.  
For git users, after successfull installation you should have observe an untracked file in `git status` (`src/setup/src/Magento/Setup/Model/Installer.php`). This file should be commited.

## Problem it solves

Pretty much explained in following discussions:

- https://github.com/magento/magento2/issues/38016
- https://magento.stackexchange.com/questions/369810/using-env-php-to-override-module-status-in-config-php

At the moment Magento installation tracks module status in `config.php` file that's tracked in git repository.

In some cases it's neccessary to enable certain modules in specific environments. (For example you may want debug modules for developers to work with locally, that are not desired on production. I've also heard people disabling 2FA locally for convinience).  

At the moment the most common scenario is that module developers are providing toggles to enable/disable module through magento configuration. This, may work but adds unnecessary overhead of still loading module, config options, running conditions etc.

## Solution details

By default Magento is also reading `env.php` file for module statuses. However, it's far from beeing convinient to use.  

Every module installation requires `bin/magento setup:upgrade` that overrides `config.php` with latest status of all module.

**This module** amends the default magento behaviour by excluding modules from beeing written to `config.php` if they're already in `env.php` during `setup:upgrade`

> [!IMPORTANT]
> Module status can be still changed in by `module:enable` or `module:disable` which will write to `config.php`.
> In cases when module is defined in both `config.php` and `env.php`, status from latter file will be considered.

## Example usage scenarious

Since composer allows to define `require` and `require-dev` packages, dev tools can be added to `require-dev` and only manually enabled through `env.php`. On production `require-dev` packages doesn't have to be installed, therefore nothing should be in `config.php`.

On the other hand modules may be in `require` making it always installed. Because of modules `registration.php` file, it's automatically loaded and status will (and should) be written to `config.php`. Each environment may then use `env.php` to disable module.
