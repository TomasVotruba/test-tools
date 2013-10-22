Test Tools for PHP
==================

[![Build Status](https://travis-ci.org/lastzero/test-tools.png?branch=master)](https://travis-ci.org/lastzero/test-tools)

The test tools library provides the following components:

* A basic file fixture class to read and write data from/to the file system (no external dependencies)
* Self initializing database fixtures for Doctrine DBAL (record and playback)
* Test dependency injection support built on top of the Symfony DI container and PHPUnit

TestTools\TestCase\UnitTestCase.php contains an integrated DI container for more productive testing:

    use TestTools\TestCase\UnitTestCase;

    class FooTest extends UnitTestCase
    {
        protected $foo;

        public function setUp()
        {
            $this->foo = $this->get('foo');
        }

        public function testBar()
        {
            $result = $this->foo->bar('Pi', 2);
            $this->assertEquals(3.14, $result);
        }
    }

Simply create a config.yml (optionally config.local.yml for local modifications) in your base test directory,
for example

    Bundle/Example/Tests/config.yml
    
The yaml file must contain the sections "parameters" and "services". If you're not yet familiar with
dependency injection or the config file format, you can find more information in this documentation:

http://symfony.com/doc/current/components/dependency_injection/introduction.html

Since global state must be avoided while performing tests, the service instances are not 
cached between tests. The service definitions in the container are reused however. This significantly
improves test performance compared to a full container reinitialization before each test (about 5 to 10 times faster).

**Note**: A config parameter "fixture.path" is automatically set based on the test class filename and path 
to avoid conflicts/dependencies between different tests and enforce a consistent naming scheme.
The directory is created automatically.

When using a dependency injection container in conjunction with fixtures, you don't need to care about 
different environments such as development and production:
Configuration details (e.g. login credentials) must be valid for development 
environments only, since service / database requests should be replaced by fixtures from the file system after the 
corresponding tests were running for the first time. If a test fails on Jenkins or Travis CI
because of invalid URLs or credentials in config.yml, you must make sure that all code that 
accesses external resources is using fixtures (or mock objects) and that all fixtures are checked in properly.
 
You can use TestTools\Fixture\FileFixture.php to easily make any existing classes work with file based fixtures.
Have a look at the Doctrine fixture connection class (TestTools\Doctrine\FixtureConnection.php) to see an example.
It works as a wrapper (see wrapperClass option) for the standard connection class.

The basic concept of self initializing fixtures is described by Martin Fowler and can be applied to all
types of external data stores (databases) and services (SOAP/REST):

http://martinfowler.com/bliki/SelfInitializingFake.html

TestTools\TestCase\WebTestCase.php can be used for functional testing of Symfony controllers based on the 
regular DI configuration of your application:

    use TestTools\TestCase\WebTestCase;
    use Symfony\Component\DependencyInjection\ContainerInterface;

    class FooControllerTest extends WebTestCase
    {
        protected function configureFixtures(ContainerInterface $container)
        {
            // Service instance must provide a useFixtures() method for this to work
            $container->get('db')->useFixtures($this->getFixturePath());
        }

        public function testGetBar()
        {
            $this->client->getRequest('/foo/bar/Pi', array('precision' => 2));
            $response = $this->client->getResponse();
            $this->assertEquals(3.14, $response->getContent());
        }
    }

If you are using Composer, you just have to add "lastzero/test-tools" to your composer.json file:

    "require": {
        "lastzero/test-tools": ">=0.3"
    }
