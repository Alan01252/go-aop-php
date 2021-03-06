README
======

What is Go?!
-----------------

Go! is a PHP 5.4 library based on OOP and AOP paradigms.
It allows developers to add support of AOP to every PHP application.

Go! doesn't require any PECL-extentions, it neither uses any dark magic of Runkit nor evals, the library doesn't use DI-containers.
The code with weaved aspects is fully readable and native, it can be easily debugged with XDebug.
You can debug either classes or aspects.
The main advantage of Go! is that potentially it can be installed in every PHP-application,
because you don't have to change the application source code at all.
As an example, with the help of 10-20 lines of code we can intercept all the public, protected and static methods in all the classes
of application and display the name and the arguments of each method during its execution.

[![Build Status](https://secure.travis-ci.org/lisachenko/go-aop-php.png?branch=master)](https://travis-ci.org/lisachenko/go-aop-php)
[![Dependencies Status](https://d2xishtp1ojlk0.cloudfront.net/d/1416156)](http://depending.in/lisachenko/go-aop-php)
<a href="https://flattr.com/submit/auto?user_id=lisachenko&url=https://github.com/lisachenko/go-aop-php&title=Go!%20AOP%20PHP%20Framework&language=en_GB&tags=aop,php,framework,programming,library"><img align="bottom" alt="Flattr this project!" src="https://api.flattr.com/button/flattr-badge-large.png"></a>

What is AOP?
------------

[AOP (Aspect-Oriented Programming)](http://en.wikipedia.org/wiki/Aspect-oriented_programming) is an approach to cross-cutting concerns, where the concerns are designed and implemented
in a "modular" way (that is, with appropriate encapsulation, lack of duplication, etc.), then integrated into all the relevant
execution points in a succinct and robust way, e.g. through declarative or programmatic means.

In AOP terms, the execution points are called join points, a particular set of them is called a pointcut and the new
behavior that is executed before, after, or "around" a join point is called advice. You can read more about AOP in
[Introduction](http://go.aopphp.com/docs/introduction/) section.

PHP traits can be used to implement some aspect-like functionality.

Requirements
------------

Go! library is developed for PHP 5.4.0 and up, but it can partially work with PHP 5.3.0 code.

Note that PHP versions before 5.4 will not work completely, if you try to use
 aspects for code that uses Late Static Binding (LSB) feature.

Go! library will not work with eAccelerator.

Known bugs
------------

* Version 5.4.5 of PHP has a [bug #62836  Seg fault or broken object references on unserialize()](https://bugs.php.net/bug.php?id=62836).
Class advices will be always broken after unserialize().
* Version 5.4.8 of PHP has a [bug #63481  Segmentation fault caused by unserialize()](https://bugs.php.net/bug.php?id=63481). 
Interceptors for class can be unserialized incorrectly.
* Version 5.4.11 of PHP has a [bug #64070  Inheritance with Traits failed with error](https://bugs.php.net/bug.php?id=64070). 
Method interception in traits is broken, however DeclareParent advice can be used correctly.

Installation
------------

Go! library can be installed with composer or manually with git submodules. Installation is quite easy:

1. Download go-aop-php using composer or git submodule
2. Create an application aspect kernel
3. Configure the aspect kernel in the front controller
4. Adjust the front controller of your application for proxying autoloading requests to the aspect kernel
5. Create an aspect
6. Register the aspect in the aspect kernel

### Step 1: Download go-aop-php using composer or git submodule

Add Go! AOP library to your composer.json:

```js
{
    "require": {
        "lisachenko/go-aop-php": "*"
    }
}
```

Now ask composer to download the library with its dependencies by running the command:

``` bash
$ php composer.phar update lisachenko/go-aop-php
```

Composer will install the library to your project's `vendor/lisachenko/go-aop-php` directory.

Alternative way for installing the library is to use git submodules:

``` bash
git submodule add https://github.com/lisachenko/go-aop-php vendor/lisachenko/go-aop-php
```

Dependencies should be also installed as submodules:
``` bash
git submodule add https://github.com/Andrewsville/PHP-Token-Reflection vendor/andrewsville/php-token-reflection
git submodule add https://github.com/doctrine/common vendor/doctrine/common (use 2.3.x tags)
git submodule add https://github.com/jakubledl/dissect vendor/jakubledl/dissect
```

### Step 2: Create an application aspect kernel

The aim of this library is to provide easy AOP integration to your application.
Your first step then is to create the `AspectKernel` class
for your application. This class will manage all aspects of your
application in one place.

The library provides base class to make it easier to create your own kernel.
To create your application kernel extend the abstract class `Go\Core\AspectKernel`

``` php
<?php
// app/ApplicationAspectKernel.php

use Go\Core\AspectKernel;
use Go\Core\AspectContainer;

/**
 * Application Aspect Kernel
 */
class ApplicationAspectKernel extends AspectKernel
{

    /**
     * Configure an AspectContainer with advisors, aspects and pointcuts
     *
     * @param AspectContainer $container
     *
     * @return void
     */
    protected function configureAop(AspectContainer $container)
    {
    }
}
```

### 3. Configure the aspect kernel in the front controller

To configure the aspect kernel, call `init()` method of kernel instance.

``` php
// front-controller, for Symfony2 application it's web/app_dev.php

// Do not use application autoloader for the following files
include __DIR__ . '/../vendor/lisachenko/go-aop-php/src/Go/Core/AspectKernel.php';
include __DIR__ . '/../app/ApplicationAspectKernel.php';

// Initialize an application aspect container
$applicationAspectKernel = ApplicationAspectKernel::getInstance();
$applicationAspectKernel->init(array(
        'appLoader'     => __DIR__ . '/autoload.php',
        // Application root directory
        'appDir'        => __DIR__ . '/../',
        // Cache directory
        'cacheDir'      => __DIR__ . '/path/to/cache/for/aop',
        // Configuration for autoload namespaces
        'autoloadPaths' => array(
            'Go'               => __DIR__ . '/../vendor/lisachenko/go-aop-php/src/',
            'TokenReflection'  => __DIR__ . '/../vendor/andrewsville/php-token-reflection/',
            'Doctrine\\Common' => __DIR__ . '/../vendor/doctrine/common/lib/',
            'Dissect'          => __DIR__ . '/../vendor/jakubledl/dissect/src/'
        ),
        // Include paths restricts the directories where aspects should be applied, or empty for all source files
        'includePaths' => array(
            __DIR__ . '/../src/'
        )
));
```

Look at the key `appLoader` in the config. It must return the path to the application
loader file, typically `autoload.php` or `bootstrap.php`. Aspect kernel will delegate control to that
file for registering an autoloader for application classes.

### 4. Adjust the front controller of your application for proxying autoloading requests to the aspect kernel

At this step we have a configured aspect kernel and can try to switch to it instead of default autoloader:

``` php
// front-controller, for Symfony2 application it's web/app_dev.php

// Comment default loader of application at the top of file
// include __DIR__ .'/../vendor/autoload.php'; // for composer
// include __DIR__ .'/../app/autoload.php';  // for old applications
```

### 5. Create an aspect

Aspect is the key element of AOP philosophy. And Go! library just uses simple PHP classes for declaring aspects!
Therefore it's possible to use all features of OOP for aspect classes.
As an example let's intercept all the methods and display their names:

``` php
// Aspect/MonitorAspect.php

namespace Aspect;

use Go\Aop\Aspect;
use Go\Aop\Intercept\FieldAccess;
use Go\Aop\Intercept\MethodInvocation;
use Go\Lang\Annotation\After;
use Go\Lang\Annotation\Before;
use Go\Lang\Annotation\Around;
use Go\Lang\Annotation\Pointcut;

/**
 * Monitor aspect
 */
class MonitorAspect implements Aspect
{

    /**
     * Method that will be called before real method
     *
     * @param MethodInvocation $invocation Invocation
     * @Before("execution(public Example->*(*))")
     */
    public function beforeMethodExecution(MethodInvocation $invocation)
    {
        $obj = $invocation->getThis();
        echo 'Calling Before Interceptor for method: ',
             is_object($obj) ? get_class($obj) : $obj,
             $invocation->getMethod()->isStatic() ? '::' : '->',
             $invocation->getMethod()->getName(),
             '()',
             ' with arguments: ',
             json_encode($invocation->getArguments()),
             "<br>\n";
    }
}
```

Easy, isn't it? We declared here that we want to install a hook before the execution of
all dynamic public methods in the class Example. This is done with the help of annotation
`@Before("execution(public Example->*(*))")`
Hooks can be of any types, you will see them later.
But we doesn't change any code in the class Example! I can feel you astonishment now )

### 6. Register the aspect in the aspect kernel

To register the aspect just add an instance of it in the `configureAop()` method of the kernel:

``` php
// app/ApplicationAspectKernel.php

use Aspect/MonitorAspect;

//...

    protected function configureAop(AspectContainer $container)
    {
        $container->registerAspect(new MonitorAspect());
    }

//...
```

Now you are ready to use the power of aspects! Feel free to change anything everywhere.

Documentation
-------------

Documentation about Go! library can be found at [official site][1].

[1]: http://go.aopphp.com
