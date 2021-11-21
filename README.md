# PHPSupabase

PHPSupabase is a library written in php language, which allows you to use the resources of a project created in Supabase ([supabase.io](https://supabase.io)), through integration with its Rest API.

## Content

- [About Supabase](#about-supabase)
- [PHPSupabase Features](#phpsupabase-features)
- [Instalation & Loading](#instalation-&-features)
- [How to use](#how-to-use)
    - [Auth Class](#auth-class)
        - [Create a new user with email and password](#create-a-new-user-with-email-and-password)
        - [Sign in with email and password](#sign-in-with-email-and-password)
        - [Get the data of the logged in user](#get-the-data-of-the-logged-in-user)
        - [Update user data](#update-user-data)
    - [Database class](#database-class)
        - [Insert data](#insert-data)
        - [Update data](#update-data)
        - [Delete data](#delete-data)
        - [Fetch data](#fetch-data)
        - [Comparison operators](#comparison-operators)

## About Supabase

Supabase is "The Open Source Firebase Alternative". Through it, is possible to create a backend in less than 2 minutes. Start your project with a Postgres Database, Authentication, instant APIs, realtime subscriptions and Storage.

## PHPSupabase Features

- Create and manage users of a Supabase project
- Manage user authentication (with email/password, magic links, among others)
- Insert, Update, Delete and Fetch data in Postgres Database (by Supabase project Rest API)
- A QueryBuilder class to filter project data in uncomplicated way

## Instalation & loading

PHPSupabase is available on [Packagist](https://packagist.org/packages/rafaelwendel/phpsupabase), and instalation via [Composer](https://getcomposer.org) is the recommended way to install it. Add the follow line to your `composer.json` file:

```json
"rafaelwendel/phpsupabase" : "^0.0.1"
```

or run

```sh
composer require rafaelwendel/phpsupabase
```

## How to use

To use the PHPSupabse library you must have an account and a project created in the Supabase panel. In the project settings (API section), you should note down your project's `API key` and `URL`. (NOTE: Basically we have 2 suffixes to use with the url: `/rest/v1` & `/auth/v1`)

To start, let's instantiate the `Service()` class. We must pass the `API key` and `url` (with one of the suffixes mentioned above) in the constructor

```php
<?php

require "vendor/autoload.php";

$service = new PHPSupabase\Service(
    "YOUR_API_KEY", 
    "https://aaabbbccc.supabase.co/auth/v1/"
);
```

The `Service` class abstracts the actions with the project's API and also provides the instances of the other classes (`Auth`, `Database` and `QueryBuilder`).

### Auth class

Let's instantiate an object of the `Auth` class

```php

$auth = $service->createAuth();
```

The `$auth` object has several methods for managing project users. Through it, it is possible, for example, to create new users or even validate the sign in of an existing user.

#### Create a new user with email and password

See how to create a new user with `email` and `password`.

```php

$auth = $service->createAuth();

try{
    $auth->createUserWithEmailAndPassword('newuser@email.com', 'NewUserPassword');
    $data = $auth->data(); // get the returned data generated by request
    echo 'User has been created! A confirmation link has been sent to the '. $data->email;
}
catch(Exception $e){
    echo $auth->getError();
}
```

This newly created user is now in the project's user table and can be seen in the "Authentication" section of the Supabase panel. To be enabled, the user must access the confirmation link sent to the email.

#### Sign in with email and password

Now let's see how to authenticate a user. The Authentication request returns a `acess_token` (Bearer Token) that can be used later for other actions and also checks expiration time. In addition, other information such as `refresh_token` and user data are also returned. Invalid login credentials result in throwing a new exception

```php

$auth = $service->createAuth();

try{
    $auth->signInUserWithEmailAndPassword('user@email.com', 'UserPassword');
    $data = $auth->data(); // get the returned data generated by request

    if(isset($data->access_token)){
        $userData = $data->user; //get the user data
        echo 'Login successfully for user ' . $userData->email;

        //save the $data->acess_token in Session, Cookie or other for future requests.
    }
}
catch(Exception $e){
    echo $auth->getError();
}
```

#### Get the data of the logged in user

To get the user data, you need to have the `access_token` (Bearer Token), which was returned in the login action.

```php

$auth = $service->createAuth();
$bearerToken = 'THE_ACCESS_TOKEN';

try{
    $data = $auth->getUser($bearerToken);
    print_r($data); // show all user data returned
}
catch(Exception $e){
    echo $auth->getError();
}
```

#### Update user data

It is possible to update user data (such as email and password) and also create/update `metadata`, which are additional data that we can create (such as `first_name`, `last_name`, `instagram_account` or any other).

The `updateUser` method must take the `bearerToken` as argument. In addition to it, we have three more optional parameters, which are: `email`, `password` and `data` (array). If you don't want to change some of this data, just set it to `null`.

An example of how to save/update two new meta data (`first_name` and `last_name`) for the user.

```php

$auth = $service->createAuth();
$bearerToken = 'THE_ACCESS_TOKEN';

$newUserMetaData = [
    'first_name' => 'Michael',
    'last_name'  => 'Jordan'
];

try{
    //the parameters 2 (email) and 3(password) are null because this data will not be changed
    $data = $auth->updateUser($bearerToken, null, null, $newUserMetaData);
    print_r($data); // show all user data returned
}
catch(Exception $e){
    echo $auth->getError();
}
```
Note that in the array returned now, the keys `first_name` and `last_name` were added to `user_metadata`.

### Database class

The Database class provides features to perform actions (insert, update, delete and fetch) on the Postgre database tables provided by the Supabase project.

For the samples below, consider the following database structure:

```sql
categories (id INT AUTO_INCREMENT, categoryname VARCHAR(32))
products (id INT AUTO_INCREMENT, productname VARCHAR(32), price FLOAT, categoryid INT)
```

The Database class is also instantiated from the `service` object. You must pass the `table` that will be used and its respective primary key (usually `id`).

Let's create an object to work with the `categories` table:

```php
$db = $service->initializeDatabase('categories', 'id');
```

Through the `db` variable it is possible to perform the actions on the `categories` table.

#### Insert data

Inserting a new record in the `categories` table:

```php
$db = $service->initializeDatabase('categories', 'id');

$newCategory = [
    'categoryname' => 'Video Games'
];

try{
    $data = $db->insert($newCategory);
    print_r($data); //returns an array with the new register data
    /*
        Array
        (
            [0] => stdClass Object
                (
                    [id] => 1
                    [categoryname] => Video Games
                )

        )
    */
}
catch(Exception $e){
    echo $e->getMessage();
}
```

Now let's insert a new product from category `1 - Video Games`:

```php
$db = $service->initializeDatabase('products', 'id');

$newProduct = [
    'productname' => 'XBOX Series S',
    'price'       => '299.99',
    'categoryid'  => '1' //Category "Video Games"
];

try{
    $data = $db->insert($newProduct);
    print_r($data); //returns an array with the new register data
    /*
        Array
        (
            [0] => stdClass Object
                (
                    [id] => 1
                    [productname] => XBOX Series S
                    [price] => 299.99
                    [categoryid] => 1
                )
        )
    */
}
catch(Exception $e){
    echo $e->getMessage();
}
```

#### Update data

To update a record in the database, we use the `update` method, passing as parameter the `id` (PK) of the record to be updated and an `array` containing the new data (NOTE: For now, it is not possible to perform an update using a parameter other than the primary key).

In the example below, we will update the `productname` and `price` of the product with `id=1` ("Xbox Series S" to "XBOX Series S 512GB" and "299.99" to "319.99"):

```php
$db = $service->initializeDatabase('products', 'id');

$updateProduct = [
    'productname' => 'XBOX Series S 512GB',
    'price'       => '319.99'
];

try{
    $data = $db->update('1', $updateProduct); //the first parameter ('1') is the product id
    print_r($data); //returns an array with the product data (updated)
    /*
        Array
        (
            [0] => stdClass Object
                (
                    [id] => 1
                    [productname] => XBOX Series S 512GB
                    [price] => 319.99
                    [categoryid] => 1
                )
        )
    */
}
catch(Exception $e){
    echo $e->getMessage();
}
```

#### Delete data

To delete a record from the table, just call the `delete` method and pass the `id` (PK) of the record to be deleted as a parameter.

The following code deletes the product of `id=1` in the `products` table:

```php
$db = $service->initializeDatabase('products', 'id');

try{
    $data = $db->delete('1'); //the parameter ('1') is the product id
    echo 'Product deleted successfully';
}
catch(Exception $e){
    echo $e->getMessage();
}
```
#### Fetch data

The following methods for fetching data are available in the `Database` class:

- `fetchAll()`: fetch all table records;
- `findBy(string $column, string $value)`: fetch records filtereds by a column/value (using the `=` operator);
- `findBy(string $column, string $value)`: fetch records filtereds by a column/value (using the `LIKE` operator);
- `join(string $foreignTable, string $foreignKey)`: make a `join` between the seted table and another table related and fetch records;
- `createCustomQuery(array $args)`: build a custom SQL query. The following `keys` are valid for the `args` argument:
    - `select`
    - `from`
    - `join`
    - `where`
    - `range`

All the mentioned methods return the self instance of `Database` class. To access the fetched data, call the `getResult` method.

See some examples:

```php
$db = $service->initializeDatabase('products', 'id');

try{
    $listProducts = $db->fetchAll()->getResult(); //fetch all products
    foreach ($listProducts as $product){
        echo $product->id . ' - ' . $product->productname . '($' . $product->price . ') <br />';
    }
}
catch(Exception $e){
    echo $e->getMessage();
}
```

Now, an example using the `findBy` method:

```php
$db = $service->initializeDatabase('products', 'id');

try{
    $listProducts = $db->findBy('productname', 'PlayStation 5')->getResult(); //Searches for products that have the value "PlayStation 5" in the "productname" column
    foreach ($listProducts as $product){
        echo $product->id . ' - ' . $product->productname . '($' . $product->price . ') <br />';
    }
}
catch(Exception $e){
    echo $e->getMessage();
}
```

Searching for `products` and adding a join with the `categories` table:

```php
$db = $service->initializeDatabase('products', 'id');

try{
    $listProducts = $db->join('categories', 'id')->getResult(); //fetch data from "products" JOIN "categories"
    foreach ($listProducts as $product){
        //SHOW "productname" - "categoryname"
        echo $product->productname . ' - ' . $product->categories->categoryname . '<br />';
    }
}
catch(Exception $e){
    echo $e->getMessage();
}
```

An example of a custom query to search `id,productname,price` for all `products` "JOIN" `categories` filtering by `price` (`price` greater than `200.00`):

```php
$db = $service->initializeDatabase('products', 'id');

$query = [
    'select' => 'id,productname,price',
    'from'   => 'products',
    'join'   => [
        [
            'table' => 'categories',
            'tablekey' => 'id'
        ]
    ],
    'where' => 
    [
        'price' => 'gt.200' //"gt" means "greater than" (>)
    ]
];

try{
    $listProducts = $db->createCustomQuery($query)->getResult();
    foreach ($listProducts as $product){
        echo $product->id . ' - ' . $product->productname . '($' . $product->price . ') <br />';
    }
}
catch(Exception $e){
    echo $e->getMessage();
}
```

Other examples for custom query:

```php
//products with price > 200 AND productname LIKE '%n%'
$query = [
    'select' => 'id,productname,price',
    'from'   => 'products',
    'where' => 
    [
        'price' => 'gt.200', //"gt" means "greater than" (>)
        'productname' => 'like.%n%' //like operator
    ]
];

//products with categoryid = 1
$query = [
    'select' => 'id,productname,price',
    'from'   => 'products',
    'where' => 
    [
        'categoryid' => 'eq.1', //"eq" means "equal" (=)
    ]
];

//products with price < 1000 LIMIT 4 results
$query = [
    'select' => 'id,productname,price',
    'from'   => 'products',
    'where' => 
    [
        'price' => 'lt.1000', //"lt" means "less than" (<)
    ],
    'range' => '0-3' //4 first rows
];
```

#### Comparison operators

Some operators available for the `where` clause:
- `eq`: equal
- `neq`: not equal
- `gt`: greater than
- `gte`: greater than or equal
- `lt`: less than
- `lte`: less than or equal
- `like`: search for a specified pattern in a column
- `ilike`: search for a specified pattern in a column (case insensitive)