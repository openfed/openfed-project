This is a project template for Drupal 11 sites built with the Openfed distribution.

## Requirements

- At least PHP 8.3 installed
- Composer 2.7 or newer (required by Drupal 11)

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
composer create-project openfed/openfed-project:~14.0 MYPROJECT
```

#### Second pass: run `composer update` once more

A new project needs **two** Composer runs. The first run (`composer create-project`) downloads everything but does not finish: it stops with messages like these:

```
- Required package "codemirror/codemirror" is not present in the lock file.
...
Script composer install handling the post-update-cmd event returned with error code 4
```

This is expected. After the first run the project does not yet have a `docroot/sites/default/settings.php`, `services.yml` or `files` folder, and the JavaScript libraries of some contrib modules are missing. Finish the installation with:

```
cd MYPROJECT
composer update
```

When the second run is done, `docroot/sites/default/settings.php` exists and the libraries are in `docroot/libraries`. The project is then ready for `drush site:install`.

**Why:** modules like `webform` and `ckeditor_codemirror` list their JavaScript libraries in a `composer.libraries.json` file inside the module. This project merges those files with the `composer-merge-plugin` (see [Contrib Libraries](#contrib-libraries-composer-merge-plugin)), but they only exist once the modules are downloaded. The first run therefore creates a lock file without these libraries. When the project's `post-update-cmd` then runs `composer install`, it finds the libraries missing from the lock file and stops with error code 4 (Composer's "lock file invalid"). The second run includes them.

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

## Contrib Libraries ([composer-merge-plugin](https://github.com/wikimedia/composer-merge-plugin))

This project uses the `wikimedia/composer-merge-plugin` to include additional Composer dependencies from contrib modules.

### How it works

Some contrib modules (like `ckeditor_codemirror` and `webform`) define their own library dependencies in a `composer.libraries.json` file. To have these dependencies installed, the path to the `composer.libraries.json` file must be added to the `merge-plugin.include` array in `composer.json`.

These files only exist after the module is downloaded, so on a new project the libraries are installed by the second `composer update` run. See [Second pass](#second-pass-run-composer-update-once-more).

### Adding new contrib library files

After running `composer create-project`, if you install a contrib module that has a `composer.libraries.json` file, add it to the `merge-plugin.include` array in `composer.json`:

```json
"extra": {
    "merge-plugin": {
        "include": [
            "docroot/modules/contrib/ckeditor_codemirror/composer.libraries.json",
            "docroot/modules/contrib/webform/composer.libraries.json",
            "docroot/modules/contrib/YOUR_MODULE/composer.libraries.json"
        ]
    }
}
```

Then run:

```
composer update
```

This will merge the library dependencies and download them to the appropriate location.

### Notes

- The merge only happens when the file exists, so entries for not-yet-installed modules won't cause errors.
- Run `composer update` after adding new include paths to trigger the merge.

### Custom Composer Installer Types

This project defines and uses custom Composer installer types using:
- [composer/installers](https://github.com/composer/installers)
- [mnsami/composer-custom-directory-installer](https://github.com/mnsami/composer-custom-directory-installer)

Both plugins have to be enabled in `config.allow-plugins`, together with `symfony/runtime`, which Drupal 11.4 requires.

The `theme-library` and `simplesamlphp-module` types are declared by default, so `smillart/framework-sass-source-files` is installed in `docroot/libraries/_sass-framework`. The example below only shows how to place it in a custom theme instead.

#### [_sass-framework](https://openfed.github.io/openfed-libraries/#sass)

The `theme-library` package type is used for frontend assets (for example Sass source files, JavaScript, or shared theme resources) that are not Drupal modules or themes themselves, but are consumed by themes.

##### Installer configuration

```json
"extra": {
  "installer-types": [
    "theme-library"
  ],
  "installer-paths": {
    "docroot/themes/custom/<custom_theme>/{$name}": [
        "smillart/framework-sass-source-files"
    ]
  }
}
```
This ensures that packages of type `theme-library` (including `smillart/framework-sass-source-files`) are installed in a predictable location.

Then run:

```
composer require smillart/framework-sass-source-files
```

##### .gitignore

Add the following line to the project root .gitignore file to make sure we don’t commit these libraries in version control.
```
/docroot/themes/custom/*/_sass-framework
```

##### Why this is required

Composer considers a package type “supported” only if an installer knows where to place it.
Declaring an installer-type without an associated installer-path will result in the error:

> ⚠️ **Package type "theme-library" is not supported**

By defining both:
- installer-types
- and a matching installer-path

Composer is able to resolve and install packages of type theme-library correctly.

## Using Patches

This project uses [Composer Patches 2](https://github.com/cweagans/composer-patches). The patches of the Openfed distribution are applied automatically. The specific patches of your project belong in `composer.patches.json`, referenced by `extra.composer-patches.patches-file`. The core patch depth is set in `extra.composer-patches.package-depths`.

Both settings must stay in your root `composer.json`. Composer Patches 2 does not read them when they are merged in from `composer.openfed.json`, and without the depth the core patches don't apply.

Composer Patches 2 stores the patches it resolved in `patches.lock.json`. Commit this file. After changing patches, run:

```
composer patches-relock
composer patches-repatch
```

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
