---
layout: post
title: "PHP Class Design for MySQL?"
date: 2013-10-09 15:24
comments: true
categories: 
- PHP
- MySQL
- Object Oriented
---
Since I first started out my journey on PHP, I have been trying to find a good design to make coding less messy when using MySQL.
Anyone use PHP knows that mixing PHP and MySQL together makes the code less readable and less modifiable, worse, it makes debugging
much harder. These situations would only get worse if you don't refactor your code. So, here is my solution for PHP (sure, it is
applicable in any language), using an abstract class as the base class for all records:

<!-- more -->

{% codeblock Record %}
<?php
abstract class Record{
    //The tables where the operation will take place
    const UPDATE_TABLE = 0; //The table where the record to be updated
    const DELETE_TABLE = 1; //The table where the record to be deleted

    //Operations against the table/s
    const OP_DELETE = 'delete_record';
    const OP_UPDATE = 'update_record';
    const OP_CREATE = 'create_record';
    const OP_READ = 'read_record';

    //For type checking
    const TYPE_STRING = 'string';
    const TYPE_BOOL = 'boolean';
    const TYPE_INT = 'integer';
    const TYPE_DOUBLE = 'double';
    const TYPE_ARRAY = 'array';
    const TYPE_OBJECT = 'object';
    const TYPE_RESOURCE = 'resource';
    const TYPE_NULL = 'NULL';
    const TYPE_UNKNOWN = 'unknown type';

    //Log types for log4php
    const LOG_TYPE_WARN = "WARN";
    const LOG_TYPE_INFO = "INFO";
    const LOG_TYPE_DEBUG = "DEBUG";

    const ID = 'id';
    
    protected $logger;
    protected $connection;

    /**
     * Constructor
     * @param connection MySQLi object used to connect to MySQL database
     * @param hostname The hostname of the server
     * @param username The username to connect to MySQL
     * @param passwd The password to connect to MySQL
     * @param dbname The database name of MySQL to use
     */
    function __construct(mysqli $connection = null,
                         $hostname = "localhost",
                         $username = "root",
                         $passwd = "",
                         $dbname = "purchase_order"){
        if($connection === null)
            $this->connection = new mysqli($hostname, $username,
                                           $passwd, $dbname);
        else
            $this->connection = $connection;

        $this->connection->set_charset('utf8');

        $this->logger = null;
    }

    /**
     * Set the logger for log4php
     * @param logger The logger used for loggin
     */
    function setLogger($logger = null){
        $this->logger = $logger;
    }

    /**
     * Get the connection for the database
     */
    public function getConnection(){
        return $this->connection;
    }

    /**
     * Get the connection error description
     * @return The error for the connection
     */
    public function error(){
        return $this->connection->error;
    }

    /************************************
     *RETRIEVE
     ************************************/

    /**
     * Custom query without escapes, escape your values before passing into
     * this function
     * @param fields MySQL fields comma separated
     * @param tbl MySQL table comma separated
     * @param condition Full MySQL conditions without 'WHERE' keyword
     f @param additional An additional MySQL queries, have to be in MySQL
     * @return Associative array of results or null on failure
     */
    public function select($fields = null, $tbl = null, $condition = null,
                           $additional = null){
        if(!$this->isValidValues(array($fields, $tbl), self::TYPE_STRING))
            return null;

        $sql = "SELECT $fields FROM $tbl";

        $authCond = null;
        $auth = $this->isAuthorized(self::OP_READ, $authCond);
        if(!$auth){
            $this->log('Unauthorized read access!', self::LOG_TYPE_WARN);
            return null;
        }else if($auth && $authCond != null)
            $sql .= " WHERE $authCond";

        if($condition != null){
            if($authCond != null)
                $sql .= " AND $condition $additional";
            else
                $sql .= " WHERE $condition $additional";
        }else
            $sql .= " $additional";
        $this->log("Executing SQL: $sql", self::LOG_TYPE_DEBUG);

        $query = $this->connection->query($sql);
        if($query){
            $result = $this->fetchAllAssoc($query);
            $query->close();
            $this->log("Query success: Fetched ". count($result)."rows",
                        self::LOG_TYPE_DEBUG);

            return $result;
        }else
            $this->log("Query failed: ". $this->error(), self::LOG_TYPE_WARN);

        return null;
    }

    /**
     * Get all records with starting records and number of records
     * @param start The starting record, default 0
     * @param count The number of records, set to 0 to list all, default 0
     * @return An associative array based on field name as keys or null on
     * failure
     */
    public function getAll($start = 0, $count = 0){
        $this->log('Retrieving all records', self::LOG_TYPE_INFO);
        $fields = null;
        $tables = null;
        $conditions = null;

        $this->getAllQueryStatements($fields, $tables, $conditions);

        if(!$this->isValidValues(array($fields, $tables), self::TYPE_STRING))
            return null;

        if($count != 0)
            return $this->select($fields, $tables, $conditions, "LIMIT $start,
                                 $count");
        else
            return $this->select($fields, $tables, $conditions);
    }

    /************************************
     *CREATE
     ************************************/

    /**
     * Insert a record
     * @param fields The fields to be inserted
     * @return True on success, null on failure
     */
    public function insert(array $fields = null){}

    /************************************
     *UPDATE
     ************************************/

    /**
     * Updates a record, using template method.
     * At least the 'id' key has to be set.
     * @param fields The field/s to be updated
     * @see subclass::getUpdateColumns()
     */
    public function update(array $fields = null){
        $this->log('Updating record', self::LOG_TYPE_INFO);

        //Authorization
        $authCond = null;
        if(!$this->isAuthorized(self::OP_UPDATE, $authCond)){
            $this->log('Unauthorized update!', self::LOG_TYPE_WARN);
            return null;
        }

        $table = $this->getTableName(self::UPDATE_TABLE);
        $columns = $this->getUpdateColumns($fields, $table);

        if($fields === null || !$this->validKeys(array(self::ID), $fields))
            return null;

        $this->preUpdate($fields[self::ID]);

        $result = null;
        if($table && count($columns) > 0){
            $sql = "UPDATE " . $table . " SET ";
            $count = count($columns);
            if(isset($columns['conditions']))
                $count = count($columns) - 1;
            for($i = 0; $i < $count; $i++){
                $sql .= $columns[$i];
                if($i < $count -1)
                    $sql .= ", "; //Insert comma if not last element
            }

            if(!isset($columns['conditions']))
                $sql .= " WHERE id=" . $fields[self::ID];
            else
                $sql .= ' WHERE '. $columns['conditions'];

            if($authCond != null)
                $sql .= " AND $authCond";

            $this->log("Executing SQL: $sql", self::LOG_TYPE_DEBUG);

            $result = $this->connection->query($sql);
            if(!$result){
                $this->log("SQL statement failed: ". $this->error(),
                                    self::LOG_TYPE_WARN);
                $result = null;
            }else{
                $this->log('SQL execution successful!',
                            self::LOG_TYPE_INFO);
                $result = $fields['id'];
            }
        }else if($table === '' || $table === null){
            $this->log('Invalid table name for update!',
                        self::LOG_TYPE_WARN);
            $result = null;
        }else if(count($columns) <= 0){
            $this->log('No columns returned!', self::LOG_TYPE_WARN);
            $result = null;
        }else
            $this->log('An unknown mutant error has occured! Arghh!!',
                        self::LOG_TYPE_WARN);

        $this->postUpdate($result);
        return $result;
    }

    /************************************
     *DELETE
     ************************************/
    /**
     * Delete a record
     * @param id The id that identifies the record to be deleted
     * @return Deleted ID on success null on failure
     */
    public function delete($id = null){
        if(!$this->isValidValues(array($id), self::TYPE_INT))
            return null;

        $this->log("Deleting record $id", self::LOG_TYPE_INFO);

        $data = null;
        $this->preDelete($id);

        //Authorization
        $authCond = null;
        if(!$this->isAuthorized(self::OP_UPDATE, $authCond)){
            $this->log('Unauthorized update!', self::LOG_TYPE_WARN);
            return null;
        }

        $sql = "DELETE FROM " . $this->getTableName(self::DELETE_TABLE)
             . " WHERE id=$id";
        $this->log("Executing SQL: $sql", self::LOG_TYPE_DEBUG);
        $result = $this->connection->query($sql);
        if(!$result){
            $this->log("SQL statement failed: ". $this->error(),
                        self::LOG_TYPE_WARN);
            $result = null;
        }else
            $result = $id;

        $this->postDelete($result);

        return $result;
    }

    /************************************
     *TEMPLATE METHOD
     ************************************/

    /**
     * Template method for getAll() function
     * Modify fields, tables and conditions to suit your need
     * @param fields The fields that need to be modified
     * @param tables The tables that need to be modified
     * @param conditions The conditions that need to be modified (optional)
     */
    abstract protected function getAllQueryStatements(&$fields, &$tables,
                                                      &$conditions);

    /**
     * Template method for update() function
     * If 'conditions' key is set, it will override the original WHERE clause
     * @return array An array of individual 'field'='value' string for SET
     * MySQL clause
     * @see Record::update()
     */
    protected function getUpdateColumns(array &$fields = null){}

    /**
     * Template method for delete() and update() to get the table name to be
     * altered
     */
    abstract protected function getTableName($operation);

    /**
     * Operations before the deletion of a record
     * @param id The id of the record to be deleted
     */
    protected function preDelete($id){}
    /**
     * Operations after the deletion of a record
     * @param result True on success null on failure
     */
    protected function postDelete($result){}

    /**
     * Operations before the update process
     * @param id The id of the record to be deleted
     */
    protected function preUpdate($id){}
    /**
     * Operations after the update process
     * @param result True on success null on failure
     */
    protected function postUpdate($result){}

    /**
     * Whether an operation against a record is authorized
     */
    protected function isAuthorized($operation = null, &$condition = null){
        return true;
    }

    /************************************
     *HANDY UTILITIES
     ************************************/

    /**
     * Converts 2D array to 1D array
     * @param array The array to convert
     * @return A converted 1D array or null on failure
     */
    public function toSingleArray(array $array = null){
        $this->log('Converting 2D '. count($array) .' to 1D array',
                    self::LOG_TYPE_INFO);
        if($array === null || count($array) == 0)
            return null;

        $result = array();
        foreach($array as $row){
            foreach($row as $key => $value)
                $result[$key] = $value;
        }
        $this->log('Conversion resulted in an array size of '. count($result),
                    self::LOG_TYPE_INFO);

        return $result;
    }

    /**
     * Validate if an array of values contain not allowable values, return false if not
     * @param requiredKeys The keys that are non-empty and non-null
     * @param fields The fields that needed to be checked
     * @return True on success or false on violation
     */
    public function validKeys(array $requiredKeys = null, array $fields = null){
        $this->log('Checking for invalid keys', self::LOG_TYPE_INFO);
        if($fields === null || !$requiredKeys){
            $this->log('Null array found!', self::LOG_TYPE_WARN);
            return false;
        }

        foreach($requiredKeys as $key){
            if(!isset($fields[$key])){
                $this->log('Required field key ('. $key .') not found!',
                            self::LOG_TYPE_WARN);
                return false;
            }
            if($fields[$key] === null){
                $this->log('Null not allowed for key ('. $key .')!',
                            self::LOG_TYPE_WARN);
                return false;
            }
        }

        return true;
    }

    /**
     * Check if values are valid and the type of the values
     * @param values The values to check against
     * @param datatype Accepted data type for $values
     * @return True on success or false on violation
     */
    public function isValidValues($value = null, $datatype = self::TYPE_STRING){
        $this->log('Checking for invalid values (non-'. $datatype .')',
                    self::LOG_TYPE_INFO);
        if($value === null){
            $this->log('Null array found! $values can\'t be null!',
                        self::LOG_TYPE_WARN);
            return false;
        }

        if(gettype($value) == self::TYPE_ARRAY){
            foreach($value as $val){
                if($val === null || gettype($val) != $datatype){
                    if(gettype($val) == self::TYPE_ARRAY)
                        $val = implode(', ', $val);
                    $this->log('Invalid values found => "'. $val
                              .'" of type "'. gettype($val) .'"',
                                self::LOG_TYPE_WARN);
                    return false;
                }
            }
        }else{
            if($value === null || gettype($value) != $datatype){
                $this->log('Invalid values found => "'. $value
                          .'" of type "'. gettype($value) .'"',
                            self::LOG_TYPE_WARN);
                return false;
            }
        }

        return true;
    }

    /**
     * Write to log file
     * @param msg The message to write
     * @param type The type of log
     */
    protected function log($msg = "", $type = self::LOG_TYPE_INFO){
        if($this->logger != null){
            if($type == self::LOG_TYPE_WARN){
                $this->logger->warn($msg);
            }else if($type == self::LOG_TYPE_INFO){
                $this->logger->info($msg);
            }else if($type == self::LOG_TYPE_DEBUG){
                $this->logger->debug($msg);
            }
        }
    }

    public function fetchAllAssoc($result = null){
        if($result === null)
            return null;

        $assoc = array();
        while($row = $result->fetch_assoc()){
            $assoc[] = $row;
        }

        return $assoc;
    }
}
?>
{% endcodeblock %}

The design that here make heavy use of IDs in the table. So, you might need to tailor it to suit your needs.
After creating this class, you can start using it as the base class for all your other database operations. Just extend it
like you would with other classes, and start implementing classes in the blocks marked __TEMPLATE METHOD__, abstract
functions are mandatory which you must implement, while non-abstract functions are optional, they are there just so you don't
need to implement what you don't use. A sample of a derived class would look like this:

{% codeblock Sample Implementation %}
<?php
require_once('Record.php');
/**
 * This class will handle all the item records in the item table
 */
class ItemRecord extends Record{

    /************************************
     *CREATE
     ************************************/
    /**
     * $fields should have all the following keys except 'description':
     * - ITEM_NO: The item number (string)
     * - ITEM_DESC: Description of the item (string)
     * - ITEM_QTY: Quantity of the item (integer)
     * - ITEM_PRICE: Price of the item (float or double)
     * {@inheritdoc}
     */
    public function insert(array $fields = null){
        if(!$this->validKeys(array(self::ITEM_NO, self::ITEM_QTY, self::ITEM_PRICE), $fields))
            return null;
        if(!$this->isValidInput($fields))
            return null;

        $this->log('Inserting record', self::LOG_TYPE_INFO);

        $insertFields = 'item_no, quantity , price, item_description';
            
        $values = "'". $this->connection->escape_string($fields[self::ITEM_NO])
            . "', ". $fields[self::ITEM_QTY] .', '. $fields[self::ITEM_PRICE] .', ';
        if(isset($fields[self::ITEM_DESC]))
            $values .= "'". $this->connection->escape_string($fields[self::ITEM_DESC]) ."'";
        else
            $values .= "null";

        $sql = "INSERT INTO ". self::ITEM_TABLE ." ($insertFields) VALUES ($values)" ;

        $this->log("Executing SQL: $sql", self::LOG_TYPE_DEBUG);

        if($this->connection->query($sql))
            return $this->connection->insert_id;

        $this->log("SQL execution failed: ". $this->error(),
                    self::LOG_TYPE_WARN);
        return null;
    }

    /************************************
     *TEMPLATE METHOD IMPLEMENTATION
     ************************************/
    /**
     * {@inheritDoc}
     */
    protected function getTableName($operation){
        return self::ITEM_TABLE;
    }

    /**
     * {@inheritDoc}
     */
    public function getAllQueryStatements(&$fields, &$tables, &$conditions){
        $fields = '*';
        $tables = self::ITEM_TABLE;
    }

    /**
     * $fields may have the following keys:
     * - ITEM_NO: The item number
     * - ITEM_DESC: Description of the item
     * - ITEM_QTY: Quantity of the item
     * - ITEM_PRICE: Price of the item
     * {@inheritDoc}
     */
    public function getUpdateColumns(array &$fields = null){
        $columns = array();

        if(!$this->isValidInput($fields)){
            $this->log('Incorrect data type detected!', self::LOG_TYPE_WARN);
            return null;
        }

        if(isset($fields[self::ITEM_NO]))
            $columns[] = "item_no='" . $this->connection->escape_string($fields[self::ITEM_NO]) . "'";
        if(isset($fields[self::ITEM_DESC]))
            $columns[] = "item_description='" . $this->connection->escape_string($fields[self::ITEM_DESC]) . "'";
        if(isset($fields[self::ITEM_QTY]))
            $columns[] = "quantity=" . $fields[self::ITEM_QTY];
        if(isset($fields[self::ITEM_PRICE]))
            $columns[] = "price=" . $fields[self::ITEM_PRICE];

        return $columns;
    }

    /************************************
     *UTILITIES
     ************************************/

    /**
     * Checks whether input is valid
     * @param fields Array of key value pair to check against
     * @return True if valid or false if invalid
     */
    public function isValidInput($fields = null){
        if((isset($fields[self::ITEM_NO]) && !$this->isValidValues(array($fields[self::ITEM_NO]), self::TYPE_STRING))
        || (isset($fields[self::ITEM_DESC]) && !$this->isValidValues(array($fields[self::ITEM_DESC]), self::TYPE_STRING))
        || (isset($fields[self::ITEM_QTY]) && !$this->isValidValues(array($fields[self::ITEM_QTY]), self::TYPE_INT))
        || (isset($fields[self::ITEM_PRICE]) && !$this->isValidValues(array($fields[self::ITEM_PRICE]), self::TYPE_DOUBLE)))
            return false;

        return true;
    }
}
?>
{% endcodeblock %}

##Usage Notes##
1. Inserting data into table has to be manually implemented with the insert() function.
2. The getTableName() function __MUST__ be implemented, and has to return the table name, the parent class wouldn't know if you don't tell them which table to update/delete/retrieve/update.
3. All classes extending `Record` will automatically have a delete() function without the need for implementation, it uses ID to identify a row in the table, override this function if you don't use ID.
4. Retrieval of records are implemented in getAll() function, which in turn calls getAllQueryStatements() function, you have to set the fields, tables and conditions in this function, no need for return (it's pass by reference).
5. The update() function will call the getUpdateColumns(), so, you have to override getUpdateColumns() to return queries in an array, each index consist of field=value, this will be passed directly into the queries.
6. All functions starting with pre and post are functions that will be called before and after an operation, it's optional.
7. isAuthorized() function is for authorization if you need your application to authorize certain user access to information, it returns true by default, which means any user is authorized to access any data. You perform some checks in this function, and decide whether the user is authorized to access the data. The variable, `$operation` is used to tell you what table operation is performing, whether it's delete (OP\_DELETE), update (OP\_UPDATE), create (OP\_CREATE) or read (OP\_READ), variable `$condition` is used to add additional conditions into the queries. Once logic is done, you have to return true to authorize access or false to unauthorize the access.
8. You can either group multiple related tables (e.g. related by foreign keys) into a class extending Record or you can create a class extending Record for every tables you have in your database. However, I still find it more maintainable by creating each class for each table.
9. This is **NOT** a perfect design, but it should suit any small - medium projects. For larger projects, I probably would not use PHP, I would use a framework if I really need to. Also, the chosen architecture might not be suitable for this design.

With this design, you implement only the parts that differ while maintaining the parts of the code (A.K.A template method, it's a design pattern by G.O.F.), which minimizes codes to type. Also, if you want to change something that affects all record operations, you change only the base class, or if you wanted to add some functions for all record operations, you can just add it in the base class and it is available in the derived class. However, there is performance penalty for this design, especially when you create a class for each tables in your database, as always, object creation is expensive. But looking at the benefits it offers, I think it is worth the price.
