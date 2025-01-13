---
title: Basic DB syntaxes
date: 2024-10-26T00:00:00.000Z
description: A list of basic syntax of MySQL database
tags:
  - database
  - mysql
  - php
---

## Connecting a localhost

### for linux using ubuntu

1. Install :

```bash
sudo apt install mysql-server
```

2. check mysql service status:

```bash
sudo systemctl status mysql.service # check status

sudo systemctl start mysql.service # if the status is not active
```

3. start mysql

```bash
sudo mysql -uroot
```

by default,`root` is the user and no password. To Alter the user do the following.

```bash
mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'yourPassword';
```

- `root` : the user.
- `localhost` : the address
- `yourPassword` : the password.

after altering the **USER** you can start mysql like this:

```bash
sudo mysql -u root -p
```

> this is mostly for testing and scaffoling setup. If you want to run a dev/prod development.
> run `sudo mysql_secure_installation

this will be the prompts on running that command.

```bash


Securing the MySQL server deployment.

Enter password for user root:

VALIDATE PASSWORD COMPONENT can be used to test passwords
and improve security. It checks the strength of password
and allows the users to set only those passwords which are
secure enough. Would you like to setup VALIDATE PASSWORD component?

Press y|Y for Yes, any other key for No: n
Using existing password for root.
Change the password for root ? ((Press y|Y for Yes, any other key for No) : no

 ... skipping.
By default, a MySQL installation has an anonymous user,
allowing anyone to log into MySQL without having to have
a user account created for them. This is intended only for
testing, and to make the installation go a bit smoother.
You should remove them before moving into a production
environment.

Remove anonymous users? (Press y|Y for Yes, any other key for No) : n

 ... skipping.


Normally, root should only be allowed to connect from
'localhost'. This ensures that someone cannot guess at
the root password from the network.

Disallow root login remotely? (Press y|Y for Yes, any other key for No) : n

 ... skipping.
By default, MySQL comes with a database named 'test' that
anyone can access. This is also intended only for testing,
and should be removed before moving into a production
environment.


Remove test database and access to it? (Press y|Y for Yes, any other key for No) : n

 ... skipping.
Reloading the privilege tables will ensure that all changes
made so far will take effect immediately.

Reload privilege tables now? (Press y|Y for Yes, any other key for No) : Y
Success.

All done!
```

## Basic syntax

difference between **""** (double qoutes) and **''** (single qoute)

- **""** : use for identifiers
- **''** : use for strings and values

- Create table
  - two ways of creating unique constraints through inline/during creation of table and creating a unique index for the particular column or multiple columns.
- Create Index
- Alter table
- Query table
  - select filter
  - query with constraints (where)
- Inserting table

  - pitfall for adding and inserting about the comma `,` at the last value

  ```sql
   will error
  mysql> UPDATE notes SET user_id = 1, WHERE id = 1;

  will not error
  mysql> UPDATE notes SET user_id = 1 WHERE id = 1;

  ```

- foreign key

  - adding while altering

    - Method 1:

    Syntax:

    ```sql
    ALTER yourtable ADD column_name data_type opts
    REFERENCES referenced_table(referenced_table_column);
    ```

    Example:

    ```sql
    ALTER notes ADD user_id int NOT NULL
    REFERENCES users(id);
    ```

    -Method 2:

    ```sql
    ALTER your_table
    ADD column_name data_type opts,
    ADD CONSTRAINT name_fk_key FOREIGN KEY (column_name) REFERENCES referenced_table(referenced_table_column);
    ```

    Example:

    ```sql
    ALTER notes
    ADD user_id int NOT NULL,
    ADD CONSTRAINT fk_notes_user FOREIGN KEY (user_id) REFERENCES users(id);
    ```

ON UPDATE and ON DELETE -> orphans
CASCADE delete the connected data
RESTRICT cant delete the data as long as it is still connected.

## Database query security

- ### SQL injection

  - #### GET request

    The most common security breach in a sql queries. What it does is it can modify your database by adding or modifying the structure of your intended query.
    This is especially common when a paramter or a variable is being passed **Inline** to a database query.

    Example using query paramater:
    The [class and config used in this example](#query-example-structure-and-config)

    ```php
    <?php
    $url = "https://example.com/?id=3"

    $id = $_GET['id']; // $id now is 3

    $db = new Database($config['dsm']);

    // id from query params
    $statement = $db->query("SELECT * FROM posts where id = {$id}")->fetch();
    ```

    This looks normal but the moment that the query params which is the `$id` is being passed in the sql query this becomes a security vulnerability.

    ```php
    <?php
    // Imagine this a the browser url search
    // and will add more than the id=3
    $url = "https://example.com/?id=3; drop table posts";
    $id = $_GET['id']; // $id now is 3

    $db = new Database($config['dsm']);

    $statement = $db->query("SELECT * FROM posts where id = {$id}")->fetch();
    // the sql that will be executed will look like this :
    /*
      SELECT * FROM posts where id = 3;
      DROP TABLE posts;
    */
    ```

    Youll get the result but your posts table is also deleted.

    so for security measures:

    > [!caution] SQL security
    > NEVER INLINE YOUR variable or query params as part of a SQL query.

  <!--NOTE: already handled the precaution in php blog-->

  - ### dynamic sql query with security in mind

    to prevent the attacker to do something Malicious to your sql query. Php `PDO` `execute` method have an option to pass an `array`.
    The accepted argument for `execute` method:

    - **associated array** with **wildcard** : we use wild card as an identifier for sql query values. and we will use that wild card as a `key/property` and `variable` as the value. (example below)

    - **array** with `?` : if we want to use just the array there are things to keep in mind.
      1. `?` : use ? as a placeholder for the values of your sql queries
      2. sequence : if we use the `?` we have to be mindful of the **sequence** of the values.

    ```php
    <?php
    $config = require("configs.php");

    $id = $_GET['id'];

    $db = new Database($config['database'], 'root', 'ting');
    // declaring the variable in an identifier/wild card
    $query = "SELECT * FROM posts WHERE id = :id";
    $posts = $db->query($query, [':id' => $id]);
    // can be any sequence as long as the identifier is correct

    // adding ?
    /*$query = "SELECT * FROM posts WHERE id = ?";*/
    /*$posts = $db->query($query, [$id]);
    must follow the sequence when passing as an array
    */

    foreach ($posts as $post) {
        echo "<li>{$post['title']}</li>";
    }

    ```

  - #### POST request

    There is also a security risk in a post request, this is common for handling a form submission. Eventho, you setup a guard rails like never passing the variable directly or inline. It is still possible to create a security risk.

    Example:
    we will use this [example class structure](#query-example-structure)

    ```php
    <?php
    $body = $_POST;
    /*output:
    array(1) {
      ["body"]=>
      string(3) "foo"
    }
    */

    $db = new Database($config['dsm']);

    $statement = $db->query("INSERT INTO posts(body) VALUES(:body) ",['body'=>$body]);
    ```

    Now this looks normal and always we prevented the risk of sql injection by separating the sending of request of the query and the variable associated with it.

    But with this Post request introduce another problem when used/received the normal way:

    ```php
    <?php

    $db = new Database($config['dsm']);

    $posts = $db->query("SELECT * FROM posts")->fetchAll();
    ?>

    <ul>
      <?php foreach($posts as $post): ?>
        <?= $post ?>
      <?php endforeach ?>
    </ul>
    ```

    In this normal and the usual way of receiving the `$posts` from DB you let the user takes control of what it is to `echo` inside the `<ul>`.
    By doing so The new security risk here is the ability to manipulate the html elements and template most especially the `<script>` tag and the attacker can now have an access to your javascript which is dangerous. like so:

    Imagine the user input like this:

    ```php
    <?php
    $body = $_POST;
    /*output:
    array(1) {
      ["body"]=>
      string(3) "foo <h1>I took control</h1><script>alert("i can now control the JS")</script>"
    }
    */

    ```

    now your frontend will look something like this:

    ```php

    <ul>
      <?php foreach($posts as $post): ?>
        <?= $post ?>

      <!-- foo -->
      <!-- <h1>I took control</h1> -->
      <!-- <script>alert('i can now control the JS')</script> -->

      <?php endforeach ?>
    </ul>
    ```

    This is risky, the user can now do everything that whatever the script tag is capable of.

    ##### Preventing post security risk using php

    In php there is a simple way to prevent this by using the helper `htmlspecialchars()`
    By using this helper it will convert all the special characters like `<`, `'` , `"`, `>` and other possible special characters to a html entities. like so:

    ```php

    <ul>
      <?php foreach($posts as $post): ?>
        <?= htmlspecialchars($post) ?>

      <!-- foo -->
      <!-- &lt;h1&gt;I took control&lt;/h1&gt; -->
      <!-- &lt;script&gt;alert(&qout;i can now control the JS&qout;)&lt;/script&gt; -->
      <?php endforeach ?>
    </ul>
    ```

    now all the html tags became html entities with this even the `script` tag is now being rendered.

  - #### Query Example structure and config

  ```php
  <?php

    $config = [
      'dsm' => "mysql:host=localhost;port:3306;dbname=mydb;charset=utf8mb4",
      'user' => "root",
      'password' => "my-pass",
    ]


  class Database
      {
          private $connection;

          public function __construct($config, $userName = "root", $password = "")
          {
              $dsm = "mysql:" . http_build_query($config, "", ";");
              $this->connection = new PDO($dsm, $userName, $password, [PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC]); //so that it will only be associated array
          }

          public function query($query, $params = [])
          {

              $statement = $this->connection->prepare($query);

              $statement->execute($params);

              return $statement;
          }
      }

  ```
