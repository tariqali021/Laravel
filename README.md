
#### SERVICE CONTAINER

- DI is the ability to swap implementations of the injected class. useful during testing
- an approach for managing class dependencies and performing dependency injection.
- injected class can be bind in the service provider to tell which class instance to return
- useful to bind an interface with any class/service implementation.
- used to resolve sub-dependencies of the object we are building.  
- resolving means returning an object of class with all it's dependencies. This is done by service container that keeps all the classes with their dependencies.
- dependencies are resolved either by `make/resolve` methods or by typ-hinting in controller, events etc.

**Binding (simple, singleton, instance)**
- You can bind 
    - simply a class using `bind('injected_class', 'returned_class or return_from_closure')`
    - a singleton using `singleton('injected_class', 'return_class or return_from_closure')`
    - an instance using `instance('injected_class', 'instance_of_class')`

````php 
$this->app->bind( 'App\Contracts\Logger' , 'App\Services\FileLogger')

// This binding means that whenever "Logger" is injected in any class implementation it will return the object of "FileLogger"

public function __construct(App\Contracts\Logger $logger){
    // $logger is the instance of FileLogger class here
}
````
**Contextual Binding**
- means to inject same class but with different implementation

````php
$this->app->when(PhotoController::class)
    ->needs(Filesystem::class)
    ->give(function () {
        return Storage::disk('local');
    });

$this->app->when(VideoController::class)
    ->needs(Filesystem::class)
    ->give(function () {
        return Storage::disk('s3');
    });
````

**Tagging**
- means to bind an array of different interfaces/classes using `tage('array_of_intefaces', 'user_defined_tag_name')` then use that tag using `return new AnyClass($app->tagged('user_defined_tag_name'));`

**Resolve Binding**
- Atter all bindings registered in service provider, you can resolve class instance from container using `$app->method` `make('name_of_class_or_interface')` or helper `resolve('name_of_class_or_interface')`


### SERVICE PROVIDER

- ServiceProviders are the simple classes that are used to register things like registering service_containers, events, middlewares etc for the framework.
- This is a central place for your application that decides bindings for the service being provided and
  boot all registered services.           
- This has 2 methods
    - `register` used for binding classes in service container
    - `boot` is called after all services in `register` method have been registered. Here you can also type hint services         

**Registering Provider**
- service provider can be registered in `config/app.php` file's `providers` array. (Eager Loaded on every request)

**Deferred Provider**
- Deffer the provider (Lazy Load) if just there are only bindings in service provider. This will improve performance. Laravel will load this provider only when you resolve thsi service
- The `provider` method is used for deffered laoding which will return array_of_service_container_bindings registered by this provider

### FACADES

- Facades provide a "static" interface to classes that are available in the application's service container. 
- Facades serve as "static proxies", provides short syntax
- Facades use dynamic methods to proxy method calls to objects resolved from the service container

**How it works**
- Facade is a class that provides access to an object from the container.
- The Facade base class makes use of the `__callStatic()` magic-method to defer calls from your facade to an object resolved from the container. 
- Facade class defines the method getFacadeAccessor(). This method's job is to return the name of a service container binding. 
- When a user references any static method on the Cache facade, Laravel resolves the cache binding from the service container and runs the requested method against that object.

### AUTOLOADING

- to load files automatically from storage when needed
- when you use a class in your application, the autoloader checks if it’s already loaded, and if not, the autoloader loads the necessary class into memory. So the class is loaded on the fly where it’s needed—this is called autoloading
- When you’re using autoloading, you don’t need to include all the library files manually; you just need to include the autoloader file which contains the logic of autoloading, and the necessary classes will be included dynamically.
-  without autoloading you would need to use `require` or `include` to include files

**Autoloading using Composer**
- Using composer you can autoload via composer.json file in the root of project. Composer provides four different methods for autoloading files:
    - file autoloading
    - classmap autoloading
    - PSR-0 autoloading
    - PSR-4 autoloading
    
### REQUEST LIFE CYCLE

- First the requests are directed to `public/index.php` by your web server (Apache / Nginx) configuration
- Next The `index.php` file loads the Composer generated autoloader definition, and then retrieves an instance of the Laravel application from `bootstrap/app.php`.
- Next, the incoming request is sent to either the HTTP kernel or the console kernel, depending on the type of request that is entering the application. (kernels serve as the central location that all requests flow)
- The HTTP kernel extends the `Illuminate\Foundation\Http\Kernel` class, which defines an array of bootstrappers that will be run before the request is executed. These bootstrapers load envirenment & configurations & srvice providers, register facades & providers and boot providers. Then the request passes through the `middlewares` defined in `kernel.php` file like session middleware, check maitenance mode, verify CSRF token,
- Next `kernel` loads the service providers configured in `config/app.php` configuration file's `providers` array. The `register` method of each provider will be called. Then, once all of the providers have been registered, the `boot` method will be called on each provider. By calling the `register` method of every service provider first, service providers may depend on every container binding being registered and available by the time the `boot` method is executed. The `RouteServiceProvider` loads route files.
- Next the request will be handed off to the router for dispatching. Also the router will run any route specific middleware.
- Next after passing through middlewares, router or controller method will be executed that will return response.
- Next the response will travel back through router middleware by giving a chance to modify response, the HTTP kernel's handle method returns the response object and the `index.php` file calls the `send` method on the returned response. The `send` method sends the response content to the user's web browser.
 

