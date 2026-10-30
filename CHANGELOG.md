CHANGELOG
=========

30 October 2026 - Version 14.0.0-alpha1
------------------------------
- The project template now targets Drupal 11 and requires PHP 8.3 (#15).
- Updated the Composer setup: Composer Patches 2 (its settings are now in composer.json), the new custom installer plugin and symfony/runtime.
- Removed the CKEditor 4 installer types.
- A new project needs a second "composer update" after "composer create-project". See the README.

25 February 2026 - Version 13.6.2
------------------------------
- The composer-merge-plugin package version is now limited.

09 February 2026 - Version 13.6.1
------------------------------
- The "openfed-upgrade" script is now also included to facilitate major Openfed upgrades.
- The _sass-framework library can now be required via Composer for custom themes.
- Reordered the merge-plugin includes to ensure compatibility with ckeditor_codemirror, which requires version 5.6.21.
