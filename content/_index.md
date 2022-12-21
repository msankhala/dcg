
+++
title = "Drupal Common Snippets"
date = 2022-12-10T07:38:51+05:30
+++

# Drupal Common Snippets

Welcome to the Drupal modules snippets website. This website is a curated list
of common Drupal snippets that you might need in day to day development. Here you can find common snippets for both drupal modules and themes. Consider
contributing to this project.

## Drupal 9

### Module info file

**Source:** [Let Drupal know about your module with an .info.yml file](https://www.drupal.org/docs/creating-modules/let-drupal-know-about-your-module-with-an-infoyml-file)

```yaml
name: My Module
description: 'My Module Description'
package: My Package

type: module
ore_version_requirement: ^9.4 || ^10

dependencies:
  - drupal:link
  - drupal:views
  - paragraphs:paragraphs
  - webform:webform (>=6.1.0)

test_dependencies:
 - drupal:image

configure: hello_world.settings

php: 8.0

hidden: true
required: true

# Note: do not add the 'version' or 'project' properties yourself.
# They will be added automatically by the packager on drupal.org.
# version: 1.0
# project: 'hello_world'
```

### Theme info file

**Source:** [Defining a theme with an .info.yml file](https://www.drupal.org/node/2349827)

```yaml
name: Fluffiness
description: 'A cuddly theme that offers extra fluffiness.'
package: My Package

type: theme
core_version_requirement: ^8 || ^9
base theme: stable9

libraries:
  - fluffiness/global-styling

libraries-override:
  contextual/drupal.contextual-links:
    css:
      component:
        /core/themes/stable/css/contextual/contextual.module.css: false

libraries-extend:
  core/drupal.user:
    - fluffiness/user1
    - fluffiness/user2

module_dependencies:
  - my_custom_module:my_custom_module
  - drupal:views
  - paragraphs:paragraphs
  - components:components (>=8.x-2.x)

engine: twig
logo: images/logo.png
screenshot: fluffiness.png

regions:
  header: 'Header'
  content: 'Content'
  sidebar_first: 'Sidebar first'
  footer: 'Footer'

regions_hidden:
  - sidebar_last

features:
  - comment_user_verification
  - comment_user_picture
  - favicon
  - logo
  - node_user_picture

stylesheets-remove:
  - core/assets/vendor/normalize-css/normalize.css
  - '@stable9/css/core/assets/vendor/normalize-css/normalize.css'

ckeditor_stylesheets:
  - https://fonts.googleapis.com/css?family=Open+Sans
  - css/base/elements.css
```
