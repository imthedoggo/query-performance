# query-performance

### Create a table

	CREATE TABLE customers (
	user_id serial PRIMARY KEY,
	username VARCHAR ( 50 ) UNIQUE NOT NULL,
	password VARCHAR ( 50 ) NOT NULL,
	email VARCHAR ( 255 ) UNIQUE NOT NULL);


### Generate many entries for meaningful analysis

```
  DO
  $do$
    BEGIN 
     FOR i IN 1..100025 LOOP
      INSERT INTO public.customers(username, password, email)
  	  VALUES (i, 'pw', i);
     END LOOP;
    END
  $do$;
```
	
### Create index
```
CREATE INDEX idx_customers_email ON customers (email);
```

### EXPLAIN (FORMAT JSON) before the SELECT will run query performance analysis



#### Sequential scan, goes through all the table's records
```
SELECT user_id, username, password, email
FROM public.customers
```

#### Index scan, fetches data from the index related data structure and returns the touple identifier. The corresponding heap page is directly accessed to get the data
```
  EXPLAIN (FORMAT JSON) SELECT user_id, username, password, email 
	FROM public.customers
 	WHERE username = 'avgu';
  ```
	
#### Index-only scan
```
  EXPLAIN (FORMAT JSON) SELECT username 
	FROM public.customers
	WHERE username = 'avgu';
  ```
	
#### Sequential scan, when the queried field is not indexed
```
  EXPLAIN (FORMAT JSON) SELECT user_id, username, password, email
	FROM public.customers
 	WHERE password = 'pw';
  
	
  EXPLAIN (FORMAT JSON) SELECT user_id, username, password, email
	FROM public.customers
 	WHERE email = 'email' and password = 'pw';
  ```

#### Set workers for a faster execution of large tables

```
SET max_parallel_workers = 4;
SET max_parallel_workers_per_gather = 4;
SET max_worker_processes = 4;
  ```
	
