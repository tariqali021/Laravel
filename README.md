
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
 
### Modular approach in Laravel
- Means subdividing your projects in smaller parts.
- Usefull for larger applications because each feature/module has it's own folder containing controllers, models, routes, views.
- Usefull if you want to reuse that feature/module in some other application.
- Done via autoloaidng Module folder or registering views, configurations, controllers manually.
- Done via any pre developed packages.

### Laravel Jetstream
- Designed application Starter kit for fresh laravel application.
- This provides the implementation for your application's login, registration, email verification, two-factor authentication, session management, API via Laravel Sanctum, and optional team management features.

### Laravel Telescope
- Laravel Telescope is an elegant debug assistant for the Laravel framework.
- provides insight into the requests coming into your application, exceptions, log entries, database queries, queued jobs, mail, notifications, cache operations, scheduled tasks, variable dumps and more.

### Laravel Socialite
-  Laravel Socialite provides a simple, convenient way to authenticate with OAuth providers like Facebook, Twitter, LinkedIn, Google, GitHub, GitLab, and Bitbucket.

### Laravel Scout
- Laravel Scout provides a simple, driver based solution for adding full-text search to your Eloquent models. Using model observers.
- This will automatically keep your search indexes in sync with your Eloquent records.
- Currently, Scout ships with an _Algolia driver_ however, writing custom drivers is simple and you are free to extend Scout with your own search implementations.
- **Full Text Search** is a comprehensive search method that compares every word of the search request against every word within the document or database. It lets the user find a word or phrase anywhere within the database or document.  A full-text query returns any documents that contain at least one word match.
- **Algolia** is a hosted search engine capable of delivering real-time results from the first keystroke. 

### Laravel Passport
- Laravel Passport provides a full OAuth2 server implementation for your Laravel application.
- If your application absolutely needs to support OAuth2, then you should use Laravel Passport.
- if you are attempting to authenticate a single-page application, mobile application, or issue API tokens, you should use Laravel Sanctum. Laravel Sanctum does not support OAuth2

### Laravel Envoyer (Deployment Management)
- Envoyer is Platform as a Service (PaaS) to deploy PHP and Laravel applications with zero downtime.
- Easy rollbacks in case of any crash while deployment.

### Laravel Forge (Server Management)

- Laravel Forge is Deployment as a Service.
- will go offline while deployemnt.(No zero downtime)
- Laravel Forge is a server management and site deployment service. This provides a GUI for server management.
- It can be used to automate the deployment of any web application that uses a PHP server.
- Instead of installing each component like NGINX, MySQL, Redis and PHP to run web app, You can automate all these installations & configurations using Laravel Forge.
- You will manually scale servers.
- Forge can do a variety of things: add sub-domains, install SSL certificates, create queue workers, create Cron jobs, etc.
- This can create and manage servers on the following server providers like DigitalOcean, AWS.
- Forge also supports the ability to use your own custom server. There is an option for that _Custom VPS_.

### Laravel Vapor 

- Laravel Vapor is an auto-scaling, serverless deployment platform for Laravel, powered by `AWS Lambda`.
- You can manage your Laravel infrastructure on Vapor.
- Some features are Auto-scaling, Zero-downtime deployments, Redis, Database & DNS Management, File uploads on S3.

**NOTES**
- _Create Vapor Account_ before integrating Vapor into your application. 
- _Install Vapor CLI_ to deploy your Laravel Vapor applications using the Vapor CLI.
- _Install Vapor core package_ that contains various Vapor runtime files and a service provider to allow your application to run on Vapor. 
- _Link with AWS_ using an an active AWS account on your team's settings management page in order to deploy projects or create other resources using Vapor.

### Laravel Homestead (Development Envirenment)
- _Vagrant_ is an open-source software product for building and maintaining portable virtual software development environments; e.g., for VirtualBox, KVM, Hyper-V, Docker containers, VMware, and AWS
- _Vagrant Boxes_ are the package format for Vagrant environments. A box can be used by anyone on any platform that Vagrant supports to bring up an identical working environment.
- _Laravel Homestead_ is a pre-packaged Vagrant box that provides you a wonderful development environment without requiring you to install PHP, a web server, and any other server software on your local machine. 

**Serverless**
Serverless computing (or serverless for short), is an execution model where the cloud provider (AWS, Azure, or Google Cloud) is responsible for executing a piece of code by dynamically allocating the resources. And only charging for the amount of resources used to run the code. The code is typically run inside stateless containers that can be triggered by a variety of events including http requests, database events, queuing services, monitoring alerts, file uploads, scheduled events (cron jobs), etc. The code that is sent to the cloud provider for execution is usually in the form of a function.

**AWS Lambda** is a serverless compute service that lets you run code without provisioning or managing servers.
**OAuth 2.0** is the industry-standard protocol for authorization.


**These are quick notes to revise optimization techniques. Full explanation can be found a https://geekflare.com/laravel-optimization/**

### PHP App Optimization
- optimization can be done at foru level.
- **Lnaguage level** means use faster version of the language & avoid the coding style feature that makes the code slow.
- **Framework level** means framework architecture or features specific.
- **Infrastructure level** means PHP process manager, database, web server etc.
- **Hardware level** means hosting provider where your app is hosted.


### Laravel App Optimization
- **Be aware of n+1 database queries.** Means you get the models & then loop through the models to get it's relaion model that will execute individual query in loop.
This is also called as **lazy loading**. To optimize this use **eager loading** using method **with('*relation_model*')** that will get the related models in single query using **joins**. 

- **Cache the configuration.** Means laravel boots everything & parse all configuration files on each request. Loading configurations on each request will slow the request time.
To optimize this use **php artisan config:cache** command that will combine all configuration files into one config file. Larvel will read only this config file on each request 
& hence it's faster than reading all configuration files. By using this approach, **env()** method will return null except the config files so be aware of that. To undo this behaviour use command **php artisan cache:clear**.

- **Reduce autoloaded services.** Means laravel loads a ton of services from **config/app.php** on each request. You can optimize request by commenting out unneeded services.
This is suitable for API's where we don't need AuthServiceProvider, ViewServiceProvider etc.

- **Be wise with middleware stacks.** Means be aware of the middleware stacks like **web, api** in **app/Http/kernel.php**. If you have large application, these middlware become
silent burden for the request if there's no business reason for them. You can optimize this by applying selectively middlwares to requests if possible because adding something globally is convenient but there can be performance issue.

- **Avoid the ORM (at times).** Means laravel do alot work hen you query using the eloquent model. It will create new models & set all the attributes for each model when we query forlarge no of requests.
1000 records means 100 models with all attributes set. You can optimize this using **DB::raw()** for larger or complex queries.

- **Use caching as much as possible.** Means it will take same time each time request is for same action. You can optimize this by caching the repeated or static queries results using laravel built-in caching feature. caching means precomputing and storing expensive results (expensive in terms of CPU and memory usage),
and simply returning them when the same query is repeated.

- **Prefer in-memory caching.**. Means using cache can optimize request time. Using laravel built-in caching system will store the cached result in the file if you do not use any in-memory cache like Redis, Memcached. You can optimize 
request or query results time using in-memory caching system that stores the results in RAM. RAM is 10-20 times faster than SSD that's why still in-memory caching is preferable. 100,000 read operations per second are common using Redis.

- **Cache the routes.** Means this is the same as caching config. Request time increases when laravel read from route files if you have no. of route files. You can optimize this by caching the routes that will end up with single route file. 
Use **php artisan route:cache** to cache the routes & undo this using command **php artisan route:clear**.

- **Using Autoloader optimization.** Means it takes time to find and include classes by a given namespace string takes time. You can optimize this on production using command **composer install --optimize-autoloader --no-dev**

- **Using Queues.** Means it will take time to send a notification to users via email, sms etc. You can optimize this using laravel built-in queue feature that will queue all notification requests & execute the one by one. 

- **Asset optimization (Laravel Mix).** Means use laravel mix feature to minify & bundle frontend resources like css, js files into single file.


