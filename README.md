# pgvector-vector-search-psql

vector search in pgvector using psql


# Create an Azure Database for PostgreSQL server (flexible)

https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/quickstart-create-server-portal

# Install psql (Local) 

brew services start postgresql@15

# Connect to the PostgreSQL database using psql 

https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/quickstart-create-server-portal#connect-to-the-postgresql-database-using-psql


# Execute following PSQL commands to review vector search 


# Enable Vector Extension and insert Vectors

`postgres=> CREATE DATABASE vectorpgsqldb;`

`postgres=> \c vectorpgsqldb`

psql (15.4 (Homebrew), server 15.3)
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off)
You are now connected to database "vectorpgsqldb" as user "citus".

`vectorpgsqldb=> CREATE EXTENSION vector;`

CREATE EXTENSION

`vectorpgsqldb=> CREATE TABLE items (id bigserial PRIMARY KEY, embedding vector(3));`

CREATE TABLE

`vectorpgsqldb=> INSERT INTO items (embedding) VALUES ('[1,2,3]'), ('[4,5,6]');`

INSERT 0 2

`vectorpgsqldb=> SELECT * FROM items ORDER BY embedding <-> '[3,1,2]' LIMIT 5;`

 id | embedding 
----+-----------
  1 | [1,2,3]
  2 | [4,5,6]
(2 rows)

`vectorpgsqldb=> INSERT INTO items (embedding) VALUES ('[1,2,3]'), ('[4,5,6]');`

INSERT 0 2

`vectorpgsqldb=> INSERT INTO items (id, embedding) VALUES (1, '[1,2,3]'), (2, '[4,5,6]')
    ON CONFLICT (id) DO UPDATE SET embedding = EXCLUDED.embedding;`

INSERT 0 2


# Querying Vector Data

Get the nearest neighbors to a vector

`vectorpgsqldb=> SELECT * FROM items ORDER BY embedding <-> '[3,1,2]' LIMIT 5;`

 id | embedding 
----+-----------
  3 | [1,2,3]
  1 | [1,2,3]
  4 | [4,5,6]
  2 | [4,5,6]
(4 rows)

Get the nearest neighbors to a row

`vectorpgsqldb=> SELECT * FROM items WHERE id != 1 ORDER BY embedding <-> (SELECT embedding FROM items WHERE id = 1) LIMIT 5;`
 id | embedding 
----+-----------
  3 | [1,2,3]
  4 | [4,5,6]
  2 | [4,5,6]
(3 rows)


Get rows within a certain distance

`vectorpgsqldb=> SELECT * FROM items WHERE embedding <-> '[3,1,2]' < 5;`
 id | embedding 
----+-----------
  3 | [1,2,3]
  1 | [1,2,3]
(2 rows)

Get the distance

`vectorpgsqldb=> SELECT embedding <-> '[3,1,2]' AS distance FROM items;`

     distance      
-------------------
 2.449489742783178
 5.744562646538029
 2.449489742783178
 5.744562646538029
(4 rows)


inner product

`vectorpgsqldb=> SELECT (embedding <#> '[3,1,2]') * -1 AS inner_product FROM items;`

 inner_product 
---------------
            11
            29
            11
            29
(4 rows)

cosine similarity

`vectorpgsqldb=> SELECT 1 - (embedding <=> '[3,1,2]') AS cosine_similarity FROM items;`

 cosine_similarity  
--------------------
 0.7857142857142857
 0.8832601106161003
 0.7857142857142857
 0.8832601106161003
(4 rows)

Average vectors

`vectorpgsqldb=> SELECT AVG(embedding) FROM items;`

      avg      
---------------
 [2.5,3.5,4.5]
(1 row)

Average groups of vectors
               ^
`vectorpgsqldb=> SELECT id, AVG(embedding) FROM items GROUP BY id;`

 id |   avg   
----+---------
  2 | [4,5,6]
  3 | [1,2,3]
  4 | [4,5,6]
  1 | [1,2,3]
(4 rows)

