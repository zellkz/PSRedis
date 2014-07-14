# PSRedis - Sentinel wrapper for PHP redis clients

A PHP client for redis sentinel connections as a wrapper on other redis clients

## Installation

The easiest way to install is by using composer.  The package is available on
[packagist](https://packagist.org/packages/sparkcentral/predis-sentinel) so installing should be as easy as putting
the following in your composer file:

```
"require": {
    "sparkcentral/predis-sentinel": "*"
},
```

## Usage

### Basic example

The most basic example makes use of the Predis adapter by default.  This is the least amount of code needed to get
going with PSRedis, although we are making plans to make the configuration more concise in the future.

```php
// configure where to find the sentinel nodes

$sentinel1 = new Client('192.168.50.40', '26379');
$sentinel2 = new Client('192.168.50.41', '26379');
$sentinel3 = new Client('192.168.50.30', '26379');

// now we can configure the master name and the sentinel nodes

$masterDiscovery = new MasterDiscovery('integrationtests');
$masterDiscovery->addSentinel($sentinel1);
$masterDiscovery->addSentinel($sentinel2);
$masterDiscovery->addSentinel($sentinel3);

// discover where the master is

$master = $masterDiscovery->getMaster();
```

### Customizing the adapter

You can choose what kind of client adapter to use or even write your own.  If you write your own you need to make sure
you implement the **\PSRedis\Client\ClientAdapter** interface.

```php
// we need a factory to create the clients
$clientFactory = new PredisClientCreator();

// we need an adapter for each sentinel client too!

$clientAdapter = new PredisClientAdapter($clientFactory, Client::TYPE_SENTINEL);
$sentinel1 = new Client('192.168.50.40', '26379', $clientAdapter);

$clientAdapter = new PredisClientAdapter($clientFactory, Client::TYPE_SENTINEL);
$sentinel2 = new Client('192.168.50.41', '26379', $clientAdapter);

$clientAdapter = new PredisClientAdapter($clientFactory, Client::TYPE_SENTINEL);
$sentinel3 = new Client('192.168.50.30', '26379', $clientAdapter);

// now we can configure the master name and the sentinel nodes

$masterDiscovery = new MasterDiscovery('integrationtests');
$masterDiscovery->addSentinel($sentinel1);
$masterDiscovery->addSentinel($sentinel2);
$masterDiscovery->addSentinel($sentinel3);

// discover where the master is

$master = $masterDiscovery->getMaster();
```

### Configuring backoff

When we fail to discover the location of the master, we need to back off and try again.  The back off mechanism is
configurable and you can implement your own by implementing the **\PSRedis\Client\BackoffStrategy**

Here is an example using the incremental backoff strategy:

```php
$sentinel = new Client('192.168.50.40', '26379');
$masterDiscovery = new MasterDiscovery('integrationtests');
$masterDiscovery->addSentinel($sentinel);

// create a backoff strategy (half a second initially and increment with half of the backoff on each succesive try)
$incrementalBackoff = new Incremental(500, 1.5);
$incrementalBackoff->setMaxAttempts(10);

// configure the master discovery with this backoff strategy
$masterDiscovery->setBackoffStrategy($incrementalBackoff);

// try to discover the master
$master = $masterDiscovery->getMaster();
```

## Testing

*Note:* For testing, we still use PHPUnit 3.7 due to a bug in PhpStorm not allowing us to run unit tests from our IDE.  See
http://youtrack.jetbrains.com/issue/WI-21666

### Unit testing

We use [PHPUnit](https://github.com/sebastianbergmann/phpunit) for unit testing and [Phake](https://github.com/mlively/Phake) for mocking.  Both are installed using composer.  Running the unit tests can
be done using the following command:

```
./vendor/bin/phpunit -c ./phpunit.xml --bootstrap ./tests/bootstrap.php
```

### Integration testing

To run the integration tests, make sure you install [Vagrant](http://www.vagrantup.com).  We have used it together with
[VirtualBox](https://www.virtualbox.org).

The VM's are provisioned with [ansible](http://www.ansible.com/home).

After installing all of these, execute the following in the project root to provision the machines:

```
vagrant up
``

After that, run the integration tests with

```
./vendor/bin/phpunit -c ./phpunit.int.xml --bootstrap ./tests/bootstrap.int.php
```

You will see some warnings that need to be fixed, but the tests themselves should all pass.  The warnings are a result
of the reset of the environment just before every integration test.