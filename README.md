sfEnvironmentFixturesPlugin
===========================

The `sfEnvironmentFixturesPlugin` is a symfony plugin that provides support
for defining additional fixtures for each symfony environment.
The plugin does so in such a way that any symfony task which internally uses
the `data-load` task will function in the same way.

This plugin works with either Doctrine or Propel.

Installation
------------

* Install the plugin as submodule:

        $ git submodule add git@github.com:abarry/sfEnvironmentFixturesPlugin.git plugins/sfEnvironmentFixturesPlugin

* Enable the plugin in `config/ProjectConfiguration.class.php`:

        class ProjectConfiguration extends sfProjectConfiguration
        {
          public function setup()
          {
            $this->enablePlugins(array(
              'sfEnvironmentFixturesPlugin',
            ));
          }
        }

Usage
-----

The plugin will search the data directory (usually `data`)
for a sub-directory that follows the naming convention:

    fixtures_<environment name>

For example, when running in the `dev` environment, fixtures will be
loaded from `data/fixtures_dev`.

Simply place the fixture files in the corresponding directories and they
will be loaded any time the `data-load` task is invoked.

You should see output similar to the following:

    >> doctrine  Including data fixtures from "/path/to/project/data/fixtures"
    >> fixtures  Including data fixtures for environment "dev"
    >> fixtures  Including data fixtures from "/path/to/project/data/fixtures_dev"
    >> doctrine  Data was successfully loaded

You may see some messages repeated depending on the task being run.

Basic example
-------------

Consider the following file structure:

    /path/to/project
      ...
      /data
        /fixtures
          fixtures.yml
        /fixtures_dev
          fixtures.yml
      ...

With the main fixtures file containing:

    # /path/to/project/data/fixtures/fixtures.yml

    Fruit:
      fruit_apple:
        name: apple
      fruit_banana:
        name: banana

And the dev fixtures file containing:

    # /path/to/project/data/fixtures_dev/fixtures.yml

    Fruit:
      fruit_orange:
        name: orange
      fruit_peach:
        name: peach

Running the `data-load` task and specifying the `dev` environment
(using `--env=dev`) would give us the following data set:

    +----+--------+
    | id | name   |
    +----+--------+
    |  1 | apple  |
    |  2 | banana |
    |  3 | orange |
    |  4 | peach  |
    +----+--------+

In this case fixtures are first loaded from the standard symfony directory,
followed by the fixtures directory for the `dev` environment.

If we were to specify the `prod` environment by using `--env=prod` we would
end up with a different set of data:

    +----+--------+
    | id | name   |
    +----+--------+
    |  1 | apple  |
    |  2 | banana |
    +----+--------+

In this case the `dev` fixtures are ignored and only the fixtures from the
standard symfony directory are loaded. However, if we had a `fixtures_prod`
directory, it's contents would also be loaded.

Running the `data-load` task without specifying an environment would give us
the following data set:

    +----+--------+
    | id | name   |
    +----+--------+
    |  1 | apple  |
    |  2 | banana |
    |  3 | orange |
    |  4 | peach  |
    +----+--------+

This is because the default environment for tasks is `dev`.

Example with relationships
--------------------------

Now consider a more complex file structure:

    /path/to/project
      ...
      /data
        /fixtures
          /fruit
            categories.yml
            fruits.yml
        /fixtures_dev
          /fruit
            fruits.yml
        /fixtures_prod
          /fruit
            fruits.yml
      ...

With the main fixture files containing:

    # /path/to/project/data/fixtures/fruit/categories.yml

    Category:
      category_citrus:
        name: citrus
      category_stone_fruit:
        name: stone fruit

And:

    # /path/to/project/data/fixtures/fruit/fruits.yml

    Fruit:
      fruit_apple:
        name: apple
      fruit_banana:
        name: banana

The dev fixtures file containing:

    # /path/to/project/data/fixtures_dev/fruit/fruits.yml

    Fruit:
      fruit_orange:
        name: orange
        Category: category_citrus

And the prod fixtures file containing:

    # /path/to/project/data/fixtures_prod/fruit/fruits.yml

    Fruit:
      fruit_peach:
        name: peach
        Category: category_stone_fruit

Running the `data-load` task and specifying the `dev` environment
(using `--env=dev`) would give us the following data set:

    +----+--------+-------------+
    | id | name   | category_id |
    +----+--------+-------------+
    |  1 | apple  | NULL        |
    |  2 | banana | NULL        |
    |  3 | orange | 1           |
    +----+--------+-------------+

In this case fixtures are first loaded from the standard symfony directory,
followed by the fixtures directory for the `dev` environment.

As you can see, even though the categories are stored in a different fixtures
directory to the orange, it is still able to reference the citrus category
using the `category_citrus` record label.

If we were to specify the `prod` environment by using `--env=prod` we would
end up with a different set of data:

    +----+--------+-------------+
    | id | name   | category_id |
    +----+--------+-------------+
    |  1 | apple  | NULL        |
    |  2 | banana | NULL        |
    |  3 | peach  | 2           |
    +----+--------+-------------+

Showing that the fixtures for the `prod` environment have been loaded instead.

This example also shows that the plugin is capable of reading fixtures from
sub-directories, just like the main symfony fixtures directory.

Important points
----------------

* The default environment for the `data-load` task is `dev`, so unless you
specify an environment with the `--env` option, fixtures in the
`fixtures_dev` directory will be loaded, which could be very bad in a
production environment.

* Fixtures are *always* loaded from the main `fixtures` directory regardless
of the current environment.

* Environment-specific fixtures can still reference record labels from the
main fixture file, making references easy to set up.

* It's a good idea to keep record labels unique across all fixtures. This will
avoid any confusion when specifying relationships in your fixtures.

* Fixtures are loaded in a recursive manner similar to the main fixtures
directory, meaning you can have fixtures in sub-directories if you so desire.

* Fixtures can still contain PHP code.
