# Laravel HAL Doctrine

This project is a hydrator for [laravel-hal](https://github.com/API-Skeletons/laravel-hal)
for Doctrine.  Instead of manually creating every hydrator for your entities, this library
will introspect an entity or collection and generate the HAL for it including links to 
other collections for one to many relationships and embedding many to one and one to one
entities.

A grouped minimal configuration file will allow for excluded fields and associations and
configure the routes for each entity.


## Use

### Create a hydrator manager

```php
namespace App\HAL;

use ApiSkeletons\Laravel\HAL\HydratorManager as HALHydratorManager;

final class HydratorManager extends HALHydratorManager
{
    public function __construct() 
    {
        $this->classHydrators = [
            \App\ORM\Entity\Artist::class => \ApiSkeletons\Laravel\HAL\Doctrine\DoctrineHydrator::class,
        ];
    }
}
```

### Extract an entity
```php
use App\Hal\HydratorManager;

public function fetch(Entity\Artist $artist)
{
    $hydratorManager = new HydratorManager();
    return $hydratorManager->extract($artist);
}
```

will result in a HAL response like 
```json
{
  "_links": {
    "self": {
      "href": "https://website/artist/1"
    }
    "album": {
      "href": "https://website/album?filter[artist]=1"
    }
  },
  "id": 1,
  "name": "Grateful Dead"
}
```


## Configuration

```php
$config = [
    'default' => [
        'entityManager' => EntityManager::class,
        'routeNamePatterns' => [
            'entity' => 'api.{name}::fetch',
            'collection' => 'api.{name}::fetchAll',
        ],
        'entities' => [
            \App\ORM\Entity\Artist::class => [
                // Override route patterns
                'routesNames' => [
                    'entity' => 'artist::fetch',
                    'collection' => 'artist::fetchAll',
                ],
                // List of fields and associations to exclude
                'exclude' => [
                    'alias',
                ],
            ],
            /**
             * Every entity which should be extracted, whether on
             * its own or as a relationship, must be listed in the
             * config even if its configuration is emtpy.  Entities 
             * discovered in the metadata through relationships which
             * are not in the configuration will be excluded.
             */
            \App\ORM\Entity\Album::class => [],
        ],
    ],
];
```

## Naming strategies

The naming strategy for turning an entity name into a HAL identifier is configurable and
you can write your own naming strategy if desired.
