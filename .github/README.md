Invoker

Generic and extensible callable invoker.

￼

Why?

Who doesn't need an over-engineered call_user_func()?

Named parameters

Does this Silex example look familiar:

$app->get('/project/{project}/issue/{issue}', function ($project, $issue) { // ... }); 

Or this command defined with Silly:

$app->command('greet [name] [--yell]', function ($name, $yell) { // ... }); 

Same pattern in Slim:

$app->get('/hello/:name', function ($name) { // ... }); 

You get the point. These frameworks invoke the controller/command/handler using something akin to named parameters: whatever the order of the parameters, they are matched by their name.

This library allows you to invoke callables with named parameters in a generic and extensible way.

Dependency injection

Anyone familiar with AngularJS is familiar with how dependency injection is performed:

angular.controller('MyController', ['dep1', 'dep2', function(dep1, dep2) { // ... }]); 

In PHP we find this pattern again in some frameworks and DI containers with partial to full support.

PHP-DI provides a way to invoke a callable and resolve all dependencies from the container using type-hints:

$container->call(function (Logger $logger, EntityManager $em) { // ... }); 

This library provides clear extension points to let frameworks implement any kind of dependency injection support they want.

TL/DR

In short, this library is meant to be a base building block for calling a function with named parameters and/or dependency injection.

Installation

composer require PHP-DI/invoker 

Usage

Default behavior

By default the Invoker can call using named parameters:

$invoker = new Invoker\Invoker; $invoker->call(function () { echo 'Hello world!'; }); // Simple parameter array $invoker->call(function ($name) { echo 'Hello ' . $name; }, ['John']); // Named parameters $invoker->call(function ($name) { echo 'Hello ' . $name; }, [ 'name' => 'John' ]); // Use the default value $invoker->call(function ($name = 'world') { echo 'Hello ' . $name; }); // Invoke any PHP callable $invoker->call(['MyClass', 'myStaticMethod']); // Using Class::method syntax $invoker->call('MyClass::myStaticMethod'); 

Dependency injection in parameters is supported but needs to be configured with your container.

Additionally, callables can also be resolved from your container.

Parameter resolvers

Extending the behavior of the Invoker is easy and is done by implementing a ParameterResolver.

This is explained in detail in the Parameter resolvers documentation.

Built-in support for dependency injection

Rather than have you re-implement support for dependency injection with different containers every time, this package ships with 2 optional resolvers:

TypeHintContainerResolver

This resolver will inject container entries by searching for the class name using the type-hint.

ParameterNameContainerResolver

This resolver will inject container entries by searching for the name of the parameter.

These resolvers can work with any dependency injection container compliant with PSR-11.

Resolving callables from a container

The Invoker can be wired to your DI container to resolve the callables.

Security note: Do not pass untrusted user input directly to Invoker::call() when callable resolution through a container is enabled. Applications should validate or allowlist callable and class names before invoking them. A callable that can be selected by untrusted input may result in unintended application code being executed with the privileges available to the application.

For example with an invokable class:

class MyHandler { public function __invoke() { // ... } } $invoker->call('MyHandler'); 

If the container is configured:

$invoker = new Invoker\Invoker(null, $container); $invoker->call('MyHandler'); 

The same applies to class methods:

class WelcomeController { public function home() { // ... } } $invoker = new Invoker\Invoker(null, $container); $invoker->call(['WelcomeController', 'home']); // Alternatively: $invoker->call('WelcomeController::home'); 

Applications using this feature as a framework dispatcher should ensure that externally supplied callable names are validated against an explicit allowlist or otherwise constrained to intended application handlers.

Again, any PSR-11 compliant container can be provided.

