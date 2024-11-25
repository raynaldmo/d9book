---
title: General
---

# General

![views](https://api.visitor.plantree.me/visitor-badge/pv?label=views&color=informational&namespace=d9book&key=general.md)

## Get the current user

Note this will not get the user entity, but rather a user proxy with basic info but no fields or entity-specific data.

```php
$user = \Drupal::currentUser();
```

```php
$user = \Drupal\user\Entity\User::load(\Drupal::currentUser()->id());
```

Or

```php
use \Drupal\user\Entity\User;
$user = User::load(\Drupal::currentUser()->id());
```

## Get the logged in user name and email

```php
$username = \Drupal::currentUser()->getAccountName();
```

or

```php
$account_proxy = \Drupal::currentUser();
//$account = $account_proxy->getAccount();

// load user entity
$user = User::load($account_proxy->id());

$user = User::load(\Drupal::currentUser()->id());
$name = $user->get('name')->value;
```

Email

```php
$email = \Drupal::currentUser()->getEmail();
```

or

```php
$user = User::load(\Drupal::currentUser()->id());
$email = $user->get('mail')->value;
```

## Get the current Path

`\Drupal::service('path.current')->getPath()` returns the current relative path. For node pages, the return value will be in the form \"/node/32\" For taxonomy \"taxonomy/term/5\", for user \"user/2\" if it exists otherwise it will return the current request URI.

```php
$current_path  = \Drupal::service('path.current')->getPath();
// Get the alias (i.e. if the user entered node/123, this will return e.g. /bicycles/super-cool-one)
$alias = \Drupal::service('path_alias.manager')->getAliasByPath($current_path);

// Get path with query string e.g. /abc/def/123?a=fred.
$current_path_and_alias = \Drupal::request()->getRequestUri();

// Get path e.g. /abc/def/123
$current_path = Url::fromRoute('<current>')->toString();
```

[Lots more on the Drupal Stackexchange](https://drupal.stackexchange.com/questions/106103/how-do-i-get-the-current-path-alias-or-path)

## Check if you are on the Front page

```php
$is_front = \Drupal::service('path.matcher')->isFrontPage();
```

The above statement will return either TRUE or FALSE. TRUE means you are on the front page.

## Check if the site is in system maintenance mode

```php
$is_maint_mode = \Drupal::state()->get('system.maintenance_mode');
```

## Retrieve query, get or post parameters 

For `get` variables use:
```php
$query = \Drupal::request()->query->get('name');
```

For `post` variables use:

```php
$name = \Drupal::request()->request->get('name');
```

For all items in a `get`:

```php
$query = \Drupal::request()->query->all();
$search_term = $query['query'];
$collection = $query['collection'];
```

::: tip Note
Drupal will cache requests so render arrays need cache contexts specified correctly in order to successfully retrieve those parameters. See [Caching](caching#set-cache-context-correctly-when-retrieving-query-get-or-post-parameters)
:::

## Convert TranslatableMarkup to a string

To convert a TranslatableMarkup object to a string, use either the `render()` or __toString() method. This example shows the values array with an element 'save' which is a TranslatableMarkup object.  These will return `Save Citation`.

![TranslatableMarkup](/images/translatable-markup.png)

```php
$values['save']->render();
// Or.
$values['save']->__toString();
```


## Get Node URL alias or Taxonomy Alias by Node id or Term ID

Sometimes we need a relative path and sometimes we need an absolute path. There is an \$options parameter in the fromRoute() function where specify which you need.

Parameters:

- absolute true will return absolute path.
- absolute false will return relative path.

Returns the node alias. Note. If a nice url is not set using pathauto, you get `/node/1234`

```php
use Drupal\Core\Url;
$options = ['absolute' => true];  //false will return relative path.

$url = Url::fromRoute('entity.node.canonical', ['node' => 1234], $options);
$url = $url->toString(); // make a string

// OR

$node_path = "/node/1";
$alias = \Drupal::service('path_alias.manager')->getAliasByPath($node_path);

// OR

$current_path = \Drupal::service('path.current')->getPath();
```

To get the full path with the host etc. this returns: https://ddev93.ddev.site/node/1

```php
$host = \Drupal::request()->getSchemeAndHttpHost();
$url = \Drupal\Core\Url::fromRoute('entity.node.canonical',['node'=>$lab_home_nid]);
$url_alias = $url->toString();
$full_url = $host . $url->toString();
```

You can get the hostname, e.g. \"drupal8.local\", directly from the
getHost() request with:

```php
$host = \Drupal::request()->getHost();
```

## Taxonomy alias

Return taxonomy alias

```php
$options = ['absolute' => true];  //false will return relative path.
$url = Url::fromRoute('entity.taxonomy_term.canonical', ['taxonomy_term' => 1234], $options);
```

## Get current Path

For node pages this will return `node/{node id}`, for taxonomy `taxonomy/term/{term id}`, for user `user/{user id}` if exists otherwise it will return the current request URI.

```php
$currentPath  = \Drupal::service('path.current')->getPath();
```

## Get current nid, node type and title

There are two ways to retrieve the current node -- via the request or the route

```php
$node = \Drupal::request()->attributes->get('node');
$nid = $node->id();
```

OR

```php
$node = \Drupal::routeMatch()->getParameter('node');
if ($node instanceof \Drupal\node\NodeInterface) {
  // You can get nid and anything else you need from the node object.
  $nid = $node->id();
  $nodeType = $node->bundle();
  $nodeTitle = $node->getTitle();
}
```

If you need to use the node object in `hook_preprocess_page` on the preview page, you will need to use the `node_preview` parameter, instead of the `node` parameter:

```php
function mymodule_preprocess_page(&$vars) {

  $route_name = \Drupal::routeMatch()->getRouteName();

  if ($route_name == 'entity.node.canonical') {
    $node = \Drupal::routeMatch()->getParameter('node');
  }
  elseif ($route_name == 'entity.node.preview') {
    $node = \Drupal::routeMatch()->getParameter('node_preview');
  }
```

And from <https://drupal.stackexchange.com/questions/145823/how-do-i-get-the-current-node-id> when you are using or creating a custom block then you have to follow this code to get current node id. Not sure if it is correct.

```php
use Drupal\Core\Cache\Cache;

$node = \Drupal::routeMatch()->getParameter('node');
if ($node instanceof \Drupal\node\NodeInterface) {
  $nid = $node->id();
}

// for cache
public function getCacheTags() {
  //With this when your node changes your block will rebuild
  if ($node = \Drupal::routeMatch()->getParameter('node')) {
    //if there is node add its cachetag
    return Cache::mergeTags(parent::getCacheTags(), ['node:' . $node->id()]);
  }
  else {
    //Return default tags instead.
    return parent::getCacheTags();
  }
}

public function getCacheContexts() {
  //if you depend on \Drupal::routeMatch()
  //you must set context of this block with 'route' context tag.
  //Every new route this block will rebuild
  return Cache::mergeContexts(parent::getCacheContexts(), ['route']);
}
```

## How to check whether a module is installed or not

```php
$moduleHandler = \Drupal::service('module_handler');
$module_name = “views”;
if ($moduleHandler->moduleExists($module_name)) {
  echo "$module_name installed";
}
else {
  echo "$module_name not installed";
}
```

## Get current Route name

Routes are in the form: view.files_browser.page_1, test.example or test.settings_form

E.g from test.routing.yml

```yaml
test.example:
  path: '/test/example'
  defaults:
    _title: 'Example'
    _controller: '\Drupal\test\Controller\TestController::build'
  requirements:
    _permission: 'access content'

test.settings_form:
  path: '/admin/config/system/test'
  defaults:
    _title: 'Test settings'
    _form: 'Drupal\test\Form\SettingsForm'
  requirements:
    _permission: 'administer test configuration'
```

This will return Drupal route. It returns entity.node.canonical for the nodes, system.404 for the 404 pages, entity.taxonomy_term.canonical for the taxonomy pages, entity.user.canonical for the users and custom route name that we define
in modulename.routing.yml file.

```php
$current_route = \Drupal::routeMatch()->getRouteName();
```

## Get the current page title

You can use this in a controller, to return the current page title.

```php
$request = \Drupal::request();
  if ($route = $request->attributes->get(\Symfony\Cmf\Component\Routing\RouteObjectInterface::ROUTE_OBJECT)) {
$title = \Drupal::service('title_resolver')->getTitle($request, $route);}
```

## Get the current user

Note this will not get the user entity, but rather a user proxy with basic info but no fields or entity-specific data.

```php
$user = \Drupal::currentUser();
```

To get the user entity, use this which gets the user service (`\Drupal::currentUser()`), gets the uid (`->id()`), then calls `load()` to load the real user object.

```php
$user = \Drupal\user\Entity\User::load(\Drupal::currentUser()->id());
```

Or

```php
use \Drupal\user\Entity\User;
$user = User::load(\Drupal::currentUser()->id());
```

## Check if you are on the Front page

This will return true for the front page otherwise false.

```php
$is_front = \Drupal::service('path.matcher')->isFrontPage();
```



## Retrieve URL argument parameters

You can extract the url arguments with

```php
$current_path = \Drupal::service('path.current')->getPath();
$path_args = explode('/', $current_path);
$term_name = $path_args[3];
```

For https://txg.ddev.site/newsroom/search/?country=1206

![Variables display in PHPStorm debug pane](/images/image1-general.png)

## Get Current Language in a constructor

In dev1 - `/modules/custom/iai_wea/src/Plugin/rest/resource/WEAResource.php` we create the WeaResource class and using dependency injection, get the LanguageManagerInterface service passed in, then we call `getgetCurrentLanguage()`. This allows us to later retrieve the node

```php
class WEAResource extends ResourceBase {

  /**
   * @var \Drupal\Core\Language\Language
   */
  protected $currentLanguage;

  /**
   * WEAResource constructor.
   *
   * @param array $configuration
   * @param string $plugin_id
   * @param mixed $plugin_definition
   * @param array $serializer_formats
   * @param \Psr\Log\LoggerInterface $logger
   * @param \Drupal\Core\Language\LanguageManagerInterface $language_manager
   */
  public function __construct(array $configuration, string $plugin_id, mixed $plugin_definition, array $serializer_formats, \Psr\Log\LoggerInterface $logger, LanguageManagerInterface $language_manager) {
    parent::__construct($configuration, $plugin_id, $plugin_definition, $serializer_formats, $logger);
    $this->currentLanguage = $language_manager->getCurrentLanguage();
  }
}
```

Later in the class, we can retrieve the correct language version of the node:

```php
public function get($id) {
  if ($node = Node::load($id)) {
    $translatedNode = $node->getTranslation($this->currentLanguage->getId());
```

Of course, you can also get the language statically by using:

```php
Global $language = Drupal::languageManager()->getLanguage(Language:TYPE_INTERFACE)
```

This is part of the packt publishing Mastering Drupal 8 module development video series: https://www.packtpub.com/product/mastering-drupal-8-development-video/9781787124493

> [!NOTE]
> To test this in `modules/custom/pseudo_client/get/`
> ```
> php -S localhost:8888
> ```
> and put this in a browser:
> ```
> http://localhost:8888/get_item_from_drupal_core.php?domain=dev1&item=2716
> ```
> or
> ```
> http://localhost:8888/get_items_from_custom_code.php?domain=dev1
> ```
> OR just put this in browser without running `php -S`:
> ```
> http://dev1/iai_wea/actions/2716?_format=json
> ```

## Add a variable to any page on the site

In the .theme file of the theme, add a `hook_preprocess_page` function
like in `themes/custom/dprime/dprime.theme`:

```php
function dprime_preprocess_page(&$variables) {
  $language_interface = \Drupal::languageManager()->getCurrentLanguage();

  $variables['footer_address1'] = [
    '#type'=>'markup',
    '#markup'=>'123 Disk Drive, Sector 439',
  ];
  $variables['footer_address2'] = [
    '#type'=>'markup',
    '#markup'=>'Austin, Texas 78759',
  ];
```

Then in the template file e.g.
`themes/custom/dprime/templates/partials/footer.html.twig`

```twig
<div class="cell xlarge-3 medium-4">
  <address>
    {{ footer_address1 }}<br />
    {{ footer_address2 }}<br />
    Campus mail code: D9000<br />
    <a href="mailto:abc@example.com">abc@example.com </a>
  </address>
</div>
```

## Add a variable to be rendered in a node.

From dev1 custom theme burger_burgler.

Here two vars `stock_field` and `my_custom_field` are added and will be rendered by a normal node twig file. The function hook_preprocess_node is in the .theme file at `themes/custom/burger_burgler/burger_burgler.theme`.

```php
function burger_burgler_preprocess_node(&$variables) {

  $variables['content']['stock_field'] = [
    '#type'=>'markup',
    '#markup'=>'stock field here',
  ];

  $variables['content']['my_custom_field'] = [
    '#type' => 'markup',
    '#markup' => 'Hello - custom field here',
  ];
}
```

If you've tweaked your node twig template, you'll need to reference like
this:

```twig
<div class="stock-field-class">
  {{ content['stock_field'] }}
</div>
```

Note. You can always just add a variable like

```php
$variables['abc'] = 'hello';
```

which can be referenced in the template as

```twig
{{ abc }}
```

or

```twig
{{ kint(abc) }}
```

## Add a bunch of variables to be rendered in a node

You can easily grab the node from the \$variables with:

```php
$node = $variables['node'];
```

Then to access a field in the node, you can just specify them by:

```php
$node->field_ref_aof
$node->field_ref_topic
```

Here we grab a bunch of variables, cycles through them (for multi-value fields, which most of them are and build an array that can be easily rendered by twig:

From: themes/custom/txg/txg.theme

```php
function txg_preprocess_node(&$variables) {
  $view_mode = $variables['view_mode']; // Retrieve view mode
  $allowed_view_modes = ['full']; // Array of allowed view modes (for performance so as to not execute on unneeded nodes)
  $node = $variables['node'];
  if (($node->getType() == 'news_story') && ($view_mode == 'full')) {
    $aofs = _txg_multival_ref_data($node->field_ref_aof, 'aof', 'target_id');
    $units = _txg_multival_ref_data($node->field_ref_unit, 'unit', 'target_id');
    $audiences = _txg_multival_ref_data($node->field_ref_audience, 'audience', 'target_id');
    $collections = _txg_multival_ref_data($node->field_ref_program_collection, 'collection', 'target_id');
    $topics = _txg_multival_ref_data($node->field_ref_topic,  'topic', 'target_id', 'taxonomy');
    $continents = _txg_multival_ref_data($node->field_continent, 'continent', 'value', 'list');
    $countries = _txg_multival_ref_data($node->field_ref_country, 'country', 'target_id');
    $related_news_items = array_merge($topics, $aofs, $units, $audiences, $collections, $continents,  $countries);
    $variables['related_news_items'] = $related_news_items;
  }
}

/**
 * Returns array of data for multivalue node reference fields
 * ref_field = entity reference field
 * param_name = parameter name to be passed as get value
 * value_type = indicates which field to retrieve from database
 * field_ref_type = variable to determine type of reference field
 * field_term_category =
 */
function _txg_multival_ref_data($ref_field, $param_name, $value_type, $field_ref_type = 'node') {
   $values = [];
   foreach($ref_field as $ref) {
     if ($field_ref_type == 'taxonomy') {
       $term = Drupal::entityTypeManager()->getStorage('taxonomy_term')->load($ref->$value_type);
       $title = $term->getName();
     }
     else {
       $title = $value_type == 'value' ? $ref->$value_type : $ref->entity->title->value;
     }
     $id = $ref->$value_type;
     $values[] = [
       'title' => $title,
       'id' => str_replace(' ', '+', $id),
       'param_name' => $param_name,
     ];
   }
  return $values;
}
```

## Grabbing entity reference fields in hook_preprocess_node for injection into the twig template

You can easily pull in referenced fields by referring to them as

```php
$node->field_sf_contract_ref->entity->field_how_to_order->value;
```

Where `field_sf_contract_ref` is the reference field, which points to an
entity which has a field called `field_how_to_order`. Then we can jam it
into the `$variables` array and refer to it in the twig template as <code v-pre>{{
how_to_order }}</code>

From `web/themes/custom/dirt_bootstrap/dirt_bootstrap.theme`

In `function dirt_bootstrap_preprocess_node(&$variables)`

```php
if ($type === 'contract') {
  if ($view_mode === 'full') {
    $how_to_order_lookup = $node->field_sf_contract_ref->entity->field_how_to_order_lookup->value;
    $variables['how_to_order_lookup'] = $how_to_order_lookup;
    $contract_type = $node->get('field_contract_type')->value;
    if ($how_to_order_lookup === "Custom Text") {
      if ($contract_type === "DIRT") {
        $variables['how_to_order'] = $node->field_sf_contract_ref->entity->field_how_to_order->value;
      }
      else {
        $variables['how_to_order'] = $node->field_sf_contract_ref->entity->field_how_to_order_custom->value;
      }
    }
  }
}
```

## Render a list created in the template_preprocess_node()

Here we create a list in the preprocess_node custom theme burger_burgler):

```php
function burger_burgler_preprocess_node(&$variables) {

  $burger_list = [
    ['name' => 'Cheesburger'],
    ['name' => 'Mushroom Swissburger'],
    ['name' => 'Jalapeno bugburger'],
  ];
  $variables['burgers'] = $burger_list;

}
```

and render it in the twig template `node--article--full.html.twig`

```twig
<ol>
  {% for burger in burgers %}
    <li> {{ burger['name'] }} </li>
  {% endfor %}
</ol>
```

## Indexing paragraphs so you can theme the first one

Posted on
<https://www.drupal.org/project/paragraphs/issues/2881460#comment-13291215>

From themes/custom/dprime/dprime.theme

Add this to the theme:

```php
/**
 * Implements hook_preprocess_field.
 *
 * Provides an index for these fields referenced as {{ paragraph.index }}
 * in twig template.
 *
 * @param $variables
 */
function dprime_preprocess_field(&$variables) {
  if($variables['field_name'] == 'field_video_accordions'){
    foreach($variables['items'] as $idx => $item) {
      $variables['items'][$idx]['content']['#paragraph']->index = $idx;
    }
  }
}
```

`field_video_accordions` is the name of the field that holds the paragraph you want to count.

In the twig template for that paragraph, you can use the value `paragraph.index` as in:

```twig
{% if paragraph.index == 0 %}
  <li class="accordion-item is-active" data-accordion-item="">
{% else %}
  <li class="accordion-item" data-accordion-item="">
{% endif %}
```

## Add meta tags using template_preprocess_html

Also covered at
<https://drupal.stackexchange.com/questions/217880/how-do-i-add-a-meta-tag-in-inside-the-head-tag>

If you need to make changes to the `<head>` element, the `hook_preprocess_html` is the place to do it in the `.theme` file. Here we check to see that the content type is contract and then we create a fake array of meta tags and jam them into the `$variables['page']['#attached']['html_head']` element. They are then rendered on the page.

```php
/**
 * Implements hook_preprocess_html().
 */
function dir_bootstrap_preprocess_html(&$variables) {

  $node = \Drupal::routeMatch()->getParameter('node');
  if ($node instanceof \Drupal\node\NodeInterface) {
    if ($node->getType() == 'contract') {

      $brand_meta_tag[] = [[
        '#tag' => 'meta',
        '#attributes' => [
          'name' => 'brand',
          'content' => 'Dell',
        ]],
        'Dell',
      ];
..

$variables['page']['#attached']['html_head'][] = $brand_meta_tag;
```

Note that the extra "Dell" low down in the array appears to be a
description of some kind -- it isn't rendered. If you don't include the
second "Dell" you could rather use

```php
$page['#attached']['html_head'][] = [$description, 'description'];
```

For multiple tags, I had to do this version:

```php
$brand_meta_tags[] = [[
  '#tag' => 'meta',
  '#attributes' => [
    'name' => 'brand',
    'content' => 'Dell',
  ]],
  'Dell',
];
$brand_meta_tags[] = [[
  '#tag' => 'meta',
  '#attributes' => [
    'name' => 'brand',
    'content' => 'Apple',
  ]],
  'Apple',
];

foreach ($brand_meta_tags as $brand_meta_tag) {
  $variables['page']['#attached']['html_head'][] = $brand_meta_tag;
}
```

And here I do a query and build some new meta tags from `themes/custom/dirt_bootstrap/dirt_bootstrap.theme`.

```php
$brand_meta_tags = [];
$contract_id = $node->field_contract_id->value;
if ($contract_id) {

  //Lookup dirt store brand records with this contract id.
  $storage = \Drupal::entityTypeManager()->getStorage('node');
  $query = \Drupal::entityQuery('node')
    ->condition('type', 'sf_store_brands')
    ->condition('status', 1)
    ->condition('field_contract_id', $contract_id);
  $nids = $query->execute();
  foreach ($nids as $nid) {
    $store_brand_node = Node::load($nid);
    $brand = $store_brand_node->field_brand->value;
    if ($brand) {
      $brand_meta_tags[] = [[
        '#tag' => 'meta',
        '#attributes' => [
          'name' => 'brand',
          'content' => $brand,
        ]],
        $brand,
      ];
    }
  }
  foreach ($brand_meta_tags as $brand_meta_tag) {
    $variables['page']['#attached']['html_head'][] = $brand_meta_tag;
  }
}
```

## Decoding URL encoded strings

Encoded strings have all non-alphanumeric characters except -\_. replaced with a percent (%) sign followed by two hex digits and spaces encoded as plus (+) signs. This is the same way that the posted data from a WWW form is encoded, and also the same way as in `application/x-www-form-urlencoded` media type.

When you see strings like `%20` or `%E2` and you need plaintext, use `urldecode()`.

```php
echo urldecode("threatgeek/2016/05/welcome-jungle-tips-staying-secure-when-you%E2%80%99re-road") . "\n";
echo urldecode('We%27re%20proud%20to%20introduce%20the%20Amazing') . "\n";
//$str = "threatgeek/2016/05/welcome-jungle-tips-staying-secure-when-you%E2%80%99re-road";
//echo htmlspecialchars_decode($str) . "\n";
```

returns:

```
threatgeek/2016/05/welcome-jungle-tips-staying-secure-when-you’re-road
We're proud to introduce the Amazing
```

Encoding looks like this:

```php
echo urlencode("threatgeek/2016/05/welcome-jungle-tips-staying-secure-when-you're-road") . "\n";
```

returns:

```
threatgeek%2F2016%2F05%2Fwelcome-jungle-tips-staying-secure-when-you%E2%80%99re-road
```

[More at php.net](https://www.php.net/manual/en/function.urlencode.php)

## Remote media entities

For this project, I had to figure out a way to make media entities that really were remote images. i.e. the API provided images but we didn't want to store them in Drupal

I started by looking at <https://www.drupal.org/sandbox/nickhope/3001154> which was based on <https://www.drupal.org/project/media_entity_flickr>.

I tweaked the nickhope module (media_entity_remote_file) so it worked but it had some trouble with image styles and thumbnails

A good solution (thanks to Hugo) is:

https://www.drupal.org/project/remote_stream_wrapper_widget

https://www.drupal.org/project/remote_stream_wrapper

Hugo suggests using this to do migration:

```php
$uri = 'http://example.com/somefile.mp3';
$file = File::Create(['uri' => $uri]);
$file->save();
$node->field_file->setValue(['target_id' => $file->id()]);
$node->save();
```

There was no documentation so I [added some](https://www.drupal.org/project/remote_stream_wrapper/issues/2875444#comment-12881516).

## Deprecated functions like drupal_set_message

:::tip Note
`drupal_set_message()` has been removed from the codebase, so you should use `messenger()` but you can also use `dsm()` which is provided by the [devel](https://www.drupal.org/project/devel) contrib module. This is useful when working through a problem if you want to display a message on a site during debugging.
:::

From <https://github.com/mglaman/drupal-check/wiki/Deprecation-Error-Solutions>

Before

```php
drupal_set_message($message, $type, $repeat);
```

After

```php
\Drupal::messenger()->addMessage($message, $type, $repeat);
```

[Read more](https://www.drupal.org/node/2774931).

## Block excessive crawling of Drupal Views or search results with .htaccess

[From Block excessive crawling of Drupal Views or search results on Acquia.com - Jan 2024](https://acquia.my.site.com/s/article/4408794498199-Block-excessive-crawling-of-Drupal-Views-or-search-results)

PLACE THIS BLOCK directly after the "RewriteEngine on" line in your `docroot/.htaccess` or `web/.htaccess` file.


Sometimes, robot webcrawlers (like Bing, Huwaei Cloud, Yandex, Semrush, etc.) can attempt to crawl a Drupal View's search results pages, and could also be following links to each of the view's filtering options. This places extra load on your site. Additionally, the crawling (even if done by legitimate search engines) may not be increasing your site's visibility to users of search engines.

Therefore, we suggest blocking or re-routing this traffic to reduce resource consumption at the Acquia platform, avoid overages to your Acquia entitlements (for Acquia Search, Views & Visits, etc.), and to generally help your site perform better.

```
# EXAMPLE ROBOT BLOCKING CODE for Search pages or views.
# From: https://support-acquia.force.com/s/article/4408794498199-Block-excessive-crawling-of-Drupal-Views-or-search-results
#   NOTE: May need editing depending on your use case(s).
#
# INSTRUCTIONS:
# PLACE THIS BLOCK directly after the "RewriteEngine on" line
#   in your docroot/.htaccess file.
#
# This will block some known robots/crawlers on URLs when query arguments are present.
#   DOES allow basic URLs like /news/feed, /node/1 or /rss, etc.
#   BLOCKS only when search arguments are present like
#     /news/feed?search=XXX or /rss?page=21.
# Note: You can add more conditions if needed.
#   For example, to only block on URLs that begin with '/search', add this
#   line before the RewriteRule:
#     RewriteCond %{REQUEST_URI} ^/search
#
RewriteCond %{QUERY_STRING} .
RewriteCond %{HTTP_USER_AGENT} "11A465|AddThis.com|AdsBot-Google|Ahrefs|alexa site audit|Amazonbot|Amazon-Route53-Health-Check-Service|ApacheBench|AppDynamics|Applebot|ArchiveBot|AspiegelBot|Baiduspider|bingbot|BLEXBot|BluechipBacklinks|Buck|Bytespider|CCBot|check_http|cludo.com bot|contentkingapp|Cookiebot|CopperEgg|crawler4j|Csnibot|Curebot|curl|Daum|Datadog Agent|DataForSeoBot|Detectify|DotBot|DuckDuckBot|facebookexternalhit|Faraday|FeedFetcher-Google|feedonomics|Funnelback|GAChecker|Grapeshot|gobuster|gocolly|Googlebot|GoogleStackdriverMonitoring|Go-http-client|GuzzleHttp|HeadlessChrome|heritrix|hokifyBot|HTTrack|HubSpot Crawler|ICC-Crawler|Imperva|IonCrawl|KauaiBot|Kinza|LieBaoFast|Linespider|Linguee|LinkChecker|LinkedInBot|LinuxGetUrl|LMY47V|MacOutlook|Magus Bot|Mail.RU_Bot|MauiBot|Mb2345Browser|MegaIndex|Microsoft Office|Microsoft Outlook|Microsoft Word|MicroMessenger|mindbreeze-crawler|mirrorweb.com|MJ12bot|monitoring-plugins|Monsidobot|MQQBrowser|msnbot|MSOffice|MTRobot|nagios-plugins|nettle|Neevabot|newspaper|Nuclei|OnCrawl|Orbbot|PageFreezer|panscient.com|PetalBot|Pingdom.com|Pinterestbot|PiplBot|python-requests|Qwantify|Re-re Studio|Riddler|rogerbot|RustBot|Scrapy|Screaming Frog|Search365bot|SearchBlox|SearchmetricsBot|searchunify|Seekport|SemanticScholarBot|SemrushBot|SEOkicks|seoscanners|serpstatbot|SessionCam|SeznamBot|Site24x7|siteimprove|Siteimprove|SiteSucker|SkypeRoom|Sogou web spider|special_archiver|SpiderLing|StatusCake|Synack|Turnitin|trendictionbot|trendkite-akashic-crawler|UCBrowser|Uptime|UptimeRobot|UT-Dorkbot|weborama-fetcher|WhiteHat Security|Wget|www.loc.gov|Vagabondo|VelenPublicWebCrawler|Yeti|Veracode Security Scan|YandexBot|YandexImages|YisouSpider|Zabbix|ZoominfoBot" [NC]
RewriteRule ^.* - [F,L]
```

Alternatively, you can make these changes to your `docroot/robots.txt`  or `web/robots.txt` file:
```
# Do not index nor follow links that have a query string
# (e.g. /search?page=123  or /search?size=small&color=red)
User-agent: *
Disallow: /*?

# If your views or search pages use a module to convert facets/filters 
# to clean URLs (e.g. /search/page/123  or /search/size/small)
# you can try disallowing the search page's URL
User-agent: *
Disallow: /search*
```


## Using the file_system service to count files

```php
// In the create method get the file_system service.
$form->fileSystem = $container->get('file_system');

// Search filesystem recursively get all .PHP files from Drupal's core folder.
$files_count = count($this->fileSystem->scanDirectory('core', '/.php/'));

```
See this in action in the [examples module](https://www.drupal.org/project/examples) in `CacheExampleForm.php`.


## Multiple authors on a node

Thanks to Mike Anello of [DrupalEasy for this useful solution.](https://www.drupaleasy.com/blogs/ultimike/2023/02/method-utilizing-multiple-authors-single-drupal-node)

**TL;DR**
Using the [Access by Reference module](https://www.drupal.org/project/access_by_ref) allows you to specify additional authors via several methods. Mike prefers using a a reference field for this purpose. He also wanted the "Additional authors" field to be listed in the "Authoring information" accordion of the standard Drupal node add/edit form. He created a very small custom Drupal module named `multiauthor` that implements a single Drupal hook:

```php
/**
 * Implements hook_form_alter().
 */
function multiauthor_form_alter(array &$form, FormStateInterface $form_state, string $form_id): void {
  if (in_array($form_id, ['node_page_edit_form', 'node_page_form'])) {
    $form['field_additional_authors']['#group'] = 'author';
  }
}
```

This hook alters the Basic page add and edit forms, setting my custom "Additional author" field (field_additional_authors) to the "author" group in the "Additional authors" accordion. Users added to the `Additional authors` field get the same read, update, and delete permissions at the owner of the node.

## Calculating, displaying and logging elapsed time

To record how long something takes in Drupal, use the `Timer` utility class. In the example below, this info is also logged to the `watchdog` log.

In your config or `settings.local.php` (to temporarily override that value) you can enable or disable the timer with:

```php
$config['tea_teks_srp.testing']['display_elapsed_time'] = TRUE;
$config['tea_teks_srp.testing']['log_elapsed_time'] = TRUE;
```

This example uses a form so this is the constructor:

```php
class SrpVoteOnCitationForm extends FormBase {

  protected bool $displayElapsedTime = FALSE;
  protected bool $logElapsedTime = FALSE;

  public function __construct(VotingProcessorInterface $votingProcessor) {
    $this->displayElapsedTime = \Drupal::config('tea_teks_srp.testing')->get('display_elapsed_time');
    $this->displayElapsedTime = \Drupal::config('tea_teks_srp.testing')->get('log_elapsed_time');

  }
```

In `submitForm()` we start the timer, do some work and then stop the timer and report the result like this:

```php
public function submitForm(array &$form, FormStateInterface $form_state) {
  $user_id = \Drupal::currentUser()->id();
  Timer::start('vote:voter_id:' . $user_id);

  $current_path = \Drupal::service('path.current')->getPath();
  if ($this->displayElapsedTime) {
    $msg = 'Voter: ' . number_format($user_id). ' Citation: ' . number_format($citation_nid) .' Vote: ' . strtolower($voting_action) . ' Url:' . $current_path ;
    \Drupal::messenger()->addMessage($msg);
  }
  if ($this->logElapsedTime) {
    \Drupal::logger('tea_teks_srp')->info($msg);
  }

  // do the work...

  $end_time_in_ms = Timer::read($timer_name);
  Timer::stop($timer_name);
  $end_time = number_format($end_time_in_ms / 1000, 4);
  $msg = ' Vote: ' . strtolower($voting_action) .' took ' . $end_time . 's' . ' Voter: ' . number_format($user_id) . ' Citation: ' . number_format($citation_nid);
  if ($this->displayElapsedTime) {
    // Display elapsed time message.
    \Drupal::messenger()->addMessage($msg);
  }
  if ($this->logElapsedTime) {
    // Log the elapsed time message to watchdog.
    \Drupal::logger('tea_teks_srp')->info($msg);
  }
  // ...
```

## Populate a select list with the options from a list field

When you need to build a form with a drop-down (select) list of the options, you can call this function to build the select options that are defined in a `list (text)` field. You just pass the entity type e.g. `node`, the bundle or content type e.g. `article`, and the `machine name` of the field. You get back a nice array for use in the select list.

Example of list (text) field:

![Field list options](/images/field_list_options2.png)

```php
public static function getSelectOptions(string $entity_type, string $bundle, string $field_name): array {
  $options_array = [];
  $definitions = \Drupal::service('entity_field.manager')->getFieldDefinitions($entity_type, $bundle);
  if (isset($definitions[$field_name])) {
    $options_array = $definitions[$field_name]->getSetting('allowed_values');
  }
  return $options_array;
}
```

Then in our form we pass those parameters to get the `$audience_select_options`:

```php
$audience_select_options = RetrievePublisherData::getSelectOptions('node', 'teks_pub_citation', 'field_tks_audience');
```

Then use the options to populate the form element `$form['audience']` like this:

```php
$form['audience'] = [
  '#type' => 'select',
  '#title' => t('Audience'),
  '#empty_value' => '',
  '#empty_option' => '- Select the Audience -',
  '#required' => TRUE,
  '#options' => $audience_select_options,
];
```

## Get the human readable value from a list field

When you need to get the human readable value from a `list (text)` field you can use this code:

call this function to build the select options that are defined in a `list (text)` field. You just pass the entity type e.g. `node`, the bundle or content type e.g. `article`, and the `machine name` of the field. You get back a nice array for use in the select list.

Example of list (text) field:

![Field list options](/images/field_list_options2.png)

This seems to be the simplest version:

```php
  public static function getListFieldHumanReadableValue(EntityInterface $entity, string $field_name, string $list_item_value): string {
    $allowed_values = $entity->$field_name->getSetting('allowed_values');
    $human_readable_value = $allowed_values[$list_item_value];
    return $human_readable_value;
  }

```

which is called like this:

```php
$human_readable_value = VotingUtility::getListFieldHumanReadableValue($program_node, 'field_srp_program_status', 'rereview_requested');
```

`$allowed values` show up in an indexed array like this:

![list allowed values](/images/list_text_allowed_values.png)

Some other variations are:

```php
  public static function getListFieldHumanReadableValue(EntityInterface $entity, string $field_name, string

    // Option 1.
    $field = $entity->$field_name;
    $human_readable_value = $field->getFieldDefinition()
      ->getFieldStorageDefinition()
      ->getOptionsProvider('value', $field->getEntity())->getPossibleOptions()[$list_item_value];
    return $human_readable_value;

    // Option 2.
    // E.g. 'node.article.field_foo'.
    $field_string = "node.teks_pub_program.$field_name";
    $field_storage_definition = FieldConfig::load($field_string)->getFieldStorageDefinition();
    $allowed_values = $field_storage_definition->getSettings()['allowed_values'];
    $human_readable_value = $allowed_values[$list_item_value];
    return $human_readable_value;

    // Option 3.
    // E.g. 'node.article.field_foo'.
    $field_string = "node.teks_pub_program.$field_name";
    $allowed_values = FieldConfig::load($field_string)->getFieldStorageDefinition()->getSettings()['allowed_values'];
    $human_readable_value = $allowed_values[$list_item_value];
    //$x = $entity->get($field_name)->view()[0]['#markup'];
    return $human_readable_value;
  }

```

## System.schema (module is missing from your site)

When running `drush updb`, if the system reports:

```
[notice] Module rules has an entry in the system.schema key/value storage, but is missing from your site. <a href="https://www.drupal.org/node/3137656">More information about this error</a>.
[notice] Module typed_data has an entry in the system.schema key/value storage, but is not installed. <a href="https://www.drupal.org/node/3137656">More information about this error</a>.
```

[From https://www.drupal.org/node/3137656](https://www.drupal.org/node/3137656)

In the database, there is a table called `key_value` with a field called `collection` that contains the value `system.schema` for some rows. The field `name` has the names of modules.

![Image of key_value table](/images/system_schema_rules.png)

To repair these sorts of errors, you must remove the orphaned entries from the `system.schema` key/value storage system. There is no UI for doing this. You can use drush to invoke a system service to manipulate the system.schema data in the `key_value` table. For example, to clean up these two errors:

```
Module my_already_removed_module has a schema in the key_value store, but is missing from your site.
Module update_test_0 has a schema in the key_value store, but is not installed.
```

You would need to run the following commands:

```
drush php-eval "\Drupal::keyValue('system.schema')->delete('my_already_removed_module');"
drush php-eval "\Drupal::keyValue('system.schema')->delete('update_test_0');"
```

This can be done using a `hook_update_n` in your `.module` file like this:

```php
/**
 * Remove the rules and typed_data key/value entries from the system.schema.
 */
function tea_teks_update_8001() {
  \Drupal::keyValue('system.schema')->delete('rules');
  \Drupal::keyValue('system.schema')->delete('typed_data');
}
```

Alternatively, adding the missing modules with `composer install`, enabling them and deploying everything to production. Then disabling them properly, deploying that to production, then removing the modules with `composer remove` and deploying may also work. Of course, this method is a lot more work.

## Enable verbose display of warning and error messages

In `settings.php`, `settings.local.php` or `settings.ddev.php` make sure there is the following:

```php
// Enable verbose logging for errors.
// https://www.drupal.org/forum/support/post-installation/2018-07-18/enable-drupal-8-backend-errorlogdebugging-mode
$config['system.logging']['error_level'] = 'verbose';
```

Also see [Enable verbose error logging for better backtracing and debugging - April 2023](https://www.drupal.org/docs/develop/development-tools/enable-verbose-error-logging-for-better-backtracing-and-debugging)

## Reinstall modules

During module development or upgrades, it can be really useful to quickly uninstall and reinstall modules. Luckily the [devel module](https://www.drupal.org/project/devel) provides an easy way. Either navigate to `/devel/reinstall` or use the Druplicon menu option and select `development` and then click on `reinstall modules` You will need the [admin toolbar module](https://www.drupal.org/project/admin_toolbar) with it's `admin toolbar extra tools` submodule enabled.

![Menu option to reinstall modules](/images/reinstall_modules.png)



## Uninstall modules

Sometimes during development, things go goofy and you need to uninstall a module.  Hopefully you can use the Drupal user interface (Extend, uninstall modules) or `drush pmu MODULENAME`.  If these fail, try:

```sh
drush eval "\$module_data = \Drupal::config('core.extension')->get('module'); unset(\$module_data['MODULENAME']); \Drupal::configFactory()->getEditable('core.extension')->set('module', \$module_data)->save();"

drush php-eval "\Drupal::keyValue('system.schema')->delete('MODULENAME');"
```
If you have still an error you should clear the cache or rebuild the router:

```sh
drush cr
drush ev '\Drupal::service("router.builder")->rebuild();'
drush cc router (Drupal 9/10)
```

More [at this page on drupal.org](https://www.drupal.org/forum/support/post-installation/2019-10-15/how-to-uninstall-drupal-8-module-when-uninstall-page)



## View a node in JSON format

With the JSON:API and Serialization core modules enabled, simply navigate to any node and add `?_format=api_json` to the end of the URL. E.g. `https://d9book2.ddev.site/node/25?_format=api_json` 

## Display a file instead of the node

In this case, the node has a file field `field_doc_file` and we want to display that file instead of the node. This code also checks the user's permissions and rather displays the node if the user has the `administer content` permission.


Here is the route event subscriber in `docroot/modules/custom/abc_document/abc_document.services.yml`:
  
```yaml
services:
  abc_document.route_subscriber:
    class: Drupal\abc_document\Routing\RouteSubscriber
    tags:
      - { name: event_subscriber }
```

Here is the Code for the Route subscriber at `docroot/modules/custom/abc_document/src/Routing/RouteSubscriber.php`:
```php
<?php

namespace Drupal\abc_document\Routing;

use Drupal\Core\Routing\RouteSubscriberBase;
use Symfony\Component\Routing\RouteCollection;

/**
 * Listens to the dynamic route events.
 */
class RouteSubscriber extends RouteSubscriberBase {

  /**
   * {@inheritdoc}
   */
  protected function alterRoutes(RouteCollection $collection) {
    // Replace the controller for the node canonical route.
    if ($route = $collection->get('entity.node.canonical')) {
      $route->setDefaults([
        '_controller' => '\Drupal\abc_document\Controller\NodeViewController::view',
      ]);
    }
  }

}
```
And finally, the controller in `docroot/modules/custom/abc_document/src/Controller/NodeViewController.php`:

```php
<?php

namespace Drupal\abc_document\Controller;

use Drupal\Core\Entity\EntityInterface;
use Drupal\Core\StreamWrapper\StreamWrapperManagerInterface;
use Drupal\file\FileInterface;
use Drupal\node\Controller\NodeViewController as NodeViewControllerBase;
use Symfony\Component\DependencyInjection\ContainerInterface;
use Symfony\Component\HttpFoundation\BinaryFileResponse;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\HttpKernel\Exception\NotFoundHttpException;

/**
 * Defines a controller to render a single node.
 */
class NodeViewController extends NodeViewControllerBase {

  /**
   * The current request.
   *
   * @var \Symfony\Component\HttpFoundation\Request
   */
  protected Request $request;

  /**
   * The stream wrapper manager.
   *
   * @var \Drupal\Core\StreamWrapper\StreamWrapperManagerInterface
   */
  protected StreamWrapperManagerInterface $streamWrapperManager;

  /**
   * {@inheritdoc}
   */
  public static function create(ContainerInterface $container) {
    $instance = parent::create($container);

    $instance->request = $container->get('request_stack')->getCurrentRequest();
    $instance->streamWrapperManager = $container->get('stream_wrapper_manager');

    return $instance;
  }

  /**
   * {@inheritdoc}
   */
  public function view(EntityInterface $node, $view_mode = 'full', $langcode = NULL) {

    $included_types = [
      'progress_report',
      'program',
      'products',
      'finance',
      'event',
      ];

    /** @var \Drupal\node\NodeInterface $node */
    $bundle = $node->bundle();
    if (!in_array($bundle, $included_types) || !$node->hasField('field_doc_file')) {
      return parent::view($node, $view_mode);
    }

    // Display the node if the user is an admin.
    if ($this->currentUser->hasPermission('administer content')
    ) {
      return parent::view($node, $view_mode);
    }

    // If the node has no file item.
    $file = $node->get('field_doc_file')->entity;
    if (!$file) {
      throw new NotFoundHttpException();
    }
    assert($file instanceof FileInterface);
    $uri = $file->getFileUri();
    $scheme = $this->streamWrapperManager::getScheme($uri);

    // If the file does not exist.
    if (!$this->streamWrapperManager->isValidScheme($scheme) || !is_file($uri)) {
      throw new NotFoundHttpException();
    }

    // Generate the response.
    $response = new BinaryFileResponse($uri, Response::HTTP_OK, [], $scheme !== 'private');
    if (!$response->headers->has('Content-Type')) {
      $response->headers->set('Content-Type', $file->getMimeType() ?: 'application/octet-stream');
    }

    return $response;
  }

}
```




## Drupal bootstrap process

The Drupal bootstrap process is a series of steps that Drupal goes through on every page request to initialize the necessary resources and environment. You should have this article ready for your next interview.

Here are the steps:  

1. **Loading the autoloader:** The first step in the bootstrap process is to load the Composer-generated `autoloader`. This allows Drupal to use any classes defined in the codebase without explicitly requiring the files they're defined in.  
1. **Reading settings:** Drupal reads the `settings.php` file which has configuration settings for the site, such as database connection information and various other settings.
1. **Initializing the service container:** Drupal initializes the [service container](services#service-container) which is responsible for managing Drupal services. The service definitions are stored in various `.services.yml` files throughout the codebase. Drupal's service container is built on top of the Symfony service container. Documentation on the structure of this file, special characters, optional dependencies, etc. can all be found in the [Symfony service container documentation](https://symfony.com/doc/6.3/service_container.html).
1. **Handling the request:** Drupal creates a [Request object](https://api.drupal.org/api/drupal/core%21lib%21Drupal.php/function/Drupal%3A%3Arequest/8.4.x) from the global PHP variables and passes it to the `HttpKernel` to handle. The `HttpKernel` is responsible for handling the request and returning a `Response`.  
1. **Routing:** The `HttpKernel` uses the [Router service](https://git.drupalcode.org/project/drupal/-/blob/11.x/core/lib/Drupal/Core/Routing/RouteProvider.php?ref_type=heads) to match the request to a [route](routes#route). A route is a path that is defined for Drupal to return some sort of content on. The route defines a [controller](routes#controller) that should be used to generate the content for the page.  
1. **Controller execution:** A method in the controller is then executed. This method generates the content for the page. It can return a [render array](render#overview) (which Drupal will turn into HTML), a [Response object](https://www.drupal.org/docs/drupal-apis/responses/responses-overview), or some other type of content that Drupal knows how to handle.  
1. **Rendering:** If the controller returns a render array, Drupal will (via an `EventSubscriber` run it through the theme layer which renders it into HTML. This involves calling various hooks and alter functions to allow modules to modify the content.  
1. **Returning the response:** Finally, the `HttpKernel` returns a [Response object](https://www.drupal.org/docs/drupal-apis/responses/responses-overview), which is then sent to the client.

## Using hook_help

Modules can have a hook_help to display help info from the `extend` page. This is a simple example from the [examples module](https://www.drupal.org/project/examples) that shows how to use `hook_help`:

```php
/**
 * Implements hook_help().
 *
 * When implementing a hook you should use the standard text "Implements
 * HOOK_NAME." as the docblock for the function. This is an indicator that
 * further documentation for the function parameters can be found in the
 * docblock for hook being implemented and reduces duplication.
 *
 * This function is an implementation of hook_help(). Following the naming
 * convention for hooks, the "hook_" in hook_help() has been replaced with the
 * short name of our module, "hooks_example_" resulting in a final function name
 * of hooks_example_help().
 */
function hooks_example_help($route_name, RouteMatchInterface $route_match) {
  switch ($route_name) {
    // For help overview pages we use the route help.page.$moduleName.
    case 'help.page.hooks_example':
      return '<p>' . t('This text is provided by the function <code>hooks_example_help()</code>, which is an implementation of <code>hook hook_help()</code>. To learn more about how this works checkout the code in <code>hooks_example.module</code>.') . '</p>';
  }
}
```

This version loads the help text from a file:

```php
/**
 * implement hook_help
 **/
function route_play_help($route, $help) {
  switch ($route) {
    case 'help.page.route_play':
      $file_contents = file_get_contents( dirname(__FILE__) . "/README.md");
      $cleaned_contents =  Drupal\Component\Utility\Html::escape($file_contents);
      // Add breaks so the text is not all on one line.
      $cleaned_contents = str_replace("\n", "<br>", $cleaned_contents);
      return $cleaned_contents;
}

```
And from the [workbench menu access module](https://www.drupal.org/project/workbench_menu_access), this version uses the [markdown module](https://www.drupal.org/project/markdown) to display a markdown file:

```php
/**
 * Help page text.
 *
 * @param string $route_name
 *   The route name.
 * @param \Drupal\Core\Routing\RouteMatchInterface $route_match
 *   The route matcher service.
 *
 * @return string
 *   An HTML string.
 */
function workbench_menu_access_help($route_name, RouteMatchInterface $route_match) {
  $output = '';
  switch ($route_name) {
    case 'help.page.workbench_menu_access':
      $readme = __DIR__ . '/README.md';
      $text = file_get_contents($readme);

      // If the Markdown module is installed, use it to render the README.
      if ($text !== FALSE && \Drupal::moduleHandler()->moduleExists('markdown') === TRUE) {
        $filter_manager = \Drupal::service('plugin.manager.filter');
        $settings = \Drupal::configFactory()->get('markdown.settings')->getRawData();
        $config = ['settings' => $settings];
        /** @var \Drupal\filter\Plugin\FilterInterface $filter */
        $filter = $filter_manager->createInstance('markdown', $config);
        $output = $filter->process($text, 'en');
      }
      // Else the Markdown module is not installed output the README as text.
      elseif ($text !== FALSE) {
        $output = '<pre>' . $text . '</pre>';
      }

      // Add a link to the Drupal.org project.
      $output .= '<p>';
      $output .= t('Visit the <a href=":project_link">Workbench Menu Access project page</a> on Drupal.org for more information.', [
        ':project_link' => 'https://www.drupal.org/project/workbench_menu_access',
      ]);
      $output .= '</p>';
      break;
  }

  return $output;
}
```

## Display a message after a module is installed

From [Menu custom access module](https://www.drupal.org/project/menu_custom_access) this code displays a message after the module is installed.  In `web/modules/contrib/menu_custom_access/menu_custom_access.install`:


```php

function menu_custom_access_install() {
  \Drupal::messenger()->addStatus(t("Menu Custom Access is now enabled"));
}
```

## Generate a Leaflet map with popups in a block

Using the [Leaflet module](https://www.drupal.org/project/leaflet), you can create a map with popups in a block. In `web/modules/custom/leafmap/src/Plugin/Block/LeafMapBlock.php`:

```php
<?php

namespace Drupal\leafmap\Plugin\Block;

use Drupal\Core\Block\BlockBase;
use Drupal\Core\Plugin\ContainerFactoryPluginInterface;
use Symfony\Component\DependencyInjection\ContainerInterface;
use Drupal\leaflet\LeafletService;

/**
 * Provides a 'LeafMapBlock' block.
 *
 * @Block(
 *  id = "leaf_map_block",
 *  admin_label = @Translation("Leaf map block"),
 * )
 */
class LeafMapBlock extends BlockBase implements ContainerFactoryPluginInterface {

  /**
   * Drupal\leaflet\LeafletService definition.
   *
   * @var \Drupal\leaflet\LeafletService
   */
  protected $leafletService;

  /**
   * Constructs a new LeafMapBlock object.
   */
  public function __construct(array $configuration, $plugin_id, $plugin_definition, LeafletService $leaflet_service) {
    parent::__construct($configuration, $plugin_id, $plugin_definition);
    $this->leafletService = $leaflet_service;
  }

  public static function create(ContainerInterface $container, array $configuration, $plugin_id, $plugin_definition) {
    return new static(
      $configuration,
      $plugin_id,
      $plugin_definition,
      $container->get('leaflet.service')
    );
  }

  /**
   * {@inheritdoc}
   */
  public function build() {
    $build = [];

// Define your points.
    $points = [
      ['lat' => 37.7749, 'lon' => -122.4194, 'city' => 'San Francisco', 'job' => 'Software Engineer'],
      ['lat' => 34.0522, 'lon' => -118.2437, 'city' => 'Los Angeles', 'job' => 'Data Analyst'],
      ['lat' => 36.746841, 'lon' => -119.772591, 'city' => 'Fresno', 'job' => 'Web Developer'],
      ['lat' => 30.2672, 'lon' => -97.7431, 'city' => 'Austin', 'job' => 'Software Engineer'],
      // Add more points as needed.
    ];

    // Convert points to features.
    $features = [];
    foreach ($points as $point) {
      $popupContent = $point['city'] . '<br>Lat: ' . $point['lat'] . '<br>Lon: ' . $point['lon'] . '<br>Job: ' . $point['job'];
      $features[] = [
        'type' => 'point',
        'lat' => $point['lat'],
        'lon' => $point['lon'],
        'popup' => [
          'value' => $popupContent,
          ],
      ];
    }

    // Create a map with the features.
    $map = leaflet_map_get_info('OSM Mapnik');
    
    // Add Clustering by enabling the Leaflet Markercluster module.
    $map['settings']['leaflet_markercluster']['control'] = TRUE;

    $build['map'] = $this->leafletService->leafletRenderMap($map, $features, '500px');

    // Attach the library.
    $build['#attached']['library'][] = 'leafmap/leaflet-popup';

    return $build;
  }

}
```

For styling the popups, in `web/modules/custom/leafmap/leafmap.libraries.yml` add:

```yaml
leaflet-popup:
  css:
    theme:
      css/leaflet-popup.css: {}
```

and the css file at `web/modules/custom/leafmap/css/leaflet-popup.css`:

```css
.leaflet-popup-content {
  color: #333;
  font-size: 24px;
  line-height: 1.5;
}
```

It should look like this:

![Leaflet map with popups](/images/leaf-map-block.png)


### Customize the map pointers

You can customize the map pointers by specifying a path to a local `.png` file or a remote file on a CDN. Then pass the `$icon` as one of the elements of the `$features` array. You can see how in this snippet from the `build()` function:

```php
    $icon = [
      // 'iconUrl' => 'https://cdn.rawgit.com/pointhi/leaflet-color-markers/master/img/marker-icon-2x-green.png',
      //'iconUrl' => '/themes/custom/abcd/assets/img/usa-icons/api.svg',
      'iconUrl' => '/sites/default/files/icon-electric.png',
      'iconSize' => [25, 41],
      'iconAnchor' => [12, 41],
      'popupAnchor' => [1, -34],
      'shadowSize' => [41, 41],
      // shadowUrl: 'my-icon-shadow.png',
      // shadowRetinaUrl: 'my-icon-shadow@2x.png',
    ];

    $features[] = [
      'type' => 'point',
      'lat' => $lat,
      'lon' => $long,
      'popup' => [
        'value' => implode('<hr>', $popupContent),
        'options' => $this->configuration['popup_options'] ?? '{"maxWidth":"300","minWidth":"50", "autoPan": true}',
      ],
      'icon' => $icon,
    ];
```




## Add StringTranslationTrait to a class to use $this-t()

To use the `$this->t()` method in a class, add the `use StringTranslationTrait` to the class. It looks like this:

```php
<?php

declare(strict_types=1);

namespace Drupal\abc_workbench;

use Drupal\Core\Messenger\MessengerInterface;
use Drupal\Core\StringTranslation\StringTranslationTrait;

/**
 * Service for building the menu table.
 */
final class MenuTableBuilder implements MenuTableBuilderInterface {

  use StringTranslationTrait;
...
  public function buildTable(int $page): array {
    $uid = $this->currentUser->id();
    ...
        foreach ($pagedMenus as $menu_name => $menu) {
      $edit_url = Url::fromRoute('entity.menu.edit_form', ['menu' => $menu_name]);
      $add_link_url = Url::fromRoute('entity.menu.add_link_form', ['menu' => $menu_name]);
      $rows[] = [
        'title' => $menu->label(),
        'description' => $menu_label,
        'operations' => [
          'data' => [
            '#type' => 'operations',
            '#links' => [
              'edit' => [
                'title' => $this->t('Edit'),
                'url' => $edit_url,
              ],
              'add_link' => [
                'title' => $this->t('Add link'),
                'url' => $add_link_url,
              ],
            ],
          ],
        ],
      ];
    }
```


## Overriding admin menu listing 

In this instance, the requirement was to change the menu listing to only show menus that the user has access to. This is done by overriding the `MenuListBuilder` class. First you have to change the handler class in a `hook_entity_type_alter()` in `abc_workbench.module`:

```php
/**
 * Implements hook_entity_type_alter().
 */
function abc_workbench_entity_type_alter(array &$entity_types) {
  $entity_types['menu']->setHandlerClass('list_builder', AbcWorkbenchMenuListBuilder::class);
}
```
Then define the `listbuilder` service in the `.services.yml` file: `docroot/modules/custom/abc_workbench/abc_workbench.services.yml`:

```yaml
  abc_workbench.menu_list_builder:
    class: Drupal\abc_workbench\AbcWorkbenchMenuListBuilder
    arguments: [ '@entity_type.manager', '@entity_type.manager' ]
    tags:
      - { name: entity_list_builder, entity_type: menu }
```

Then lastly, you create the `AbcWorkbenchMenuListBuilder` class in `docroot/modules/custom/abc_workbench/src/AbcWorkbenchMenuListBuilder.php`.  Note that you only have to implement the functions that you want to override. In this case, it was the `getEntityIds()` function which can leverage custom or additional functions to help filter the output.  In this case, it checks the user's access to the menu and only includes those menus that the user has access to.:

Here is the `getEntityIds()` function:

```php

  /**
   * {@inheritdoc}
   */
  protected function getEntityIds() {
    // Remove the limit from the parent.
    $this->limit = NULL;

    // Get all menus to check access.
    $query = $this->getStorage()->getQuery()->sort('label', 'ASC');
    $allMenus = $query->execute();
    $includeMenus = [];

    // Check access for each menu and identify which menus to include.
    foreach ($allMenus as $menu_id) {
      // Load the menu entity using the menu ID.
      $menu = $this->getStorage()->load($menu_id);
      if ($menu && $this->checkSections($menu, $this->currentUser)) {
        $includeMenus[] = $menu->id();
      }
    }

    // Include only the menus that have access.
    $query = $this->getStorage()->getQuery()
      ->condition('id', $includeMenus, 'IN')
      ->sort('label', 'ASC');

    return $query->execute();
  }
```

Also for clarity, here is the `checkSections()` function that is called in the `getEntityIds()` function:

```php
/**
   * Check Menu access.
   *
   * @param \Drupal\Core\Entity\EntityInterface $menu
   *   The entity to check.
   * @param \Drupal\Core\Session\AccountInterface $account
   *   The account to check.
   *
   * @return bool
   *   TRUE if access is granted, FALSE otherwise.
   *
   * @throws \Drupal\Component\Plugin\Exception\InvalidPluginDefinitionException
   * @throws \Drupal\Component\Plugin\Exception\PluginNotFoundException
   */
  protected function checkSections(EntityInterface $menu, AccountInterface $account): bool {
    static $check;
    // Internal cache for performance.
    $key = $menu->id() . ':' . $account->id();
    if (!isset($check[$key])) {
      // By default, ignore menus that don't explicitly have permissions.
      $check[$key] = FALSE;
      // Check for admin role.
      if ($account->hasPermission('administer workbench menu access') || $account->hasPermission('bypass workbench access')) {
        return TRUE;
      }
      $config = $this->configFactory->get('workbench_menu_access.settings');
      $active = $config->get('access_scheme');
      $settings = $menu->getThirdPartySetting('workbench_menu_access', 'access_scheme');
      if (!is_null($active) && !is_null($settings)) {
        /** @var \Drupal\workbench_access\Entity\AccessSchemeInterface $scheme */
        $scheme = $this->entityTypeManager->getStorage('access_scheme')
          ->load($active);
        $user_sections = $this->userSectionStorage->getUserSections($scheme, $account);

        // Check children / parents.
        $check[$key] = $this->workbenchAccessManager::checkTree($scheme, $settings, $user_sections);
      }
    }
    return $check[$key];
  }
```

If you need to override the user entity listbuilder, check out [Overriding the User entity list_builder handler](https://drupal.stackexchange.com/questions/284599/overriding-the-user-entity-list-builder-handler)

- [EntityListBuilder API](https://api.drupal.org/api/drupal/core%21lib%21Drupal%21Core%21Entity%21EntityListBuilder.php/class/EntityListBuilder/10)
- [UserListBuilder API which extends EntityListBuilder](https://api.drupal.org/api/drupal/core%21modules%21user%21src%21UserListBuilder.php/class/UserListBuilder/10)
- [MenuListBuilder API which extends EntityListBuilder](https://api.drupal.org/api/drupal/core%21modules%21menu_ui%21src%21MenuListBuilder.php/class/MenuListBuilder/10)



## Troubleshoot memory problems

In some cases, where there are lots of `Node::load()`  or `Node::loadMultiple()` calls, you may run into `out of memory` errors. If increasing the memory limit in `php.ini` (e.g. `memory_limit = 1024M`) doesn't resolve this, you might try flushing the entity memory cache with:

```php
\Drupal::service('entity.memory_cache')->deleteAll();
```

There is also a [Memory limit Policy module](https://www.drupal.org/project/memory_limit_policy) that is worth checking out to override the default memory_limit for specific paths, roles etc.

You can use the `memory_get_usage()` function to see how much memory is being used. You can also use the `memory_get_peak_usage()` function to see the maximum amount of memory used during the script's execution.

```php
$mgu1 = round(memory_get_usage() / 1024 / 1024, 2) . ' MB';
$mgu2 = round(memory_get_usage(TRUE) / 1024 / 1024, 2) . ' MB';
$mgpu1 = round(memory_get_peak_usage() / 1024 / 1024, 2) . ' MB';
$mgpu2 = round(memory_get_peak_usage(TRUE) / 1024 / 1024, 2) . ' MB';
\Drupal::logger('tea_teks_srp')->info('Memory usage: ' . $mgu1 . ' ' . $mgu2 . ' Peak: ' . $mgpu1 . ' ' . $mgpu2);
```

It may be worth looking at the `getOptimalMemoryLimit()` and `setMemoryLimit()` functions in the [xmlsitemap](https://www.drupal.org/project/xmlsitemap) module.  

```php
  public function getOptimalMemoryLimit() {
    $optimal_limit = &drupal_static(__FUNCTION__);
    if (!isset($optimal_limit)) {
      // Set the base memory amount from the provided core constant.
      $optimal_limit = Bytes::toNumber(\Drupal::MINIMUM_PHP_MEMORY_LIMIT);

      // Add memory based on the chunk size.
      $optimal_limit += xmlsitemap_get_chunk_size() * 500;

      // Add memory for storing the url aliases.
      if ($this->config->get('prefetch_aliases')) {
        $aliases = $this->connection->query("SELECT COUNT(id) FROM {path_alias}")->fetchField();
        $optimal_limit += $aliases * 250;
      }
    }
    return $optimal_limit;
  }

  /**
   * {@inheritdoc}
   */
  public function setMemoryLimit($new_limit = NULL) {
    $current_limit = @ini_get('memory_limit');
    if ($current_limit && $current_limit != -1) {
      if (!is_null($new_limit)) {
        $new_limit = $this->getOptimalMemoryLimit();
      }
      if (Bytes::toNumber($current_limit) < $new_limit) {
        return @ini_set('memory_limit', $new_limit);
      }
    }
  }
```
The [full source code for the module is here](https://git.drupalcode.org/project/xmlsitemap).



Down the rabbit hole: 
- [Changing PHP memory limits - May 2022](https://www.drupal.org/docs/7/managing-site-performance-and-scalability/changing-php-memory-limits)
- [memory_get_usage()](https://www.php.net/manual/en/function.memory-get-usage.php)
- [memory_get_peak_usage()](https://www.php.net/manual/en/function.memory-get-peak-usage.php)
- [EntityMemoryCache](https://api.drupal.org/api/drupal/core%21lib%21Drupal%21Core%21Entity%21EntityMemoryCache.php/class/EntityMemoryCache/8.9.x)
- [gc_collect_cycles - Forces collection of any existing garbage cycles](https://www.php.net/manual/en/function.gc-collect-cycles.php)
- [Collecting Cycles - reference counting memory mechanisms](https://www.php.net/manual/en/features.gc.collecting-cycles.php)



## Config split

In your `settings.php` file, you identify the active split with:

```php
/*
 * Config Split settings.
 */
// Note. local is active
$config['config_split.config_split.local']['status'] = TRUE;
$config['config_split.config_split.dev']['status'] = FALSE;
$config['config_split.config_split.stage']['status'] = FALSE;
$config['config_split.config_split.prod']['status'] = FALSE;
```

Assuming you have the `config_split` module installed, you will need a `split` for each environment at `admin/config/development/configuration/config-split`.  Commonly: `local`, `dev`, `stage`, `prod`.
Specify a directory such as `../config/local` for the `local` split.  Configuration files that are specific to this split are stored here. Similarly, you could use:
* `dev`: `../config/dev`
* `test`:  `../config/test`
* `prod`:  `../config/prod`
This assumes that the `settings.php` specifies the config directory as `../config/sync` with `$settings['config_sync_directory'] = '../config/sync';`


When you configure the active split in your `settings.php`, you can export the configuration for that split with `ddev drush cex -y`. There is no need to use the `ddev drush config-split: export local` command any more.  The `ddev drush cex ` command handles it all correctly.






### Overview of steps required for each environment.
* Set the active split in `settings.php` e.g. `$config['config_split.config_split.local']['status'] = TRUE;`
* Import the current configuration with `ddev drush cim -y`
* Make your changes to the configuration via the Drupal u/i
* Export your changes with `ddev drush cex -y`. This will handle the split file e.g. `config/sync/config_split.config_split.local.yml` (for the local split) as well as the specific configuration changes that you made for that split.
* Test and commit your changes.
* Repeat for each environment.



### Enable a module on a specific environment

For a module enabled on a specific environment e.g. cron fail alert on `prod` 
* Select the `prod` split in `settings.php` (or `settings.local.php`).
* Clear cache with `ddev drush cr`
* `ddev drush cim` so you are using the `prod` split configuration.
* Enable the module in Drupal and configure it as needed.
* In the config split settings for `prod`, check the cron fail alert in `complete split`

Running a `ddev drush cst` you should see that the `prod` config split has changed, the `core.extension` has changed and there is a `cron_fail_alert.settings.yml`.

When you export the configuration with `ddev drush cex -y`, the files will be updated to reflect these changes.


Also the `config_split.config_split.prod.yml` file will list `cron_fail_alert` under the `modules` key:
```yml
label: Production
description: 'Config Split for Production'
weight: 0
stackable: false
no_patching: false
storage: folder
folder: ../config/prod
module:
  cron_fail_alert: 0
theme: {  }
complete_list:
  - environment_indicator.indicator
  - dblog.settings
partial_list: {  }
```

A `git status` should show you that the `config/prod/cron_fail_alert.settings.yml` has been created. Notice this file is in the `config/prod` directory. This means that when you deploy to `prod` (and `drush cim`), the `cron_fail_alert` module will be enabled and the configuration you specified for it will be correctly loaded.


### Use different settings for different environments
Here we want to use database logging (watchdog) for all environments, but we want to keep 100,000 items in the log for `prod`, and 10,000 items for dev, stage and local. 

* Set the active split in `settings.php` e.g. `$config['config_split.config_split.local']['status'] = TRUE;`
* Import the current configuration with `ddev drush cim -y`
* Enable the `Database logging` module in Drupal and configure it to keep 10,000 items.
* In config split, check the `dblog.settings` in the Configuration items.
* Repeat this for `dev` and `stage` environments.
* Export the configuration with `ddev drush cex -y`
 


::: tip Note
Using the [chosen](https://www.drupal.org/project/chosen) module on your site will make the config split a little easier as it displays the selected items in a more user-friendly way.
:::

## Modify SOLR Search behavior

The now deprecated way of doing this was to use the `hook_search_api_solr_query_alter()` hook in a custom module.  From [How to replace the deprecated hook hook_search_api_solr_query_alter with PreQueryEvent on Drupal.org - updated Apr 2024](https://www.drupal.org/docs/8/modules/search-api-solr/search-api-solr-howtos/how-to-replace-the-deprecated-hook-hook_search_api_solr_query_alter-with-prequeryevent) this process needs to be replaced with an event subscriber.  

You can see an [example of this](https://git.drupalcode.org/project/acquia_search/-/blob/3.1.x/src/EventSubscriber/PreQuery/EdisMax.php?ref_type=heads) in the [Acquia Search module](https://www.drupal.org/project/acquia_search) also. 

Here is the old way: In this example, this code will alter the functionality of the search when used with the `site_search` and`related_position_listings` views.  It boosts the search results based on the `field_boosted_created_date` field and the `field_position_campus` field.  It also filters the search results based on the `field_position_title_select` field and the `field_other_position_title` field.  It is an interesting example of some of the tweaking that can be done to improve search results for your use case.  For example, if you want the search results to show staff first, or show the most recent news items first, or show the most relevant job listings first, you can use this hook to adjust the search results accordingly.

```php
/**
 * Implements hook_search_api_solr_query_alter().
 * 
 * @param \Solarium\Core\Query\QueryInterface $solarium_query
 * @param \Drupal\search_api\Query\QueryInterface $query
 * @throws \Drupal\search_api\SearchApiException
 */
function abc_search_search_api_solr_query_alter(\Solarium\Core\Query\QueryInterface $solarium_query, \Drupal\search_api\Query\QueryInterface $query) {
  $request = \Drupal::request();
  $view = $query->getOption('search_api_view');

  if(!($view)) {
    return;
  }

  if ($view->id() === 'site_search' && !empty($request->get('f')) && in_array('content_type:news_universal_news_item', $request->get('f'))) {
    $boosted_created_date_field_name = 'field_boosted_created_date';
    $solr_field_names = $query->getIndex()->getServerInstance()->getBackend()->getSolrFieldNames($query->getIndex());
    if (isset($solr_field_names[$boosted_created_date_field_name]) && !empty($solr_field_names[$boosted_created_date_field_name])) {
      // Boost documents by date.
      $boost_functions = 'recip(ms(NOW,' . $solr_field_names[$boosted_created_date_field_name] . '),3.16e-15,1,1)';
      $solarium_query->getEDisMax()->setBoostFunctionsMult($boost_functions);

      // Avoid the conversion into a lucene parser expression, keep edismax.
      $solarium_query->addParam('defType', 'edismax');
      $helper = $solarium_query->getHelper();

      // if keys entered, also boost the title
      if (!empty($query->getKeys())) {
        // TODO: This doesn't do anything? Remove/refactor
        $search_terms = array_filter($query->getKeys(), function ($v, $k) {
          return is_numeric($k);
        }, ARRAY_FILTER_USE_BOTH);

        foreach ($search_terms as $term) {
          $solarium_query->addParam('bq', 'tm_X3b_und_title:' . $helper->escapeTerm($term) . '^15');
        }
      }
    }
  }

  // Related position listings search
  if ($view->id() === 'related_position_listings' && !empty($view->args)) {
    $nid = $view->args[0];
    $node = \Drupal::entityTypeManager()->getStorage('node')->load($nid);
    $solr_field_names = $query->getIndex()->getServerInstance()->getBackend()->getSolrFieldNames($query->getIndex());

    $solarium_query->addParam('defType', 'edismax');
    $query->getParseMode()->setConjunction('OR');
    $helper = $solarium_query->getHelper();

    $bqParams = [];
    $fqParams = [];

    // Set search query (based on keywords).
    if ($node->hasField('field_position_keywords') && !empty($node->field_position_keywords->value)) {
      $values = explode(',', $node->field_position_keywords->value);
      array_walk($values, function(&$v) {
        $v = trim($v, " \"'");
      });

      $query->keys('"' . implode('" "', $values) . '"');
    }

    // Filter by position
    if ($node->hasField('field_position_title_select') && isset($node->field_position_title_select->entity)) {
      // Ensure "position title" tid matches
      $field_position_title_select_field_name = 'field_position_title_select';
      if (isset($solr_field_names[$field_position_title_select_field_name]) && !empty($solr_field_names[$field_position_title_select_field_name])) {
        //$solarium_query->addParam('fq', 'itm_field_position_title_select:' . $node->field_position_title_select->entity->id());
        $fqParams[] = 'itm_field_position_title_select:' . $node->field_position_title_select->entity->id();
      }

      // Ensure other "position title" value matches if relevant
      if (strcasecmp($node->field_position_title_select->entity->label(), 'Other') === 0) {
        $field_other_position_title_field_name = 'field_other_position_title';
        if (isset($solr_field_names[$field_other_position_title_field_name]) && !empty($solr_field_names[$field_other_position_title_field_name])) {
          //$solarium_query->addParam('fq', 'ss_field_other_position_title:' . $node->field_other_position_title->value);
          $fqParams[] = 'ss_field_other_position_title:' . $helper->escapePhrase($node->field_other_position_title->value);
        }
      }
    }

    // Boost job location field.
    if ($node->hasField('field_position_campus') && !empty($node->field_position_campus->value)) {
      $bqParams[] = 'sm_field_position_campus:' . $helper->escapePhrase($node->field_position_campus->value) . '^10';
    }

    // Boost appointment type field.
    if ($node->hasField('field_appointment_type') && !empty($node->field_appointment_type->value)) {
      $bqParams[] = 'sm_field_appointment_type:' . $helper->escapePhrase($node->field_appointment_type->value) . '^10';
    }

    // Boost work schedule field.
    if ($node->hasField('field_work_schedule') && !empty($node->field_work_schedule->value)) {
      $bqParams[] = 'sm_field_work_schedule:' . $helper->escapePhrase($node->field_work_schedule->value) . '^10';
    }

    // Boost citizenship field.
    if ($node->hasField('field_citizenship') && !empty($node->field_citizenship->value)) {
      $bqParams[] = 'ss_field_citizenship:' . $helper->escapePhrase($node->field_citizenship->value) . '^10';
    }

    if (count($bqParams) > 0) {
      $solarium_query->addParam('bq', $bqParams);
    }

    if (count($fqParams) > 0) {
      $solarium_query->addParam('fq', $fqParams);
    }
  }
}
```


The new way which uses an event subscriber, looks like this:


In the `abc_search` module, the `abc_search.services.yml` file defines the event subscriber:

```yaml
services:
  abc_search.query_alter:
    class: Drupal\abc_search\EventSubscriber\SolrQueryAlterEventSubscriber
    tags:
      - { name: event_subscriber }
```

Then in the file `src/EventSubscriber/SolrQueryAlterEventSubscriber.php`:

```php
<?php

declare(strict_types=1);

namespace Drupal\abc_search\EventSubscriber;

use Drupal\search_api_solr\Event\PreQueryEvent;
use Drupal\search_api_solr\Event\SearchApiSolrEvents;
use Symfony\Component\EventDispatcher\EventSubscriberInterface;

/**
 * Alter the solr query results.
 */
final class SolrQueryAlterEventSubscriber implements EventSubscriberInterface {

  /**
   * {@inheritdoc}
   */
  public static function getSubscribedEvents(): array {
    return [
      SearchApiSolrEvents::PRE_QUERY => 'preQuery',
    ];
  }

  /**
   * {@inheritdoc}
   */
  public function preQuery(PreQueryEvent $event): void {
    $query = $event->getSearchApiQuery();
    $solarium_query = $event->getSolariumQuery();

    $request = \Drupal::request();
    $view = $query->getOption('search_api_view');
    if(!($view)) {
      return;
    }

    /*
     * When searching, if you select news in the faceted search, modify
     * boosting behavior to prioritize more recent creation date.
     */
    if ($view->id() === 'site_search' && !empty($request->get('f')) && in_array('content_type:news_universal_news_item', $request->get('f'))) {
      $boosted_created_date_field_name = 'field_boosted_created_date';
      $solr_field_names = $query->getIndex()->getServerInstance()->getBackend()->getSolrFieldNames($query->getIndex());
      if (isset($solr_field_names[$boosted_created_date_field_name]) && !empty($solr_field_names[$boosted_created_date_field_name])) {
        // Boost documents by date.
        $boost_functions = 'recip(ms(NOW,' . $solr_field_names[$boosted_created_date_field_name] . '),3.16e-15,1,1)';
        $solarium_query->getEDisMax()->setBoostFunctionsMult($boost_functions);

        // Avoid the conversion into a Lucene parser expression, keep edismax.
        $solarium_query->addParam('defType', 'edismax');
        $helper = $solarium_query->getHelper();

        // If keys entered, also boost the title.
        if (!empty($query->getKeys())) {
          $search_terms = array_filter($query->getKeys(), function ($v, $k) {
            return is_numeric($k);
          }, ARRAY_FILTER_USE_BOTH);

          foreach ($search_terms as $term) {
            $solarium_query->addParam('bq', 'tm_X3b_und_title:' . $helper->escapeTerm($term) . '^15');
          }
        }
      }
    }

    // Related position listings search.
    if ($view->id() === 'related_position_listings' && !empty($view->args)) {

      $nid = $view->args[0];
      $node = \Drupal::entityTypeManager()->getStorage('node')->load($nid);
      $solr_field_names = $query->getIndex()->getServerInstance()->getBackend()->getSolrFieldNames($query->getIndex());

      /*
       * Use Extended DisMax (eDisMax) query parser for additional features
       * and flexibility for handling complex queries.
       */
      $solarium_query->addParam('defType', 'edismax');
      $query->getParseMode()->setConjunction('OR');
      $helper = $solarium_query->getHelper();

      $bqParams = [];
      $fqParams = [];

      // Set search query (based on keywords).
      if ($node->hasField('field_position_keywords') && !empty($node->field_position_keywords->value)) {
        $values = explode(',', $node->field_position_keywords->value);
        array_walk($values, function (&$v) {
          /*
           * Remove leading or trailing whitespace, double quotes, single
           *  quotes.
           */
          $v = trim($v, " \"'");
          // This might be a good idea in the future.
          // $special_chars = ['\\', '+', '-', '&&', '||', '!', '(', ')', '{', '}', '[', ']', '^', '"', '~', '*', '?', ':', '/'];
          // $v = str_replace($special_chars, array_map(fn($char) => '\\' . $char, $special_chars), $v);
        });
        $query->keys('"' . implode('" "', $values) . '"');
      }

      // Filter by position.
      if ($node->hasField('field_position_title_select') && isset($node->field_position_title_select->entity)) {
        // Ensure "position title" tid matches.
        $field_position_title_select_field_name = 'field_position_title_select';
        if (isset($solr_field_names[$field_position_title_select_field_name]) && !empty($solr_field_names[$field_position_title_select_field_name])) {
          //$solarium_query->addParam('fq', 'itm_field_position_title_select:' . $node->field_position_title_select->entity->id());
          $fqParams[] = 'itm_field_position_title_select:' . $node->field_position_title_select->entity->id();
        }

        // Ensure other "position title" value matches if relevant
        if (strcasecmp($node->field_position_title_select->entity->label(), 'Other') === 0) {
          $field_other_position_title_field_name = 'field_other_position_title';
          if (isset($solr_field_names[$field_other_position_title_field_name]) && !empty($solr_field_names[$field_other_position_title_field_name])) {
            //$solarium_query->addParam('fq', 'ss_field_other_position_title:' . $node->field_other_position_title->value);
            $fqParams[] = 'ss_field_other_position_title:' . $helper->escapePhrase($node->field_other_position_title->value);
          }
        }
      }

      // Boost job location field.
      if ($node->hasField('field_position_campus') && !empty($node->field_position_campus->value)) {
        $bqParams[] = 'sm_field_position_campus:' . $helper->escapePhrase($node->field_position_campus->value) . '^10';
      }

      // Boost appointment type field.
      if ($node->hasField('field_appointment_type') && !empty($node->field_appointment_type->value)) {
        $bqParams[] = 'sm_field_appointment_type:' . $helper->escapePhrase($node->field_appointment_type->value) . '^10';
      }

      // Boost work schedule field.
      if ($node->hasField('field_work_schedule') && !empty($node->field_work_schedule->value)) {
        $bqParams[] = 'sm_field_work_schedule:' . $helper->escapePhrase($node->field_work_schedule->value) . '^10';
      }

      // Boost citizenship field.
      if ($node->hasField('field_citizenship') && !empty($node->field_citizenship->value)) {
        $bqParams[] = 'ss_field_citizenship:' . $helper->escapePhrase($node->field_citizenship->value) . '^10';
      }

      if (count($bqParams) > 0) {
        $solarium_query->addParam('bq', $bqParams);
      }

      if (count($fqParams) > 0) {
        $solarium_query->addParam('fq', $fqParams);
      }
    }
  }

}
```

::: tip Note
There is a useful helper class provided by the [Solarium package](https://packagist.org/packages/solarium/solarium) that can be used to escape terms and phrases.  This is used in the code above.  You access it via `$helper = $solarium_query->getHelper();` and then you can use `$helper->escapeTerm($term)` or `$helper->escapePhrase($phrase)`.  Other functions include: `escapeXMLCharacterData()`, `filterControlCharacters()` and `escapeLocalParamValue()`.  These all have some nice regex patterns you can use in your code.
:::


## Enhance the relevance of SOLR search results
This function customizes the Solr documents by adding a boosted created date field for specific content types before they are indexed.

The purpose of adding a boosted created date field is to enhance the relevance of search results by giving higher importance to the creation date of certain content types. In this case, the field_boosted_created_date is added to Solr documents to ensure that documents of the content type news_universal_news_item have their creation date indexed. This allows the search system to prioritize or boost these documents based on their creation date, potentially improving the search ranking for more recent or relevant news items. For other content types, this field is set to NULL, indicating that they do not receive the same boost based on their creation date.

Note. this function doesn't actually do any boosting.  That is handled elsewhere in the code.  This function just adds the field to the Solr documents.

```php
/**
 * Alter Solr documents before they are sent to Solr for indexing.
 *
 * @param \Solarium\QueryType\Update\Query\Document[] $documents
 *   An array of \Solarium\QueryType\Update\Query\Document\Document objects
 *   ready to be indexed, generated from $items array.
 * @param \Drupal\search_api\IndexInterface $index
 *   The search index for which items are being indexed.
 * @param \Drupal\search_api\Item\ItemInterface[] $items
 *   An array of items to be indexed, keyed by their item IDs.
 */
function abc_search_search_api_solr_documents_alter(array &$documents, \Drupal\search_api\IndexInterface $index, array $items) {
  if($index->id() !== 'abc') {
    return;
  }

  // First we need to create an Index field.
  $boosted_created_date_field_name = 'field_boosted_created_date';
  if (is_null($index->getField($boosted_created_date_field_name))) {
    $index_field = new \Drupal\search_api\Item\Field($index, $boosted_created_date_field_name);
    $index_field->setType('date');
    $index_field->setPropertyPath('created');
    $index_field->setDatasourceId('entity:node');
    $index_field->setLabel('Boosted Created Date');
    $index->addField($index_field);
    $index->save();
  }

  foreach ($documents as $document) {
    // Set the field data for Universal News content type.
    // For all other content types, set NULL
    $solr_field_name = 'ds_' . $boosted_created_date_field_name;
    if ($document->getFields()['ss_type'] === 'news_universal_news_item') {
      $document->setField($solr_field_name, $document->getFields()['ds_created']);
    } else {
      $document->setField($solr_field_name, NULL);
    }
  }
}
```

## Custom edit permissions per user

Here is a quick little module I cobbled together to allow an admin to tag a node with a term, the user would have the same term.  This allows the user to edit that node even if they don't have permissions to do so via normal Drupal channels.  

This requires a few things:
1. A vocabulary called `wecc_edit_access`
2. A field on the node called `field_wecc_edit_access` that is a term reference to the `wecc_edit_access` vocabulary.
3. A field on the user called `field_wecc_user_edit_access` that is a term reference to the `wecc_edit_access` vocabulary.

Then the following code in `web/modules/custom/wecc_node_access/wecc_node_access.module`:

```php
<?php

/**
 * @file
 * Primary module hooks for WECC Node Access module.
 */


use Drupal\Core\Session\AccountInterface;
use Drupal\node\NodeInterface;
use Drupal\Core\Access\AccessResult;
use \Drupal\user\Entity\User;
use \Drupal\Core\Form\FormStateInterface;

/**
 * Implements hook_node_access().
 */
function wecc_node_access_node_access(NodeInterface $node, $op, AccountInterface $account): AccessResult {
  $type = $node->bundle();
  if ($type !== 'page') {
    return AccessResult::neutral();
  }

  if ($op == 'update') {
    // Load term id from field field_wecc_edit_access.
    $node_edit_access_tid = $node->get('field_wecc_edit_access')->target_id;
    if (is_null($node_edit_access_tid)) {
      return AccessResult::neutral()->addCacheableDependency($node);
    }

    // load the field_wecc_user_edit_access from the user.
    $user = User::load($account->id());
    // load all the values from the field field_wecc_user_edit_access.
    $user_edit_access_tids = $user->get('field_wecc_user_edit_access')->getValue();
    if (empty($user_edit_access_tids)) {
      return AccessResult::neutral()->addCacheableDependency($user);
    }

    // Check if user has role 'content_editor'.
    if (!$account->hasRole('content_editor')) {
      return AccessResult::neutral()->addCacheContexts(['user.roles']);
    }

    // Build an array of the term ids stored in the field field_wecc_user_edit_access.
    $user_tids = [];
    foreach ($user_edit_access_tids as $user_edit_access_tid) {
      $user_tids[] = $user_edit_access_tid['target_id'];
    }
    if (in_array($node_edit_access_tid, $user_tids)) {
      return AccessResult::allowed()->addCacheableDependency($node)->addCacheableDependency($user);

    }
  }
  return AccessResult::neutral()->addCacheableDependency($node);
}


/**
 * Implements hook_form_alter().
 */
function wecc_node_access_form_alter(&$form,FormStateInterface $form_state, $form_id)
{
 // Modify form to hide field field_wecc_edit_access if user does not have role 'admin'.
  $user = \Drupal::currentUser();
  if (!$user->hasRole('administrator')) {
    if (isset($form['field_wecc_edit_access'])) {
      $form['field_wecc_edit_access']['#access'] = FALSE;
    }
  }
}
```

Finally the `web/modules/custom/wecc_node_access/wecc_node_access.info.yml` file:

```yaml
name: 'WECC Node Access'
type: module
description: 'Control edit permissions for certain nodes'
package: WECC
core_version_requirement: ^10 || ^11
```

To use this, add a term such as `Department 1` to the `wecc_edit_access` vocabulary.  Then add this term to the `field_wecc_edit_access` field on the node you want to restrict access to.  Then add the same term to the `field_wecc_user_edit_access` field on the user you want to have access to the node.  When the user does a content listing, they will see the node and be able to edit it. This also handles caching, so any changes to the terms on the node or the user are immediately reflected when the user lists the content. This will also hide the `field_wecc_edit_access` field from the user if they don't have the `administrator` role so they can't change their access permission.



## Attach a library to all forms

Use `hook_form_alter()` to attach a custom library to all forms. This is useful when you want to include custom JavaScript or CSS files on all forms. In this example, the `abc_search` library is attached to all forms.

```php
/**
 * Implements hook_form_alter().
 */
function abc_search_form_alter(&$form, \Drupal\Core\Form\FormStateInterface $form_state, $form_id) {
  // Attach the custom library to all forms.
  $form['#attached']['library'][] = 'abc_search/abc_search';
```




## Resources

- [Drupal SEO — a comprehensive Drupal self-help guide to optimise your website for search engine visibility and rankings by Suchi Garg - Sep 2023](https://salsa.digital/insights/drupal-seo-comprehensive-drupal-self-help-guide-optimise-your-website-search-engine)
- [Drupal accessibility — a comprehensive Drupal self-help guide to creating accessible websites by John Cloys -  Sep 2023](https://salsa.digital/insights/drupal-accessibility-comprehensive-drupal-self-help-guide-creating-accessible-websites)
