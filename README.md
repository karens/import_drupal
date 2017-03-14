# Import From Drupal

This is a module that is designed as an example of how to import nodes and images from a Drupal 8 site by pulling in data from a JSON feed. Much of this would be useful and relevant for pulling content in from any other JSON source.

## Prepare the Source Site
There is not much preparation needed. It should be a Drupal 8 site with an 'article' content type that has a body and 'field_image' image field.

Download and install the [JSON API](https://www.drupal.org/project/jsonapi) module. Navigate to a url like http://source-site.com/jsonapi/node/article and confirm that JSON is being output at that url.

That's it for the source.

## Prepare the Target Site

The target site should be running Drupal 8.2.3 or higher. There are changes to the way file imports work that won't work in earlier versions. It should already have a matching article content type and field_image field ready to accept the articles from the other site.

Enable the core Migrate module. Download and enable the [Migrate Plus](https://www.drupal.org/project/migrate_plus) and [Migrate Tools](https://www.drupal.org/project/migrate_tools) modules. Make sure to get the versions that are appropriate for the current version of core. Migrate Plus had 8.0 and 8.1 branches that only work with outdated versions of core, so currently you need version 8.2 of Migrate Plus.

## Edit the Migration YAML Files

Copy the Drupal Import module (this module) into your module repository. Edit the YAML files in /config/optional to alter the JSON source url so it points to the domain for the source site created in the earlier step.

The primary path used for our migration is:

```
http(s)://SOURCE-SITE.COM/jsonapi/node/article
```

This will display a JSON feed of all articles. The articles have related entities. The field_image field points to related images, and the uid/author field points to related users. To view the related images, we can alter the path as follows:

```
http(s)://SOURCE-SITE.COM/jsonapi/node/article?include=field_image
```

That will add an 'included' array to the feed that contains all the details about each of the related images.

Swapping out the domain may be the only change needed, and it's a good place to start. Read the JSON API module documentation to explore other changes you might want to make to that configuration to limit the fields that are returned, or sort or filter the list.

## Run the Migration

Once you have the right information in the YAML files, enable the module. On the command line, type this:

```
drush migrate-status
```

You should see two migrations available to run. To run them, type:

```
drush mi --all
```

The first migration is ```import_drupal_images```. This has to be run before ```import_drupal_articles```, because field_image is a reference to an image file. This migration uses the path that includes the image details, and just ignores the primary feed information.

The second migration is ```import_drupal_articles```. This pulls in the article information using the same url, this time without the included images. When each article is pulled in, it is matched to the image that was pulled in previously.

You can run one migration at a time, or even just one item at a time, while testing this out:

```
drush migrate-import import_drupal_images --limit=1
```

You can rollback and try again.

```
drush migrate-rollback import_drupal_images
```

It is important to note that if you alter the YAML files after you first install the module, you'll have to uninstall and then reinstall the module to get Migrate to see the YAML changes.

There is more information about the [Migrate API on Drupal.org](https://www.drupal.org/docs/8/api/migrate-api/migrate-api-overview).
