# CodeReview
//??? 测试$abstract = $this->normalize($abstract);函数
//??? $this->bound($abstract) 什么作用
//??? $this->rebound($abstract); 什么作用
//??? Closure函数的用法

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

### 1. Register Base Bindings

```php
$this->instance('app', $this);
$this->instance('Illuminate\Container\Container', $this);

//instance函数解析
/**
* Register an existing instance as shared in the container.
*
* @param  string  $abstract
* @param  mixed   $instance
* @return void
*/
//$this->instance('app', $this);
//$this->instance('Illuminate\Container\Container', $this);
public function instance($abstract, $instance)
{
    //$abstract如果是string,截取右边的'\'
    //如\Illuminate\Foundation\Application => Illuminate\Foundation\Application
	
    $abstract = $this->normalize($abstract);
    if (is_array($abstract)) {
    
        //如果$abstract是数组
        //e.g. $this->instance(['\Illuminate\Foundation\Application' => 'app'], $this)
        //则app是alias name，存入Container class的$aliases[ ]属性中，这样存入值是：
        //$aliases = [
        //    'app'=> 'Illuminate\Foundation\Application',
        //];
        
        list($abstract, $alias) = $this->extractAlias($abstract);
        
        //$this->alias('Illuminate\Foundation\Application', 'app')
        $this->alias($abstract, $alias);
        //$this->aliases['app'] = 'Illuminate\Foundation\Application';
    }
    
    //unset($this->aliases['Illuminate\Container\Container'])
    unset($this->aliases[$abstract]);
    $bound = $this->bound($abstract); //???
    
    //$this->instances['app'] = $this;
    $this->instances[$abstract] = $instance;
    if ($bound) {
        $this->rebound($abstract); //???
    }
}

//1.
//...
//['\Illuminate\Foundation\Application' => 'app']
protected function extractAlias(array $definition)
{
    return [key($definition), current($definition)];
}
//$this->alias('Illuminate\Foundation\Application', 'app')
public function alias($abstract, $alias)
{
    //$this->aliases['app'] = 'Illuminate\Foundation\Application'
    $this->aliases[$alias] = $this->normalize($abstract);
}


//...
public function testReboundCallbacks() 
{
    // Arrange
    $container = new Container;
    
    // Actual
    $container->instance('app', function(){
        return 'app1';
    });
    $a = 0
    $container->rebinding('app', function() use ($a) {
        $a = 1;
    });
    //???
    // 再次绑定时，触发上一次rebinding中绑定该'app'的回调
    $container->instance('app', function () {
        return 'app2';
    });
    
    // Assert
    $this->assertEqual(1, $a);
}
```

### 2. Register Base Service Providers

```php
$this->register(new \Illuminate\Events\EventServiceProvider($this));
$this->register(new \Illuminate\Routing\RoutingServiceProvider($this));

//$this->register(new \Illuminate\Events\EventServiceProvider($this));
public function register($provider, $options = [], $force = false)
{
    if (($registered = $this->getProvider($provider)) && ! $force) {
        return $registered;
    }
    
    if (is_string($provider)) {
        $provider = $this->resolveProviderClass($provider); //???
    }

    if (method_exists($provider, 'register')) {
        $provider->register();
    }
    
    foreach ($options as $key => $value) {
        $this[$key] = $value; //???
    }

    $this->markAsRegistered($provider);

    // If the application has already booted, we will call this boot method on
    // the provider class so it has an opportunity to do its boot logic and
    // will be ready for any usage by the developer's application logics.
    if ($this->booted) {
        $this->bootProvider($provider);
    }

    return $provider;
}

//1.

if (($registered = $this->getProvider($provider)) && ! $force) {
    return $registered;
}
//...
//$this->getProvider(new \Illuminate\Events\EventServiceProvider($this))||$this->getProvider('events')
public function getProvider($provider)
{
    $name = is_string($provider) ? $provider : get_class($provider);

    return Arr::first($this->serviceProviders, function ($value) use ($name) { //???
        return $value instanceof $name;
    });
}


//2.
//...
//$provider 是Class Object类型
protected function markAsRegistered($provider)
{
    $this['events']->fire($class = get_class($provider), [$provider]); //???

    $this->serviceProviders[] = $provider;

    //$this->loadedProviders['\Illuminate\Events\EventServiceProvider']
    $this->loadedProviders[$class] = true;
}


//3.
if ($this->booted) { //??? 怎么判断的
    $this->bootProvider($provider);
}
//...
protected function bootProvider(ServiceProvider $provider)
{
    if (method_exists($provider, 'boot')) {
        return $this->call([$provider, 'boot']);
    }
}
//...
/**
 * Call the given Closure / class@method and inject its dependencies.
 *
 * @param  callable|string  $callback
 * @param  array  $parameters
 * @param  string|null  $defaultMethod
 * @return mixed
 */
//$this->call([$provider, 'boot']);
public function call($callback, array $parameters = [], $defaultMethod = null)
{
    if ($this->isCallableWithAtSign($callback) || $defaultMethod) {
        return $this->callClass($callback, $parameters, $defaultMethod);
    }

    $dependencies = $this->getMethodDependencies($callback, $parameters);

    return call_user_func_array($callback, $dependencies);
}

//...
protected function callClass($target, array $parameters = [], $defaultMethod = null)
{
    $segments = explode('@', $target);
    $method = count($segments) == 2 ? $segments[1] : $defaultMethod;

    if (is_null($method)) {
        throw new InvalidArgumentException('Method not provided.');
    }

    // 然后在这样调用call([$class, $method], $parameters)
    return $this->call([$this->make($segments[0]), $method], $parameters);
}

//...
//$dependencies = $this->getMethodDependencies($callback, $parameters);
protected function getMethodDependencies($callback, array $parameters = [])
{
    $dependencies = [];

    foreach ($this->getCallReflector($callback)->getParameters() as $parameter) { //???
        $this->addDependencyForCallParameter($parameter, $parameters, $dependencies); //???
    }

    return array_merge($dependencies, $parameters); //???
}
protected function getCallReflector($callback)
{
    if (is_string($callback) && strpos($callback, '::') !== false) {
        $callback = explode('::', $callback);
    }

    if (is_array($callback)) {
        return new ReflectionMethod($callback[0], $callback[1]); //???
    }

    return new ReflectionFunction($callback);
}

/*foreach ($this->getCallReflector($callback)->getParameters() as $parameter) { //???
    $this->addDependencyForCallParameter($parameter, $parameters, $dependencies); //???
}*/

//???
protected function addDependencyForCallParameter(ReflectionParameter $parameter, array &$parameters, &$dependencies)
{
    if (array_key_exists($parameter->name, $parameters)) {
        $dependencies[] = $parameters[$parameter->name];

        unset($parameters[$parameter->name]);
    } elseif ($parameter->getClass()) {
        $dependencies[] = $this->make($parameter->getClass()->name);
    } elseif ($parameter->isDefaultValueAvailable()) {
        $dependencies[] = $parameter->getDefaultValue();
    }
}
```

#### OK 然后看下两个service provider注册了些什么？ //这里不ServiceProvider文件不是Laravel的底层
```php
public function register()
{
    $this->app->singleton('events', function ($app) {
        return (new Dispatcher($app))->setQueueResolver(function () use ($app) {
            return $app->make('Illuminate\Contracts\Queue\Factory');
        });
    });
}
//...
public function singleton($abstract, $concrete = null)
{
    $this->bind($abstract, $concrete, true);
}

public function bind($abstract, $concrete = null, $shared = false)
{
    $abstract = $this->normalize($abstract);

    $concrete = $this->normalize($concrete);

    // 如果是数组，抽取别名并且注册到$aliases[]中，上文已经讨论
    if (is_array($abstract)) {
        list($abstract, $alias) = $this->extractAlias($abstract);

        $this->alias($abstract, $alias);
    }

    $this->dropStaleInstances($abstract);

    if (is_null($concrete)) {
        $concrete = $abstract; //$concrete = 'events'
    }

    // If the factory is not a Closure, it means it is just a class name which is
    // bound into this container to the abstract type and we will just wrap it
    // up inside its own Closure to give us more convenience when extending.
    if (! $concrete instanceof Closure) { //$concrete = 'events'
        $concrete = $this->getClosure($abstract, $concrete);
    }

    /*$bindings = [
        'events' => [
            'concrete' => function ($app) {
                            return (new Dispatcher($app))->setQueueResolver(function () use ($app) {return $app->make('Illuminate\Contracts\Queue\Factory');});
                          },
            'shared'   => true,
        ],
    ];*/
    $this->bindings[$abstract] = compact('concrete', 'shared');

    // If the abstract type was already resolved in this container we'll fire the
    // rebound listener so that any objects which have already gotten resolved
    // can have their copy of the object updated via the listener callbacks.
    if ($this->resolved($abstract)) { //???
        $this->rebound($abstract); //???
    }
}
//...
//$this->dropStaleInstances('events')
protected function dropStaleInstances($abstract)
{
    unset($this->instances[$abstract], $this->aliases[$abstract]);
}
//...
//$concrete = $this->getClosure('events', function($app){...});
protected function getClosure($abstract, $concrete)
{
    // $c 就是$container，即Container Object，会在回调时传递给这个变量
    return function ($c, $parameters = []) use ($abstract, $concrete) {
        $method = ($abstract == $concrete) ? 'build' : 'make';

        return $c->$method($concrete, $parameters);
    };
}
```

#### 看下RoutingServiceProvider中注册了些什么service ？
```php
$this->app['router'] = $this->app->share(function ($app) {
    return new Router($app['events'], $app);
});
//...
public function share(Closure $closure)
{
    return function ($container) use ($closure) {
        static $object;
 
        if (is_null($object)) {
            $object = $closure($container);
        }
 
        return $object;
    };
}
```
