---
title: Queries
---

# Queries
![views](https://api.visitor.plantree.me/visitor-badge/pv?label=views&color=informational&namespace=d9book&key=queries.md)

For most work, I use entityQueries. There are a few circumstances where I've needed to get into the SQL which meant using static or dynamic queries. There are examples of these different techniques below.

## entityQuery

### Find matching nodes entityQuery

In this EntityQuery example we search for nodes of content type (bundle) ws_product and match field_product_sku with the \$sku variable.

use Drupal\node\Entity\Node;

```php
function getProductId($sku) {
  $productId = false;
  $query = \Drupal::entityQuery('node')
    ->condition('type', 'ws_product')
    ->condition('field_product_sku', $sku)
    ->accessCheck(FALSE);

  $nids = $query->execute();
  if ($nids) {
    $nid = array_values($nids);
    $node = Node::load($nid[0]);
    $productId = $node->get('field_product_id')->value;
  }
  return $productId;
}
```

### Find matching nodes entityQuery

In this entityQuery we search for published nodes of type contract with field_contract_status having the value "Active". This puts the resulting nids and node titles in a render array for display.

This is a simple query which outputs a bunch of nids and titles

```php
public function loadRawSalesforceData() {
  $node_storage = \Drupal::entityTypeManager()->getStorage('node');
  $query = \Drupal::entityQuery('node')
    ->condition('type', 'contract')
    ->condition('status', 1)
    ->condition('field_contract_status', 'Active')
    ->sort('title', 'DESC')
    ->accessCheck(FALSE);

  $nids = $query->execute();
  if ($nids) {
    $nodes = $node_storage->loadMultiple($nids);
    foreach ($nodes as $node) {
       $nid = $node->id();
       $titles[] = [
         '#type' => 'markup',
          '#markup' => "<p>" . "nid=$nid " . "Title=" . $node->getTitle(). "</p>",
       ];
    }
    return $titles;
  }
  return [
    '#markup' => $this->t('nothing, nada, not a sausage'),
    ];
}
```

### Find matching article nodes entityQuery

This example looks for an entity of type `article` with the name `$name`

```php
public function entityExists() {

  $name = 'hello';
  // See if the article named hello exists.
  $query = \Drupal::entityQuery('node')
    ->condition('type', 'article')
    ->condition('title', $name)
    ->count()
    ->accessCheck(FALSE);

  $count_nodes = $query->execute();

  if ($count_nodes == 0) {
    $str = "Found no articles";
  }
  elseif ($count_nodes > 0) {
    $str = "Found $count_nodes articles";
  }
  $render_array['content'] = [
    '#type' => 'item',
    '#markup' => $str,
  ];

  return $render_array;
}
```

### Find nodes that match a taxonomy term entityQuery

Find all nodes that match a term_id and retrieve the first 5 nodes sorted by title. This code also puts them into a render array for display.

```php
protected function loadFirstOpinion($term_id) {
  $storage = \Drupal::entityTypeManager()->getStorage('node');
  $query = \Drupal::entityQuery('node')
    ->condition('status', 1)
    ->condition('type', 'opinion')
    ->condition('field_category', $term_id, '=')
    ->sort('title', 'ASC') //or DESC
    ->range(0, 5)
    ->accessCheck(FALSE);
  $nids = $query->execute();
  $nodes = $storage->loadMultiple($nids);

  $render_array = [];
  foreach ($nodes as $node) {
    $render_array[] = [
      '#type' => 'markup',
      '#markup' => '<p>' . $node->getTitle(),
    ];
  }
  return $render_array;
```

### Find 5 nodes that have a matching taxonomy term entityQuery

We look for published nodes of node type `opinion` that have a term in the category field, sorted by title ascending, starting with the first result and giving us 5 results. The resulting titles are put into a render array.

```php
protected function loadFirstOpinion($term_id) {
  $storage = \Drupal::entityTypeManager()->getStorage('node');
  $query = \Drupal::entityQuery('node')
    ->condition('status', 1)
    ->condition('type', 'opinion')
    ->condition('field_category', $term_id, '=')
    ->sort('title', 'ASC') //or DESC
    ->range(0, 5)
    ->accessCheck(FALSE);
  $nids = $query->execute();
  $nodes = $storage->loadMultiple($nids);

  $render_array = [];
  foreach ($nodes as $node) {
    $render_array[] = [
      '#type' => 'markup',
      '#markup' => '<p>' . $node->getTitle(),
    ];
  }
  return $render_array;
```

### Find matching nodes and delete them entityQuery

```php
public function deleteQuery1() {
  $results = \Drupal::entityQuery('node')
    ->condition('type', 'event')
    ->range(0, 10)
    ->accessCheck(FALSE)
    ->execute();

  if ($results) {
    foreach ($results as $result) {
      $node = Node::load($result);
      $node->delete();
    }
  }
  $render_array['content'] = [
    '#type' => 'item',
    '#markup' => t("10 nodes deleted."),
  ];

  return $render_array;
}
```

### Sort by Node ID, title (nid)

```php
// Descending.
$query->sort('nid', 'DESC');
// Or ascending.
$query->sort('nid', 'ASC');
// Or node title.
$query->sort('title', 'ASC');
``````

### Slice up entityQuery results into batches of 100 nodes

This is often used for batch API operations.

```php
$query = \Drupal::entityQuery('node')
  ->condition('type', 'contract')
  ->condition('status', 1)
  ->sort('title', 'ASC')
  ->accessCheck(FALSE);
$nids = $query->execute();
$nid_count = count($nids);

// Grab 100 nids at a time to batch process.
$batches = [];
for ($i=0;$i<=$nid_count;$i+=100) {
  $batches[] = array_slice($nids, $i, 100);
}
```

### Query the creation date (among other things) using entityQuery

Note. The created (and changed) field uses a unix timestamp. This is an `int` 11 field in the db with a value like `1525302749` If you add a Drupal `datefield`, its data looks like `2019-05-15T21:32:00` (varchar 20)

If you want to query a date field in a content type, you will have to fiddle around with the setTimezone stuff that is commented out below. The date field referenced below (`field_date`) is a standard Drupal date field.

More at <https://blog.werk21.de/en/2018/02/05/date-range-fields-and-entity-query-update> and <https://drupal.stackexchange.com/questions/198324/how-to-do-a-date-range-entityquery-with-a-date-only-field-in-drupal-8>

```php
  protected function loadOpinionForAYear($year, $term_id) {
    $storage = \Drupal::entityTypeManager()->getStorage('node');

    // Get a date string suitable for use with entity query.
    //    $date = new DrupalDateTime();  // now
    $format = 'Y-m-d H:i';
    $start_date = DrupalDateTime::createFromFormat($format, $year . "-01-01 00:00");
    $end_date = DrupalDateTime::createFromFormat($format, $year . "-12-31 23:59");

    $start_date = $start_date->getTimestamp();
    $end_date = $end_date->getTimestamp();

//    $start_date->setTimezone(new \DateTimeZone(DateTimeItemInterface::STORAGE_TIMEZONE));
//    $end_date->setTimezone(new \DateTimeZone(DateTimeItemInterface::STORAGE_TIMEZONE));
//    $start_date = $start_date->format(DateTimeItemInterface::DATETIME_STORAGE_FORMAT);
//    $end_date = $end_date->format(DateTimeItemInterface::DATETIME_STORAGE_FORMAT);

// Set the condition.
//    $query->condition('field_date.value', $start_date, '>=');
//    $query->condition('field_date.value', $end_date, '<=');


    $query = \Drupal::entityQuery('node')
      ->condition('status', 1)
      ->condition('type', 'opinion')
      ->condition('field_category', $term_id, '=')
      ->condition('created', $start_date, '>=')
      ->condition('created', $end_date, '<=')
      ->sort('title', 'DESC')
      ->accessCheck(FALSE);
    $nids = $query->execute();
    $titles = [];
    if ($nids) {
      $nodes = $storage->loadMultiple($nids);
      foreach ($nodes as $node) {
        $titles[]= $node->getTitle();
      }
    }
    return $titles;

  }
```

### entityQuery frequently used conditions

-   Published: `->condition('status', 1)`

-   Text field not empty: `->condition('field_source_url', '', '<>')`

-   Field value \> 14: `->condition('field_some_field', 14, '>')`

-   Reference field empty: `->notExists('field_sf_account_ref');`

-   Null: `->condition($field, NULL, 'IS NULL');`

-   Not Null: `->condition($field,NULL, 'IS NOT NULL');`

Lots more at
<https://www.drupal.org/docs/8/api/database-api/dynamic-queries/conditions>

### Update menu items programatically

To update several items in a menu use hook_update.

```php
function park_academy_update_8002() {

  $mids = \Drupal::entityQuery('menu_link_content')
    ->condition('menu_name', 'park-wide-utility')
    ->accessCheck(FALSE)
    ->execute();

  foreach($mids as $mid) {

    $menu_link = \Drupal::entityTypeManager()->getStorage('menu_link_content')->load($mid);

    $title = $menu_link->getTitle();
    if ($title === 'Support') {
      $menu_link->set('weight',2);
      $menu_link->set('expanded', TRUE);
//      $menu_link->set('title','yomama');
      $menu_link->set('link', 'https://www.google.com');
      $menu_link->save();

    }
  }
}
```

### Query multi-value fields

When querying multivalue fields, you need to use `%delta` to specify the position (or delta) for the value you are looking for.  You also have to identify to the query which position (or delta) you want to query.  In the example below, we specify `field_srp_voting_status.%delta` as 1 - indicating the second position (0 based always) and `field_srp_voting_status.%delta.value` for the actual value we are looking for (either accepted, rejected or incomplete):

```php
    $vote_number = 1;
    $query = \Drupal::entityQuery('node')
      ->condition('type', 'correlation', '=')
      ->accessCheck(FALSE)
      ->condition('field_program', $this->programNid, '=')
      ->condition('field_voting_status.%delta', $vote_number, '=')
      ->condition('field_voting_status.%delta.value', [
        'accepted',
        'rejected',
        'incomplete'
      ], 'IN')
      ->accessCheck(FALSE);
    $correlation_nids = $query->execute();
    $correlation_nids = array_values($correlation_nids);
    return $correlation_nids;
```


### Query entity reference fields if they have a value or no value
To check if there is a value in an entity reference fields, use the following code

```php
$query = \Drupal::entityQuery('node')
  ->condition('type', 'srp_voting_record')
  ->accessCheck(FALSE);
if ($vote_type == 'citation') {
  // Check for empty entity reference field.
  $query->notExists('field_ref_error_feedback');
}
if ($vote_type == 'feedback_error'){
  // Check for filled entity reference field.
  $query->exists('field_ref_error_feedback');
}

```

### Entity reference fields entityQuery

In the following query, we check for a value in the entity that is referenced in the entity reference field?  For example, if you have an entity reference field which references node (entity) 27.  This query can look in node 27 and check a field value in that node.  Here we check in field_first_name for the the value `Fred`:

```php
      ->condition('field_tks_pub_expectation.entity.field_first_name', 'Fred', '=')
```

For querying for a user id, we query the `field_voter.entity:user.uid` value.  See code below:

```php
protected function loadErrorFeedbackVotingRecordNode(int $user_id, int $error_feedback_nid, int $vote_number) {
  $node = [];
  $query = \Drupal::entityQuery('node')
    ->condition('type', 'srp_voting_record')
    ->condition('field_voter.entity:user.uid', $user_id)
    ->condition('field_ref_error_feedback', $error_feedback_nid)
    ->condition('field_srp_vote_number', $vote_number)
    ->accessCheck(FALSE);
  $nids = $query->execute();
  if (!empty($nids)) {
    $nid = reset($nids);
    $node = Node::load($nid);
  }
  return $node;
}
```

### Find nodes that were modified recently

To select for nodes that were modified within the last seven days, use the following:

```php
$num_days_string = '-7 days';
$query = \Drupal::entityQuery('node')
  ->condition('type', 'srp_voting_record')
  ->condition('changed', strtotime($num_days_string), '>=');
  ->accessCheck(FALSE)
$voting_record_nids = $query->execute();
if (empty($voting_record_nids)) {
  return [];
}
// Re-arrange indexing as 0, 1, 2...
$voting_record_nids = array_values($voting_record_nids);
```

### andCondition and orCondition example

Here is an example of a submitForm function where after a node is deleted, an additional query is fired off which looks for nodes that are either:
1. Of type `srp_voting_records` with matching $ef_node_id's
2. Of type `srp_publisher_response` with matching $ef_node_id's

Then it deletes those nodes it found.

Note. the `ef` in `$ef_node_id` stands for error/feedback if that makes the code any clearer.

```php
  public function submitForm(array &$form, FormStateInterface $form_state) {
    $ef_node_id = $form_state->get('ef_node_id');
    $program_nid = $form_state->get('program_nid');
    $type_text = $form_state->get('type_text');
    $node = Node::load($ef_node_id);
    if($node) {
      $node->delete();
      \Drupal::messenger()->addMessage("{$type_text} deleted.");
      $storage = \Drupal::entityTypeManager()->getStorage('node');
      $query = \Drupal::entityQuery('node')
        ->accessCheck(FALSE)
        ->sort('title', 'ASC');
      $orCorr1 = $query->andConditionGroup()
        ->condition('type', 'srp_voting_record', '=')
        ->condition('field_ref_error_feedback', $ef_node_id, '=');
      $orCorr2 = $query->andConditionGroup()
        ->condition('type', 'srp_publisher_response', '=')
        ->condition('field_srp_feedback_error_item', $ef_node_id, '=');
      $orCorr3 = $query->orConditionGroup()
        ->condition($orCorr1)
        ->condition($orCorr2);
      $query->condition($orCorr3);
      $nids = $query->execute();
      foreach($nids as $nid)
      {
        $delete_node = Node::load($nid);
        $delete_node->delete();
      }
    }
    else {
      \Drupal::messenger()->addMessage("{$type_text} Not Found.");
    }

    $referrer_alias = $form_state->get("referrer_alias");
    $url = Url::fromUri('internal:' . $referrer_alias);
    $form_state->setRedirectUrl($url);
  }
```


### More details about condition

from - [API documentation for QueryInterface::condition](https://api.drupal.org/api/drupal/core%21lib%21Drupal%21Core%21Entity%21Query%21QueryInterface.php/function/QueryInterface%3A%3Acondition/8.6.x)


**Language specific query**

This example shows searching for entities with both the Turkish \'merhaba\' (notice \'tr\' as the last parameter) and the Polish \'siema\' (notice \'pl\' as the last parameter) within a \'greetings\' text field:

```php
$entity_ids = \Drupal::entityQuery($entity_type)
  ->accessCheck(FALSE)
  ->condition('greetings', 'merhaba', '=', 'tr')
  ->condition('greetings.value', 'siema', '=', 'pl')
  ->execute();
```

**Parameters** 

Notice that the first parameter can be a `string` or a `ConditionInterface`:

`string|\Drupal\Core\Entity\Query\ConditionInterface $field`: Name of the field being queried or an instance of ConditionInterface. In the case of the name, it must contain a field name, optionally followed by a column name. The column can be the reference property, usually "entity", for reference fields and that can be followed similarly by a field name and so on. Additionally, the target entity type can be specified by appending the ":target_entity_type_id" to "entity". Some examples:

- nid
- tags.value
- tags
- tags.entity.name
- tags.entity:taxonomy_term.name
- uid.entity.name
- uid.entity:user.name

\"tags\" is the same as \"tags.value\" as value is the default column. If two or more conditions have the same field names they apply to the same delta within that field. In order to limit the condition to a specific item a numeric delta should be added between the field name and the column name.

```php
->condition('tags.5.value', 'news')
```  
This will require condition to be satisfied on a specific delta of the field. The condition above will require the 6th value of the field to match the provided value. Further, it's possible to create a condition on the delta itself by using '%delta'. For example,

```php
  ->condition('tags.%delta', 5)
```  
will find only entities which have at least six tags. Finally, the condition on the delta itself accompanied with a condition on the value will require the value to appear in the specific delta range. For example,

```php
  ->condition('tags.%delta', 0, '>'))
  ->condition('tags.%delta.value', 'news'))
```  

will only find the "news" tag if it is not the first value. It should be noted that conditions on specific deltas and delta ranges are only supported when querying content entities.

`string|int|bool|array|null $value`: (optional) The value for $field. In most cases, this is a scalar and it's treated as case-insensitive. For more complex operators, it is an array. The meaning of each element in the array is dependent on $operator. Defaults to NULL, for most operators (except: `'IS NULL', 'IS NOT NULL'`) it always makes the condition false.

`string|null $operator`: (optional) The comparison operator. Possible values:
`'=', '<>', '>', '>=', '<', '<=', 'STARTS_WITH', 'CONTAINS', 'ENDS_WITH'`: These operators expect `$value` to be a literal of the same type as the column.
`'IN', 'NOT IN'`: These operators expect `$value` to be an array of literals of the same type as the column.
`'IS NULL', 'IS NOT NULL'`: These operators ignore `$value`, for that reason it is recommended to use a `$value` of NULL for clarity.
`'BETWEEN', 'NOT BETWEEN'`: These operators expect `$value` to be an array of two literals of the same type as the column.
If NULL, defaults to the `'='` operator.

`string|null $langcode`: (optional) The language code allows filtering results by specific language. If two or more conditions omit the langcode within one condition group then they are presumed to apply to the same translation. If within one condition group one condition has a langcode and another does not they are not presumed to apply to the same translation. If omitted (`NULL`), any translation satisfies the condition.



### User Query example

```php
  /**
   * Get the active publisher admins and editors for this publisher nid.
   *
   * @param int $publisher_nid
   *   The publisher nid.
   * @param array $roles
   *   The roles to check for.
   *
   * @return array
   *   The active publisher admins and editors for this publisher nid.
   */
  public static function getActivePublisherUsers(int $publisher_nid, array $roles = ['publisher_edit', 'publisher', 'publisher_staff', 'publisher_view']): array {
    $uids = \Drupal::entityQuery('user')
      ->condition('field_teks_publisher', $publisher_nid)
//      ->condition('roles.target_id', ['publisher_edit', 'publisher'], 'IN')
      ->condition('roles', ['publisher_edit', 'publisher'], 'IN')
      ->condition('status', 1)
      ->sort('created', 'ASC')
      ->accessCheck(FALSE)
      ->execute();

    $publisher_editors = [];
    foreach ($uids as $uid) {
      $user = User::load($uid);
      $publisher_editors[] = [
        'uid' => $uid,
        'name' => $user->getAccountName(),
        'email' => $user->getEmail(),
      ];
    }
    return $publisher_editors;
  }
```

### Paragraph entityQuery example

This code looks up related paragraphs of type `accordio_video_section`, grabs the first one (this should have used a sort to more reliably return the same value), then finds all `video_collection` nodes in that collection. In essence, this finds a list of other videos that are in the collection for the video you are viewing.

```php
  public function buildForm(array $form, FormStateInterface $form_state, $nojs = NULL) {

...

    // Grab nid of the current video detail node.
    $nid = 0;
    $nodeTitle = '';
    $default = '';
    $node = \Drupal::routeMatch()->getParameter('node');
    if ($node instanceof \Drupal\node\NodeInterface) {
      $nid = $node->id();
      $nodeType = $node->bundle();
      $nodeTitle = $node->getTitle();
      // Default value for select element.
      $default = '/node/' . $nid;
    }

    $form['title'] = [
      '#type' => 'markup',
      '#markup' => $nodeTitle,
    ];

    // Find a paragraph that has the node I am currently viewing.
    $paragraph_id = 0;
    $storage = $this->entityTypeManager->getStorage('paragraph');
    $query = \Drupal::entityQuery('paragraph')
      ->condition('status', 1)
      ->condition('type', 'accordio_video_section')
      ->condition('field_content_relation', $nid)
      ->accessCheck(FALSE);
    $ids = $query->execute();
    if (!$ids) {
      return;
    }
    $paragraphs = $storage->loadMultiple($ids);
    if ($paragraphs) {
      // Grab the first one.
      $paragraph_id = reset($paragraphs)->id();
    }

    // Find the video collection node that has this paragraph.
    $nids = [];
    $video_collection_title = '';
    if ($paragraph_id) {
      $storage = $this->entityTypeManager->getStorage('node');

      $query = \Drupal::entityQuery('node')
        ->condition('type', 'video_collection')
        ->condition('status', 1)
        ->condition('field_video_accordions', $paragraph_id)
        ->accessCheck(FALSE);
      $nids = $query->execute();
    }
    if ($nids) {
      $video_collection_nid = reset($nids);
      $video_collection_node = Node::load($video_collection_nid);
      $video_collection_title = $video_collection_node->label();
    }

    $form['collection_title'] = [
      '#type' => 'markup',
      '#markup' => $video_collection_title,
    ];


    // Now go find all the video's in this video_collection.
    $videos = [];
    $storage = $this->entityTypeManager->getStorage('paragraph');
    // Grab all the accordions and extract out the videos.
    if (isset($video_collection_node)) {
      $accordions = $video_collection_node->get('field_video_accordions');
      foreach ($accordions as $accordion) {
        if ($accordion->entity->getType() == 'accordio_video_section') {
          $id = $accordion->target_id;
          $accordion_paragraph = $storage->load($id);
          $accordion_videos = reset($accordion_paragraph->field_content_relation);
          $videos = array_merge($videos, $accordion_videos);
        }
      }
    }
```
From ~Sites/inside-mathematics/modules/custom/dana_pagination/src/Form/VideoPaginationForm.php.


## Static and Dynamic Queries

Sometimes you will use static or dynamic queries rather than entityQueries. These use actual SQL versus the `entityQuery` approach where you build the various parts of the query using PHP methods.

Dynamic queries let the Drupal database driver generate the sql string and therefore has more flexibility in the resulting sql string. Static queries are just a sql string which have no flexibility in making small adjustments for a specific database back-end. This means may not work for other databases. The core supported databases are MySQL, PostgreSQL and SQLite. When we use dynamic queries, they should work for more/all databases. Dynamic queries are however a little bit slower than static queries.

An example static query is:

```php
$database = \Drupal::database();
$query = $database->query("SELECT id, example FROM {mytable}");
$result = $query->fetchAll();
```

Dynamic queries refer to queries that are built dynamically by Drupal rather than provided as an explicit query string. All Insert, Update, Delete, and Merge queries must be dynamic. Select queries may be either static or dynamic. Therefore, "dynamic query" generally refers to a dynamic Select query.

For this static query:
```php
$result = $database->query("SELECT uid, name, status, created, access FROM {users_field_data} u WHERE uid <> 0 LIMIT 50 OFFSET 0");
```

The equivalent dynamic query is:

```php
// Create an object of type Select and directly add extra detail
// to this query object: a condition, fields and a range.
$query = $database->select('users_field_data', 'u')
  ->condition('u.uid', 0, '<>')
  ->fields('u', ['uid', 'name', 'status', 'created', 'access'])
  ->range(0, 50);
```



For more info check out
- [Static Queries on drupal.org - updated April 2023](https://www.drupal.org/docs/drupal-apis/database-api/static-queries)
- [Dynamic Queries on drupal.org](https://www.drupal.org/docs/8/api/database-api/dynamic-queries)


### Get a connection object

There are two ways to get a connection object:

```php
/** @var \Drupal\Core\Database\Connection $connection */
$connection = Database::getConnection();

//OR 

/** @var \Drupal\Core\Database\Connection $connection */
$connection = \Drupal::service('database');
```

### SQL select example

Static query example from a controller. This loads some fields from the donors table and returns a render array with a count of how many results it found.

```php
public function queryBuild1() {
  $database = \Drupal::database();
  $query = $database->query("SELECT id, name, amount  FROM {donors}");
  $results = $query->fetchAll();

  $result_count = count($results);

  $str = "Results from db query";
  $str .= "<br/> Result count = $result_count";

  $render_array['content'] = [
    '#type' => 'item',
    '#markup' => $str,
  ];

  return $render_array;
}
```

### Find the biggest value in a field

Here is a quick query and retrieve the result. In this case we are finding the highest value for the id column.

```php
public function highestId() {
  $database = \Drupal::database();
  $connection = Database::getConnection();

  $query = $connection->select('donors', 'n');
  $query->addExpression('MAX(id)', 'id');
  $result = $query->execute();
  $highest_id = intval($result->fetchField());

  $str = "Highest id = $highest_id";
  $render_array['content'] = [
    '#type' => 'item',
    '#markup' => $str,
  ];

  return $render_array;
}
```

### SQL update query - example 1

This shows how to update a status field to the new value in `$status` when the uuid matches, the event is either update or add, and the status is new.

```php
public function setUpdateStatus(string $uuid, string $status) {
  $db_connection = \Drupal::database();
  $result = $db_connection->update('nocs_info')
    ->fields(['status' => $status])
    ->condition('uuid', $uuid)
    ->condition('event', ['UPDATE', 'ADD'], 'IN')
    ->condition('status', 'new')
    ->execute();

  return $result;
}
```

### SQL update query - example 2

```php
/**
 * Converts imported eventlog item/s from UPDATE to ADD by uuid.
 *
 * @param string $uuid
 *   UUID for events to convert from UPDATE to ADD.
 *
 * @return mixed
 *   Results of update query.
 */
public function convertUpdateToAddEvent(string $uuid) {
  $update_connection = \Drupal::database();
  $result = $update_connection->update('nocs_connect')
    ->fields([
      'event' => 'ADD',
    ])
    ->condition('uuid', $uuid)
    ->condition('event', 'UPDATE')
    ->condition('status', 'new')
    ->execute();

  return $result;
}
```

### SQL update query - example 3

This will update values in a table and return the number of rows
updated.

```php
//use Drupal\Core\Database\Database;

public function updateQuery1() {
  $database = \Drupal::database();
  $query_string = "Update {donors} set amount=amount+1 where id<=10 ";
  $affectedRows = $database->query($query_string,[],
    ['return' => Database::RETURN_AFFECTED]);
  $str = "Affected rows = $affectedRows";
  $render_array['content'] = [
    '#type' => 'item',
    '#markup' => $str,
  ];

  return $render_array;
}
```

Note. This will be deprecated in Drupal 11. See <https://api.drupal.org/api/drupal/core%21lib%21Drupal%21Core%21Database%21Statement.php/function/Statement%3A%3ArowCount/9.3.x> and <https://git.drupalcode.org/project/drupal/-/blob/9.5.x/core/lib/Drupal/Core/Database/Connection.php#L968>


### SQL insert

From
<https://www.drupal.org/docs/drupal-apis/database-api/insert-queries>

Which to use\? `$connection-\>insert()` or `$connection->query()` or what are the difference between `insert()` and `query()`\?

- `insert()` has each column specified as a separate entry in the fields
    array and the code can clean each column value. query() has an SQL
    string with no way of checking individual columns.

- If you use `query()` with placeholders, the code can check the column
    values but placeholders are just an option, there is no way to
    ensure your SQL does not contain values not passed through
    placeholders.

- `insert()` passes the request through a set of hooks to let other
    modules check and modify your requests. This is the right way to
    work with other modules.

- `query()` is slightly faster because it does not pass the request
    through the hooks. You might save processing time but your code will not let other modules help your code.

- `insert()` is more likely to work with other databases and future
    versions of Drupal.

### SQL Insert Query

```php
/**
 * @throws \Exception
 */
public function insert() {

  /** @var \Drupal\Core\Database\Connection $connection */
  $connection = \Drupal::service('database');

  //    $query = $connection->insert('donors', $options);

  // single insert.
  $result = $connection->insert('donors')
    ->fields([
      'name' => 'Singleton',
      'amount' => 1,
    ])
    ->execute();
  // Note. there is an auto-increment field so insert() returns  the value
  // for the new row in $result.
  $str = "Single insert returned auto-increment value of $result";

  // Multi-insert1.
  $result = $connection->insert('donors')
    ->fields(['name', 'amount',])
    ->values(['name' => 'Multiton1', 'amount' => 11,])
    ->values(['name' => 'Multiton1', 'amount' => 22,])
    ->execute();
  $str .= "<br/>Multi-insert1 added 2 rows";

  // Multi-insert2.
  $values = [
    ['name' => 'Multiton1', 'amount' => 111,],
    ['name' => 'Multiton2', 'amount' => 222,],
    ['name' => 'Multiton3', 'amount' => 333,],
    ['name' => 'Multiton4', 'amount' => 444,],
    ['name' => 'Multiton5', 'amount' => 555,],
  ];
  $query = $connection->insert('donors')
    ->fields(['name', 'amount',]);
  foreach ($values as $record) {
    $query->values($record);
  }
  $result = $query->execute();
  $str .= "<br/>Multi-insert2 added 5 rows";

  $render_array['content'] = [
    '#type' => 'item',
    '#markup' => $str,
  ];

  return $render_array;
}
```

More at [Insert Queries on drupal.org - updated Nov 2023](https://www.drupal.org/docs/drupal-apis/database-api/insert-queries)

### SQL Delete query

This will return the number of rows affected by the SQL delete query.

```php
use Drupal\Core\Database\Database;

public function deleteQuery2() {

  $database = \Drupal::database();
  $query_string = "Delete FROM {donors} where id>10 ";
  $affectedRows = $database->query($query_string,[],['return' => Database::RETURN_AFFECTED]);

  $str = "Affected rows = $affectedRows";
  $render_array['content'] = [
    '#type' => 'item',
    '#markup' => $str,
  ];

  return $render_array;
}
```

Note. This will be deprecated in Drupal 11. See <https://api.drupal.org/api/drupal/core%21lib%21Drupal%21Core%21Database%21Statement.php/function/Statement%3A%3ArowCount/9.3.x> also <https://git.drupalcode.org/project/drupal/-/blob/9.5.x/core/lib/Drupal/Core/Database/Connection.php#L968>.

### Paragraph static query example

In the `txg.theme` file this code digs into a table for a paragraph field and grabs the delta field value using a static query.

```php
function txg_preprocess_paragraph__simple_card(&$variables) {
  $card_parent = $variables['paragraph']->getParentEntity();
  if ($card_parent->bundle() == 'home_aof_card') {
    $variables['parent_delta'] = 0;
    $database = Database::getConnection();
    $result = $database->query("
      Select delta from node__field_para_aofs n where n.bundle = 'home_page' AND n.field_para_aofs_target_id = :target_id", [':target_id' => $card_parent->id()]
    );
    if ($result) {
      while ($row = $result->fetchAssoc()) {
        $variables['parent_delta'] = $row['delta'];
      }
    }
    $parent_label = $card_parent->field_ref_aof->entity->label();
    $parent_path = $card_parent->field_ref_aof->entity->toUrl()->toString();
    $variables['parent_label'] = $parent_label;
    $variables['parent_path'] = $parent_path;
  }
}
```




### Create a custom table for your module

If you need a custom database table (or two) for use in a custom module, you can use `hook_schema` in your `module.install` file. This will cause the table(s) to be created at module install time and **removed** at module uninstall time.

````php
function nocs_connect_schema() {
  $schema['nocs_connect'] = [
    'description' => 'Stores data from event log used to import/update content to site.',
    'fields' => [
      'id' => [
        'type' => 'int',
        'not null' => TRUE,
        'description' => 'Primary Key: Unique ID of event log event.',
      ],
      'uuid' => [
        'type' => 'varchar',
        'length' => 255,
        'not null' => TRUE,
        'description' => "Unique ID for content created or updated.",
      ],
      'nid' => [
        'type' => 'varchar',
        'length' => 255,
        'not null' => FALSE,
        'description' => "Unique ID for content created or updated.",
      ],
      'event' => [
        'type' => 'varchar',
        'length' => 255,
        'not null' => TRUE,
        'description' => 'Type of event e.g. UPDATE or ADD.',
      ],
      'created' => [
        'type' => 'int',
        'not null' => TRUE,
        'description' => 'Content creation date.',
      ],
      'updated' => [
        'type' => 'int',
        'not null' => TRUE,
        'description' => 'Content update date.',
      ],
      'type' => [
        'type' => 'varchar',
        'length' => 255,
        'not null' => TRUE,
        'description' => 'Type of content created or updated.',
      ],
      'version' => [
        'type' => 'int',
        'not null' => TRUE,
        'description' => 'Content version number.',
      ],
      'status' => [
        'type' => 'varchar',
        'length' => 255,
        'not null' => TRUE,
        'default' => 'NEW',
        'description' => 'Status of event log row - NEW, PROCESSING, COMPLETE, or ERROR.',
      ],
    ],
    'primary key' => ['id'],
    'indexes' => [
      'uuid' => ['uuid'],
      'event' => ['event'],
      'type' => ['type'],
    ],
  ];

  return $schema;
}
````

## Other Queries
These are useful little queries that you can paste into your SQL tools (e.g. SequelAce, SequelPro, PhpMyAdmin etc.) or use directly in MySQL using the command line.:

```sh
ddev drush sqlc

Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MySQL connection id is 134650
Server version: 5.7.42-0ubuntu0.18.04.1-log (Ubuntu)

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MySQL [db]> SELECT table_name, round(((data_length + index_length) / 1024 / 1024), 2) as SIZE_MB FROM information_schema.TABLES WHERE table_schema = DATABASE() ORDER BY SIZE_MB DESC LIMIT 10;
```

### List of top 10 biggest tables 

```sql
SELECT table_name, round(((data_length + index_length) / 1024 / 1024), 2) as SIZE_MB FROM information_schema.TABLES WHERE table_schema = DATABASE() ORDER BY SIZE_MB DESC LIMIT 10;
```
This outputs something like this:

```
+----------------------------------+---------+
| table_name                       | SIZE_MB |
+----------------------------------+---------+
| node_field_data                  |  499.25 |
| path_alias                       |  446.16 |
| search_api_db_default_index_text |  424.98 |
| node                             |  209.55 |
| cache_entity                     |  187.59 |
| search_api_db_default_index      |  186.80 |
| node_field_revision              |  164.80 |
| search_api_item                  |  152.67 |
| node_access                      |  146.98 |
| path_alias_revision              |  126.14 |
+----------------------------------+---------+
10 rows in set (0.141 sec)
```


More on [Stack Overflow](https://stackoverflow.com/questions/9620198/how-to-get-the-sizes-of-the-tables-of-a-mysql-database/42262936#42262936)

## Use the database abstraction layer to avoid SQL injection attacks

It is bad practice to concatenate data directly into SQL queries.

```php
// Bad practice - don't do it!.
\Database::getConnection()->query('SELECT foo FROM {table} t WHERE t.name = '. $_GET['user']);
```

**Good Practice:**

Use proper argument substitution. The database layer works on top of PHP PDO, and uses an array of named placeholders:

```php
\Database::getConnection()->query('SELECT foo FROM {table} t WHERE t.name = :name', [':name' => $_GET['user']]);
```

For a variable number of argument, use an array of arguments or use the select() method.  See examples of each below:

```php
$users = ['joe', 'poe', $_GET['user']];
\Database::getConnection()->query('SELECT f.bar FROM {foo} f WHERE f.bar IN (:users[])',  [':users[]' => $users]);
```

```php
$users = ['joe', 'poe', $_GET['user']];
$result = \Database::getConnection()->select('foo', 'f')
  ->fields('f', ['bar'])
  ->condition('f.bar', $users)
  ->execute();
```

When forming a `LIKE` query, make sure that you escape condition values to ensure they don't contain wildcard characters like `"%"``:

```php
db_select('table', 't')
  ->condition('t.field', '%_' . db_like($user), 'LIKE')
  ->execute();
```

Make sure that users cannot provide any operator to a query's condition. For example, this is unsafe:

```php
db_select('table', 't')
  ->condition('t.field', $user, $user_input)
  ->execute();
```
Instead, set a list of allowed operators and only allow users to use those.

`db_query`, `db_select`, and `db_like` were deprecated and removed from Drupal 9 - instead you should use a database connection object and call the query, select, and [escapeLike](https://api.drupal.org/api/drupal/core%21lib%21Drupal%21Core%21Database%21Connection.php/function/Connection%3A%3AescapeLike/9) methods on it (the parameters are the same).



## Viewing the MariaDB General Query Log

```sh
ddev mysql -u root -proot
```

```sh
SET global general_log = 1;

SET global log_output = 'file';

SET global general_log_file = '/home/selwyn/queries.txt';
```

Log into the database container and tail the log file:
```sh
ddev ssh -s db
tail -f queries.txt
```

To turn off the general query log:
```sh
SET global general_log = 0;
```

or use `ddev restart` to restart all the containers.

The queries.txt file will automatically be deleted when the container is restarted.

Thanks to [Dries Buytart\'s Effortless inspecting of Drupal database queries article for this useful tip.](https://dri.es/effortless-inspecting-of-drupal-database-queries)


## Accessing a custom table

From the [XMLSitemap module](https://www.drupal.org/project/xmlsitemap) 

This example from `web/modules/contrib/xmlsitemap/src/XmlSitemapLinkStorage.php` accesses the `xmlsitemap` table and runs a quick query to see if the status or access fields are being changed. If they are, it sets a flag to regenerate the sitemap. The `addExpression('1')` method is used to simplify the query, and the `range(0, 1)` method is used to limit the query to the first row.  This code could definitely have been written more clearly but it does show how to run a query on a table.

```php
  public function checkChangedLinks(array $conditions = [], array $updates = [], $flag = FALSE) {
    // If we are changing status or access, check for negative current values.
    $conditions['status'] = (!empty($updates['status']) && empty($conditions['status'])) ? 0 : 1;
    $conditions['access'] = (!empty($updates['access']) && empty($conditions['access'])) ? 0 : 1;

    $query = $this->connection->select('xmlsitemap');
    $query->addExpression('1');
    foreach ($conditions as $field => $value) {
      $operator = is_array($value) ? 'IN' : '=';
      $query->condition($field, $value, $operator);
    }
    $query->range(0, 1);
    $changed = $query->execute()->fetchField();

    if ($changed && $flag) {
      $this->state->set('xmlsitemap_regenerate_needed', TRUE);
    }

    return $changed;
  }
```

[Link to this function's source code](https://git.drupalcode.org/project/xmlsitemap/-/blob/8.x-1.x/src/XmlSitemapLinkStorage.php?ref_type=heads#L283)


This code snippet from `web/modules/contrib/xmlsitemap/src/XmlSitemapLinkStorage.php` shows a query that uses `queryRange()`:

```php
  /**
   * {@inheritdoc}
   */
  public function checkChangedLink(array $link, array $original_link = NULL, $flag = FALSE) {
    $changed = FALSE;

    if ($original_link === NULL) {
      // Load only the fields necessary for data to be changed in the sitemap.
      $original_link = $this->connection->queryRange("SELECT loc, access, status, lastmod, priority, changefreq, changecount, language FROM {xmlsitemap} WHERE type = :type AND id = :id", 0, 1, [':type' => $link['type'], ':id' => $link['id']])->fetchAssoc();
    }
    ...

```
[Link to this function's source](https://git.drupalcode.org/project/xmlsitemap/-/blob/8.x-1.x/src/XmlSitemapLinkStorage.php?ref_type=heads#L262)



The following snippet shows an example of using the `merge` command.  This will add or update an existing link into the `xmlsitemap` table. In the `web/modules/contrib/xmlsitemap/src/XmlSitemapLinkStorage.php` file, the following code is used to write to the `xmlsitemap` table. Browse the [source code here](https://git.drupalcode.org/project/xmlsitemap/-/blob/8.x-1.x/src/XmlSitemapLinkStorage.php?ref_type=heads#L212).

```php
    $queryStatus = $this->connection->merge('xmlsitemap')
      ->keys([
        'type' => $link['type'],
        'id' => $link['id'],
        'language' => $link['language'],
      ])
      ->fields([
        'loc' => $link['loc'],
        'subtype' => $link['subtype'],
        'access' => (int) $link['access'],
        'status' => (int) $link['status'],
        'status_override' => $link['status_override'],
        'lastmod' => $link['lastmod'],
        'priority' => $link['priority'],
        'priority_override' => $link['priority_override'],
        'changefreq' => $link['changefreq'],
        'changecount' => $link['changecount'],
      ])
      ->execute();
```


You can [browse the source for the entire module here.](https://git.drupalcode.org/project/xmlsitemap)


## Reference

- [API documentation for query Condition class](https://api.drupal.org/api/drupal/core%21lib%21Drupal%21Core%21Database%21Query%21Condition.php/class/Condition/9.3.x)
- [API documentation for QueryInterface::condition](https://api.drupal.org/api/drupal/core%21lib%21Drupal%21Core%21Entity%21Query%21QueryInterface.php/function/QueryInterface%3A%3Acondition/8.6.x)
- [Entity query cheat sheet](https://www.metaltoad.com/blog/drupal-8-entity-api-cheat-sheet)
- [Static queries](https://www.drupal.org/docs/drupal-apis/database-api/static-queries)
- [Dynamic Queries](https://www.drupal.org/docs/8/api/database-api/dynamic-queries/introduction-to-dynamic-queries)
- [Insert Queries](https://www.drupal.org/docs/drupal-apis/database-api/insert-queries)
- [Querying date fields from 2018](https://blog.werk21.de/en/2018/02/05/date-range-fields-and-entity-query-update)
- [Querying date fields](https://drupal.stackexchange.com/questions/198324/how-to-do-a-date-range-entityquery-with-a-date-only-field-in-drupal-8)
