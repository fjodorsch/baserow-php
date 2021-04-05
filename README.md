
# Baserow PHP client
A PHP client for the Baserow.io API with human readable table and field names.

Comments, requests or bug reports are appreciated.

## Getting started

This library only manages CRUD operations using the Baserow API.

It can be used with both the SaaS hosted and self hosted versions.

For more information about the Baserow API, please see:

- https://baserow.io/api/docs
- https://api.baserow.io/api/redoc/

---

### Installation

If you're using Composer, you can run the following command:
```
composer require "stevecomrie/baserow-php:main-dev"
```
You can also download the src files directly and extract them to your web directory.


### Add the client to your project
If you're using Composer, run the autoloader
```php
require 'vendor/autoload.php';
```
Or include the Baserow.php file

```php
include('../src/Baserow.php');
include('../src/Request.php');
include('../src/Response.php');
```
### Initialize the class
Only the `api_key` parameter is required, the rest are added as a reference.

You can create your API key through your Baserow.io dashboard.

```php
use \Scomrie\Baserow\Baserow;

$baserow = new Baserow([
    'api_key' => 'API_KEY', // REQUIRED!!

	// if you are self hosting your own instance of Baserow, use this parameter to
	// point to the proper location on your server. defaults to the SaaS hosted endpoint.
    'api_url' => 'https://api.baserow.io/api/database/rows/table'

	// if set to true, will dump any errors with print_r() and exit on failure
	'debug' => false,

	// map of all tables & fields from auto-generated ###'s to human readable names
	'table_map' => []
]);
```

### Creating a human readable table map

Baserow uses incrementing integers for names when creating new tables and fields.

For example, tables are given IDs like:
 - 10001
 - 10002
 - 10003, etc

And fields within a table are given names like:
 - field_1001
 - field_1002
 - field_1003, etc

This is obviously not ideal for long-term code readability and maintenance, but totally understandable given the current stage of Baserow development.

To make developer life a littler easier, you can configure a `table_map` parameter when creating a new instance of the Baserow client that lets you map these auto-generated IDs to human readable names.

#### Sample table map

This is what a sample  table map configuration might look like.

```
$table_map = [
	'Customers'  => [ 10001, [
		'firstName' =>  'field_1001',
		'lastName'  =>  'field_1002',
		'active'    =>  'field_1003',
		'projects'  =>  'field_1004',
	]],

	'Projects' => [ 10002, [
		'name'     => 'field_1005',
		'dueDate'  => 'field_1006',
		'customer' => 'field_1007',
	]],
];
```

You can find the table and field IDs generated by Baserow using the [API documentation for your database](https://baserow.io/api/docs/).

When you initialize the Baserow client with a `table_map` you are able to use your new human readable names anywhere that you would have previously used a table #### or field_####.

The `table_map` parameter is optional, but the rest of the examples in this document will assume that you're using it.

#### Table map recommendations

You probably don't want to use spaces in your human readable field names.

Your table map configuration will need to be manually updated if you add or remove fields.

In addition to maintaining your sanity when writing code, the table map configuration also make it possible to create a staging / production environment using two separate databases, each with their own table map.


## Communicating with the API

### Retrieving a "page" of records from a table
Get entries from the table "Contacts". 
```php
$response = $baserow->list( 'Contacts' );

print_r($response);
```


### Using $params to filter & sort records

You don't have to use all the params, they are added as a reference.

```php
$params = array(
    "page" => 1,
    "size" => 100,
    "order_by" => "lastName",
    "filter__firstName__equal" => "Steve",
    "include" => "firstName,lastName",
);

$response = $baserow->list( 'Contacts', $params );
print_r($response);
```
The full list of available parameters for use with the `list()` or `all()` functions can be found within the [API documentation](https://baserow.io/api/docs/) that is automatically generated by Baserow for your database.


### Retrieving ALL records within a table
If you need to return more than a single page of results, you can use the `all()` function to automatically retrieve the entire paginated result set.
```php
$response = $baserow->all( 'Contacts', $params );
print_r($response);
```


### Creating a new record
We will create new entry in the table Contacts
```php
$newContact = [
	'firstName' => "Steve",
	'lastName'  => "Comrie",
	'active'    => true,
	'projects'  => [ 2 ],
];

if( $contact = $baserow->create( "Contacts", $newContact ) ) {
	echo "Created new contact: " . $contact->id . "\n";
} else {
	print_r( $baserow->error() );
}
```

### Updating an existing record
Use the row ID to update the entry
```php
$contactID = 1;
$updatedFields = [
	'firstName' => "Steven",
	'active'    => false,
];

if( $baserow->update( "Contacts", $updatedFields, $contactID ) ) {
	echo "Contact updated!\n";
}
```

### Deleting a record
Use the row ID to delete the entry
```php
$deleteContactID = 4;
if( $baserow->delete("Contacts", $deleteContactID ) ) {
	echo "Contact deleted!\n";
}
```
### Accessing API error messages
Any time an API operation fails, you can access the reason for the failure using the `error()` function. 
```php
$error = $baserow->error();
print_r( $error );
```


## Credits

Copyright (c) 2021 - Programmed by Steve Comrie.

Thanks to Bram Wiepjes for developing Baserow.io and to Sleiman Tanios whose Airtable API Client this library borrows from substantially.
