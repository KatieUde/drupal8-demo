# Composer template for Drupal projects

[![Build Status](https://travis-ci.org/caxy/drupal-project.svg?branch=8.x)](https://travis-ci.org/caxy/drupal-project)

This project template should provide a kickstart for managing your site
dependencies with [Composer](https://getcomposer.org/).

If you want to know how to use it as replacement for
[Drush Make](https://github.com/drush-ops/drush/blob/master/docs/make.md) visit
the [Documentation on drupal.org](https://www.drupal.org/node/2471553).

## Usage

First you need to [install composer](https://getcomposer.org/doc/00-intro.md#installation-linux-unix-osx).

> Note: The instructions below refer to the [global composer installation](https://getcomposer.org/doc/00-intro.md#globally).
You might need to replace `composer` with `php composer.phar` (or similar) 
for your setup.

After that you can create the project:

```
composer create-project caxy/drupal-project:8.x-dev some-dir --stability dev --no-interaction
```

With `composer require ...` you can download new dependencies to your 
installation.

```
cd some-dir
composer require drupal/devel:~1.0
```

The `composer create-project` command passes ownership of all files to the 
project that is created. You should create a new git repository, and commit 
all files not excluded by the .gitignore file.

## What does the template do?

When installing the given `composer.json` some tasks are taken care of:

* Drupal will be installed in the `docroot`-directory.
* Autoloader is implemented to use the generated composer autoloader in `vendor/autoload.php`,
  instead of the one provided by Drupal (`docroot/vendor/autoload.php`).
* Modules (packages of type `drupal-module`) will be placed in `docroot/modules/contrib/`
* Theme (packages of type `drupal-theme`) will be placed in `docroot/themes/contrib/`
* Profiles (packages of type `drupal-profile`) will be placed in `docroot/profiles/contrib/`
* Creates default writable versions of `settings.php` and `services.yml`.
* Creates `sites/default/files`-directory.
* Latest version of drush is installed locally for use at `vendor/bin/drush`.
* Latest version of DrupalConsole is installed locally for use at `vendor/bin/drupal`.

## Updating Drupal Core

This project will attempt to keep all of your Drupal Core files up-to-date; the 
project [drupal-composer/drupal-scaffold](https://github.com/drupal-composer/drupal-scaffold) 
is used to ensure that your scaffold files are updated every time drupal/core is 
updated. If you customize any of the "scaffolding" files (commonly .htaccess), 
you may need to merge conflicts if any of your modfied files are updated in a 
new release of Drupal core.

Follow the steps below to update your core files.

1. Run `composer update drupal/core --with-dependencies` to update Drupal Core and its dependencies.
1. Run `git diff` to determine if any of the scaffolding files have changed. 
   Review the files for any changes and restore any customizations to 
  `.htaccess` or `robots.txt`.
1. Commit everything all together in a single commit, so `docroot` will remain in
   sync with the `core` when checking out branches or running `git bisect`.
1. In the event that there are non-trivial conflicts in step 2, you may wish 
   to perform these steps on a branch, and use `git merge` to combine the 
   updated core files with your customized files. This facilitates the use 
   of a [three-way merge tool such as kdiff3](http://www.gitshah.com/2010/12/how-to-setup-kdiff-as-diff-tool-for-git.html). This setup is not necessary if your changes are simple; 
   keeping all of your modifications at the beginning or end of the file is a 
   good strategy to keep merges easy.

## Generate composer.json from existing project

With using [the "Composer Generate" drush extension](https://www.drupal.org/project/composer_generate)
you can now generate a basic `composer.json` file from an existing project. Note
that the generated `composer.json` might differ from this project's file.

## Creating a site profile

We advise that all projects be created as a [Drupal 8 profile](https://www.drupal.org/node/2210443) and that
Composer be used to manage Drupal dependencies within the profile.

```bash
mkdir docroot/profiles/specialproject
```

Create a `composer.json` file in the profile's directory, for example:

```json
{
    "name": "my-company/special-project-profile",
    "type": "drupal-profile",
    "repositories": [
        {
            "type": "composer",
            "url": "https://packagist.drupal-composer.org"
        }
    ],
    "require": {
        "drupal/metatag": "~8.0@dev"
    }
}
```

Add this to your root level `composer.json`:

```json
{
    "require": {
        "wikimedia/composer-merge-plugin": "^1.3.0"
    },
    "extra": {
        "merge-plugin": {
            "include": [
                "docroot/profiles/specialproject/composer.json"
            ]
        }
    }
}
```

This will allow you to maintain a `composer.json` file for your profile separate
from the Drupal composer platform's dependencies while keeping the simplicity of
running `composer update` from the root level to update dependencies for the
Drupal platform and the profile at the same time.

## FAQ

### Should I commit the contrib modules I download?

Composer recommends **no**. They provide [argumentation against but also 
workrounds if a project decides to do it anyway](https://getcomposer.org/doc/faqs/should-i-commit-the-dependencies-in-my-vendor-directory.md).

### Should I commit the scaffolding files?

The [drupal-scaffold](https://github.com/drupal-composer/drupal-scaffold) plugin can download the scaffold files (like
index.php, update.php, …) to the web/ directory of your project. If you have not customized those files you could choose
to not check them into your version control system (e.g. git). If that is the case for your project it might be
convenient to automatically run the drupal-scaffold plugin after every install or update of your project. You can
achieve that by registering `@drupal-scaffold` as post-install and post-update command in your composer.json:

```json
"scripts": {
    "drupal-scaffold": "DrupalComposer\\DrupalScaffold\\Plugin::scaffold",
    "post-install-cmd": [
        "@drupal-scaffold",
        "..."
    ],
    "post-update-cmd": [
        "@drupal-scaffold",
        "..."
    ]
},
```
### How can I apply patches to downloaded modules?

If you need to apply patches (depending on the project being modified, a pull 
request is often a better solution), you can do so with the 
[composer-patches](https://github.com/cweagans/composer-patches) plugin.

To add a patch to drupal module foobar insert the patches section in the extra 
section of composer.json:
```json
"extra": {
    "patches": {
        "drupal/foobar": {
            "Patch description": "URL to patch"
        }
    }
}
```

### How do I use this on Acquia cloud?

Add this to your `settings.php` before deploying to Acquia Cloud. Replace `AH_SITE_GROUP` with the name of your site
group in Acquia Cloud.

```php
<?php

// On Acquia Cloud, this include file configures Drupal to use the correct
// database in each site environment (Dev, Stage, or Prod). To use this
// settings.php for development on your local workstation, set $db_url
// (Drupal 5 or 6) or $databases (Drupal 7 or 8) as described in comments above.
if (file_exists('/var/www/site-php')) {

  // As of 29 October 2015, Acquia Cloud does not support release candidates of Drupal 8,
  // so we must define `conf_path` function which was removed between beta15 and RC1.
  if (!function_exists('conf_path')) {
    function conf_path() {
      $request = \Symfony\Component\HttpFoundation\Request::createFromGlobals();
      return \Drupal\Core\DrupalKernel::findSitePath($request, FALSE);
    }
  }
  require('/var/www/site-php/AH_SITE_GROUP/AH_SITE_GROUP-settings.inc');
}
```

The Acquia Connector should be added to the root level `composer.json` unless your site profile
can only run on the Acquia Cloud environment.

```json
{
    "require": {
        "drupal/acquia_connector": "8.1.*@dev"
    }
}
```

### Do I need [Composer Manager](https://www.drupal.org/project/composer_manager)?

You never need this module, and it probably will not work correctly as there is no longer
a composer.json file within the document root. It is incorrect for contrib modules to
declare this dependency explicitly, because it is never the only way to run a module
that has composer PHP dependencies.

In a module or your project profile, add this hook implementation:

```php
<?php

/**
 * Implements hook_system_info_alter().
 */
function MODULE_system_info_alter(array &$info, \Drupal\Core\Extension\Extension $file, $type) {
  // remove composer_manager dependency.
  if (isset($info['dependencies']) && !empty($info['dependencies'])) {
    $info['dependencies'] = array_diff($info['dependencies'], array('composer_manager'));
  }
}
```
