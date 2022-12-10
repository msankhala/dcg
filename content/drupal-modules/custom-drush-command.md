---
title: "Custom Drush Command"
date: 2022-12-10T07:43:25+05:30
---

Your module can provide custom drush commands. Create a drush command class file in your module `mymodule/src/Commands/MyCustomDrushCommand.php`

```php

<?php

namespace Drupal\mymodule\Command;

use Drupal\Core\Entity\EntityTypeManagerInterface;
use Drupal\Core\Logger\LoggerChannelFactoryInterface;
use Drupal\Core\StringTranslation\StringTranslationTrait;
use Drupal\Core\DependencyInjection\DependencySerializationTrait;
use Drush\Commands\DrushCommands;
use Drupal\Core\Database\Connection;
use Symfony\Component\Console\Input\InputOption;

/**
 * A Drush commandfile.
 *
 * In addition to this file, you need a drush.services.yml
 * in root of your module.
 *
 * See these files for an example of injecting Drupal services:
 *   - http://cgit.drupalcode.org/devel/tree/src/Commands/DevelCommands.php
 *   - http://cgit.drupalcode.org/devel/tree/drush.services.yml
 */
class MyCustomDrushComm extends DrushCommands {

  use StringTranslationTrait;
  use DependencySerializationTrait;

  /**
   * Entity type service.
   *
   * @var \Drupal\Core\Entity\EntityTypeManagerInterface
   */
  private $entityTypeManager;

  /**
   * Logger service.
   *
   * @var \Drupal\Core\Logger\LoggerChannelFactoryInterface
   */
  private $loggerChannelFactory;

  /**
   * Database connection service.
   *
   * @var \Drupal\Core\Database\Connection
   */
  protected $database;

  /**
   * Constructs a new UpdateVideosStatsController object.
   *
   * @param \Drupal\Core\Entity\EntityTypeManagerInterface $entityTypeManager
   *   Entity type service.
   * @param \Drupal\Core\Logger\LoggerChannelFactoryInterface $loggerChannelFactory
   *   Logger service.
   * @param \Drupal\Core\Database\Connection $database
   *   Database connection manager.
   */
  public function __construct(EntityTypeManagerInterface $entityTypeManager, LoggerChannelFactoryInterface $loggerChannelFactory, Connection $database) {
    $this->entityTypeManager = $entityTypeManager;
    $this->loggerChannelFactory = $loggerChannelFactory;
    $this->database = $database;
  }

  /**
   * Delete the feeds of a given type.
   *
   * @param string $type
   *   Type of feed to delete.
   *
   * @command mymodule:feeds:delete-feeds
   * @aliases mymodule-fdel
   * @option all
   *   Delete all feeds. If all option is given then type is ignored.
   *
   * @usage mymodule:feeds:delete-feeds <feed-type>
   *   <feed-type> is the type of feeds to delete.
   * @usage mymodule:feeds:delete-feeds --all
   *   Delete all feeds.
   */
  public function deleteFeeds($type = '', array $options = [
    'all' => FALSE,
    // You can add more options here, as per the @option annotation.
  ]) {

    $all = $this->input()->getOption('all');
    if ($all) {
      if ($this->confirmDelete()) {
        $this->deleteAllFeedsByTruncate();
        $this->logger()->success($this->t('All feeds deleted.'));
        $this->loggerChannelFactory->get('mymodule')->info('All feeds deleted.');
        return;
      }
    }

    // Check if the type is valid and get the fids.
    if (!empty($type) && $this->isValidFeedType($type)) {
      if ($this->confirmDelete($type)) {
        $fids = $this->getFeedsByType($type);
      }
    }
    else {
      $feedTypes = $this->getAllFeedTypes();
      $this->logger()->error(dt("Invalid feed type given, please provide a valid feed type to delete."));
      $this->logger()->error(dt("Available feed types are: @types", ['@types' => implode(', ', $feedTypes)]));
      return;
    }

    // Early return if no feeds to delete.
    if (empty($fids)) {
      $this->logger()->notice($this->t('No feeds to delete.'));
      return;
    }

    // 1. Log the start of the script.
    $this->logger()->notice(dt('Start deleting feeds of type @type', ['@type' => $type]));
    $this->loggerChannelFactory->get('mymodule')->info(dt('Start deleting feeds of type @type', ['@type' => $type]));

    // Prepare the operation. Here we could do other operations on feeds.
    try {
      // Write logic to delete feeds.

      // Show some information.
      $this->logger()->success($this->t('Finished deleting feeds of type @type', ['@type' => $type]));
      // Add the same message to drupal logs.
      $this->loggerChannelFactory->get('mymodule')->info($this->t('Finished deleting feeds of type @type', ['@type' => $type]));

    }
    catch (\Exception $e) {
      $this->logger()->error('Encountered the following error while trying to delete the feed: @error',
        ['@error' => $e]
      );
      $this->loggerChannelFactory->get('mymodule')->info('Encountered the following error while trying to delete the feed: @error',
        ['@error' => $e]
      );
    }
  }

  /**
   * Delete all feed types.
   */
  private function deleteAllFeedsByTruncate() {
    $this->database->truncate('feeds_feed')->execute();
  }

  /**
   * Get all the feed types.
   */
  private function getAllFeedTypes() {
    // Get the all the distinct types of feeds and create an array.
    $query = $this->database->select('feeds_feed', 'ff');
    $query->fields('ff', ['type']);
    $query->distinct();
    $result = $query->execute()->fetchAll();
    $types = [];
    foreach ($result as $item) {
      $types[] = $item->type;
    }
    return $types;
  }

  /**
   * Get all the feeds of a specific type.
   *
   * @param string $type
   *   Feed type.
   */
  private function getFeedsByType($type) {
    // Get the all the feeds of a specific type.
    $storage = $this->entityTypeManager->getStorage('feeds_feed');
    $query = $storage->getQuery()
      ->condition('type', $type);
      // ->range(0, 1);
    $fids = $query->execute();
    return $fids;
  }

  /**
   * Get all the feeds of all types.
   */
  private function getAllFeeds() {
    // Get the all the feeds.
    $storage = $this->entityTypeManager->getStorage('feeds_feed');
    $query = $storage->getQuery();
    $fids = $query->execute();
    return $fids;
  }

  /**
   * Check for confirmation.
   *
   * @param string $type
   *   Feed type. If type is passed then it will check for confirmation for
   *   specific type, else it will check for confirmation for all types.
   * @return bool
   *   Confirmation.
   */
  private function confirmDelete($type = '') {
    // Ask for confirmation as this operation can not be undone.
    if ($type) {
      $question = $this->t('Are you sure you want to delete all feeds of type @type?', ['@type' => $type]);
      $warning = $this->t('This operation can not be undone. Are you sure you want to delete all feeds of type @type?', ['@type' => $type]);
    }
    else {
      $question = $this->t('Are you sure you want to delete all feeds?');
      $warning = $this->t('This operation can not be undone. Are you sure you want to delete all feeds?');
    }
    $this->io()->warning($question);

    return $this->io()->confirm($warning);
  }

  /**
   * Check if feed type is valid.
   */
  private function isValidFeedType($type) {
    $types = $this->getAllFeedTypes();
    if (in_array($type, $types)) {
      return TRUE;
    }
    return FALSE;
  }

}

```
