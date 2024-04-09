This is a project template for Drupal 10 sites built with the Openfed distribution.

## Requirements

- At least PHP 8.1 installed
- Composer 2.0

## File Structure

There are 3 json files:

### composer.json

The main json file for your project, which you can use to require extra repositories.
You can override this at your will, just make sure that **composer-merge-plugin** settings and package are kept in order to use the json files mentioned bellow.

For more info about **composer-merge-plugin** settings and options check https://github.com/wikimedia/composer-merge-plugin

### composer.openfed.json

Includes all Openfed related settings and should not be changed once you create your project. However, you should update this file regularly based on the most recent version in this repository.

### composer.patches.json

Where all the specific patches for the project should be set. This repo doesn't apply patches so this file will always be empty and it's here just to be used as a template or starting point.
See bellow how to apply patches using the command line.

## Usage

### Installation

This project requires the installation of [composer](https://getcomposer.org/doc/00-intro.md#installation-linux-unix-osx).

> Note: The instructions below refer to the [global composer installation](https://getcomposer.org/doc/00-intro.md#globally).
You might need to replace `composer` with `php composer.phar` (or similar)
for your setup.

After that you can create the project:

```
composer create-project openfed/openfed-project:~12.2.0 MYPROJECT
```

### Update

The best is to download the latest version and replace your project with the files. If you have custom modules defined on your composer.json, you need to copy the to the new composer.json file.
You can delete the existing composer.libraries.json as it has been removed from this project.

Since Openfed 8.x-10.0, there's a composer script (Experimental) which you can run in order to have your local project **partially** updated. To update your projects you can:
- Backup your site
- Run <pre>composer run-script project-update</pre>
- Manually update composer.json (it's recommended to use the composer.json from this repo and ajust it to use your projects/patches)
- Run <pre>composer update</pre>

Your project should now be updated.

### Require new modules

With `composer require ...` you can download new dependencies to your
installation.

```
cd MYPROJECT
composer require drupal/devel:~1.0
```

The `composer create-project` command passes ownership of all files to the
project that is created. You should create a new git repository, and commit
all files not excluded by the .gitignore file.

## Using Patches

There's a composer.patches.json file which should be used to define all the needed patches for your project.
It's possible to manage patches using the command line due to szeidler/composer-patches-cli library. See below some commands and check https://github.com/szeidler/composer-patches-cli for more info.

### Patch Add
```
composer patch-add <package> <description> <url>
```

Example:
```
composer patch-add drupal/core "SA-CORE-2018-002" "https://cgit.drupalcode.org/drupal/rawdiff/?h=8.5.x&id=5ac8738fa69df34a0635f0907d661b509ff9a28f"
```
You can omit arguments for an interactive mode.

### Patch Remove
```
composer patch-remove <package> <description>
```

Example:
```
composer patch-remove drupal/core "SA-CORE-2018-002"
```
You can omit arguments for an interactive mode.

### Patch List
```
composer patch-list <package>
```

If the package argument is omitted, the command will return all defined patches.

## Troubleshooting

### Memory limit errors

When running "composer install" you may get some memory limit issues. This is due to the composer dependency resolver since we have a big list of dependencies.
To bypass this issue, you have 3 options:

#### option 1

Temporarely increase the memory limit as described at https://getcomposer.org/doc/articles/troubleshooting.md#memory-limit-errors

#### option 2

If you are creating the project for the first time, use the recommended installation procedure by using "composer create-project" command.

#### option 3

Run "composer update" twice. At first it will throw the same error but on the second attempt it will run successfully.
