# CodeReview

```php
public function __construct($basePath = null)
{
    $this->registerBaseBindings();
    $this->registerBaseServiceProviders();
    $this->registerCoreContainerAliases();
    if ($basePath) {
        $this->setBasePath($basePath);
    }
}
```

Create Application过程中做了4件事：

1.  register base bindings.
2.  register base service providers(\Illuminate\Events\EventServiceProvider and \Illuminate\Routing\RoutingServiceProvider).
3.  register core service aliases 
('app',
'auth',
'auth.driver',
'blade.compiler',
'cache',
'cache.store',
'config',
'cookie',
'encrypter',
'db',
'db.connection', 
'events',
'files',
'filesystem',
'filesystem.disk',
'filesystem.cloud',
'hash',
'translator',
'log',
'mailer', 
'auth.password',
'auth.password.broker',
'queue',
'queue.connection',
'queue.failer',
'redirect',
'redis',
'request', 
'router',
'session',
'session.store',
'url',
'validator',
'view'), and these core service will be registered later.

4.  set the base path, including 
  'path' = __DIR__ . '/app',
  'path.base' = __DIR__ ,
  'path.lang' = __DIR__ . '/resources/lang',
  'path.config' = __DIR__ . '/config',
  'path.public' = __DIR__ . '/public', 
  'path.storage' = __DIR__ . '/storage', 
  'path.database' = __DIR__ . '/database',
  'path.resources' = __DIR__ . '/resources',
  'path.bootstrap' = __DIR__ . '/bootstrap'.
  U can get theses path everywhere in the way, e.g.  public_path('/js/app.js') === __DIR__ . '/public/js/app.js';
