# hive2elastic

hive2elastic synchronises [hive](https://github.com/steemit/hivemind)'s hive_posts_cache table to elasticsearch and keeps it updated.


## Before start

Some additional database objects have to be created on hive's database.

**Follow steps below:**

1- Stop hive. Make sure all hive processes stopped.

2- Create database objects on hive's database.

```
CREATE TABLE __h2e_posts
(
    post_id INTEGER PRIMARY KEY
);
```

```
INSERT INTO __h2e_posts (post_id) SELECT post_id FROM hive_posts_cache;
```

```
CREATE OR REPLACE FUNCTION __fn_h2e_posts()
  RETURNS TRIGGER AS
$func$
BEGIN   
    IF NOT EXISTS (SELECT post_id FROM __h2e_posts WHERE post_id = NEW.post_id) THEN
    	INSERT INTO __h2e_posts (post_id) VALUES (NEW.post_id);
	END IF;
	RETURN NEW;
END
$func$ LANGUAGE plpgsql;
```

```
CREATE TRIGGER __trg_h2e_posts
AFTER INSERT OR UPDATE ON hive_posts_cache
FOR EACH ROW EXECUTE PROCEDURE __fn_h2e_posts();
```

3- Start hive

**Make sure database credentials that you use has delete permission on __h2e_posts table**

## Installation

Run `pip3 install -e .` to install.

## Configuration

You can configure hive2elastic by these environment variables:


|	Argument	|	Environment Variable	|	Description | Default|
|	--------	|	--------	|	--------	|  --------	|  
|	--db-url	|	DB_URL	|	Connection string for hive database	| -- | 
|	--es-url	|	ES_URL	|	Elasticsearch server url	| -- | 
|	--es-index	|	ES_INDEX	|	 index name on elasticsearch	| hive_posts | 
|	--es-type	|	ES_TYPE	|	 type name on elasticsearch | hive_posts | 
|	--bulk-size	|	BULK_SIZE	|	 number of records indexed in a single loop | 500 | 
|	--max-workers	|	MAX_WORKERS	|	 max workers for converting process | 2 | 


## Example configuration and running

```
$ export DB_URL=postgresql://username:passwd@localhost:5432/hive 
$ export ES_URL=http://localhost:9200/
$ export BULK_SIZE=2000                 
$ export MAX_WORKERS=4

$ hive2elastic_post
```
