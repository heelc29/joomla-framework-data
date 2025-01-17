## The Data Package [![Build Status](https://ci.joomla.org/api/badges/joomla-framework/data/status.svg?ref=refs/heads/3.x-dev)](https://ci.joomla.org/joomla-framework/data)

[![Latest Stable Version](https://poser.pugx.org/joomla/data/v/stable)](https://packagist.org/packages/joomla/data)
[![Total Downloads](https://poser.pugx.org/joomla/data/downloads)](https://packagist.org/packages/joomla/data)
[![Latest Unstable Version](https://poser.pugx.org/joomla/data/v/unstable)](https://packagist.org/packages/joomla/data)
[![License](https://poser.pugx.org/joomla/data/license)](https://packagist.org/packages/joomla/data)

### `Data\DataObject`

`Data\DataObject` is a class that is used to store data but allowing you to access the data by mimicking the way PHP handles class properties. Rather than explicitly declaring properties in the class, `Data\DataObject` stores virtual properties of the class in a private internal array. Concrete properties can still be defined but these are separate from the data.

#### Construction

The constructor for a new `Data\DataObject` object can optionally take an array or an object. The keys of the array or the properties of the object will be bound to the properties of the `Data\DataObject` object.

```php
use Joomla\Data\DataObject;

// Create an empty object.
$object1 = new DataObject;

// Create an object with data. You can use an array or another object.
$data = array(
    'foo' => 'bar',
);

$object2 = new DataObject($data);

// The following should echo "bar".
echo $object2->foo;
```

#### General Usage

`Data\DataObject` includes magic getters and setters to provide access to the internal property store as if they were explicitly declared properties of the class.

The `bind` method allows for injecting an existing array or object into the `Data\DataObject` object.

The `dump` method gets a plain `stdClass` version of the `Data\DataObject` object's properties. It will also support recursion to a specified number of levels where the default is 3 and a depth of 0 would return a `stdClass` object with all the properties in native form. Note that the `dump` method will only return virtual properties set binding and magic methods. It will not include any concrete properties defined in the class itself.

The `JsonSerializable` interface is implemented. This method proxies to the `dump` method (defaulting to a recursion depth of 3). Note that this interface only takes effect implicitly in PHP 5.4 so any code built for PHP 5.3 needs to explicitly use either the `jsonSerialize` or the `dump` method before passing to `json_encode`.

The `Data\DataObject` class also implements the `IteratorAggregate` interface so it can easily be used in a `foreach` statement.

```php
use Joomla\Data\DataObject;

// Create an empty object.
$object = new DataObject;

// Set a property.
$object->foo = 'bar';

// Get a property.
$foo = $object->foo;

// Binding some new data to the object.
$object->bind(array('goo' => 'car');

// Get a plain object version of the data object.
$stdClass = $object->dump();

// Get a property with a default value if it is not already set.
$foo = $object->foo ?: 'The default';

// Iterate over the properties as if the object were a real array.
foreach ($object as $key => $value)
{
    echo "\n$key = $value";
}

if (version_compare(PHP_VERSION, '5.4') >= 0)
{
	// PHP 5.4 is aware of the JsonSerializable interface.
	$json = json_encode($object);
}
else
{
	// Have to do it the hard way to be compatible with PHP 5.3.
	$json = json_encode($object->jsonSerialize());
}
```

### `Data\DataSet`

`Data\DataSet` is a collection class that allows the developer to operate on a list of `Data\DataObject` objects as if they were in a typical PHP array (`Data\DataSet` implements the `ArrayAccess`, `Countable` and `Iterator` interfaces).

#### Construction

A typical `Data\DataSet` object will be instantiated by passing an array of `Data\DataObject` objects in the constructor.

```php
use Joomla\Data\DataObject;
use Joomla\Data\DataSet;

// Create an empty object.
$players = new DataSet(
    array(
        new DataObject(array('race' => 'Elf', 'level' => 1)),
        new DataObject(array('race' => 'Chaos Dwarf', 'level' => 2)),
    )
);
```

#### General Usage

Array elements can be manipulated with the `offsetSet` and `offsetUnset` methods, or by using PHP array nomenclature.

The magic `__get` method in the `Data\DataSet` class effectively works like a "get column" method. It will return an array of values of the properties for all the objects in the list.

The magic `__set` method is similar and works like a "set column" method. It will set all a value for a property for all the objects in the list.

The `clear` method will clear all the objects in the data set.

The `keys` method will return all of the keys of the objects stored in the set. It works like the `array_keys` function does on an PHP array.

```php
use Joomla\Data\DataObject;

// Add a new element to the end of the list.
$players[] => new DataObject(array('race' => 'Skaven', 'level' => 2));

// Add a new element with an associative key.
$players['captain'] => new DataObject(array('race' => 'Human', 'level' => 3));

// Get a keyed element from the list.
$captain = $players['captain'];

// Set the value of a property for all objects. Upgrade all players to level 4.
$players->level = 4;

// Get the value of a property for all object and also the count (get the average level).
$average = $players->level / count($players);

// Clear all the objects.
$players->clear();
```

`Data\DataSet` supports magic methods that operate on all the objects in the list. Calling an arbitrary method will iterate of the list of objects, checking if each object has a callable method of the name of the method that was invoked. In such a case, the return values are assembled in an array forming the return value of the method invoked on the `Data\DataSet` object. The keys of the original objects are maintained in the result array.

```php
use Joomla\Data\DataObject;
use Joomla\Data\DataSet;

/**
 * A custom data object.
 *
 * @since  1.0
 */
class PlayerObject extends DataObject
{
    /**
     * Get player damage.
     *
     * @return  integer  The amount of damage the player has received.
     *
     * @since   1.0
     */
    public function hurt()
    {
        return (int) $this->maxHealth - $this->actualHealth;
    }
}

$players = new DataSet(
    array(
        // Add a normal player.
        new PlayerObject(array('race' => 'Chaos Dwarf', 'level' => 2,
        	'maxHealth' => 40, 'actualHealth' => '32')),
        // Add an invincible player.
        new PlayerObject(array('race' => 'Elf', 'level' => 1)),
    )
);

// Get an array of the hurt players.
$hurt = $players->hurt();

if (!empty($hurt))
{
    // In this case, $hurt = array(0 => 8);
    // There is no entry for the second player
    // because that object does not have a "hurt" method.
    foreach ($hurt as $playerKey => $player)
    {
        // Do something with the hurt players.
    }
};
```

### `Data\DumpableInterface`

`Data\DumpableInterface` is an interface that defines a `dump` method for dumping the properties of an object as a `stdClass` with or without recursion.

## Installation via Composer

Add `"joomla/data": "~3.0"` to the require block in your composer.json and then run `composer install`.

```json
{
	"require": {
		"joomla/data": "~3.0"
	}
}
```

Alternatively, you can simply run the following from the command line:

```sh
composer require joomla/data "~3.0"
```

If you want to include the test sources, use

```sh
composer require --prefer-source joomla/data "~3.0"
```
