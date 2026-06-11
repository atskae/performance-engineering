# Chapter 3: Data Models and Query Languages
**Data model**: a structured representation of data
* Differ by levels of abstractions (high to low):
    * Application developer: data structures/APIs on how to represent users, organizations, relationships
    * Represent those data structures in JSON/XML
    * " in bytes/memory
    * " hardware circuitry

## Relational Versus Document Models
* **Relational model**: data is organized by *relations* (tables in SQL), each *relation* is an unordered collection of tuples (a row in SQL)
    * ex) SQL
        * Proposed by Edgar Codd in the 1970s
* **Document model*: represent data as JSON
    * Schema is more flexible than the relational model
    * ex) MongoDB, Couchbase

### The Object-Relational Mismatch
* SW development is done in object-oriented programming
    * More natural to use document model than relational model of tables, rows, columns

#### Object-relational mapping (ORM)
* ORMs provide boilerplate code to translate relational to document model

#### The document data model for one-to-many relationships
(See diagram p.70-p71)

### Normalization, Denormalization, and Joins
* **Normalization**: assign an ID to a value of a field instead of the field being free-form
    * ex) "Washington DC" - ID=1, Name: "Tom", Location: 1
    * Requries table joins to get the mapping from ID to the value
    * Allow consistency, ease of updating
    * Fast to update, slow to query
    * Best for fast changing variables (ex. social media like counts, profile picture)
* **Denormalization**: allow free-form values for a field
    * ex) Name: "Tom", Location: "Washington DC"
    * ex) Name: "Bob", Location: "DC"
    * Slow to update (need to update the value in many places), fast to query (no ID lookups)

* *hydrating the ID*: performing a join to get the mapping from ID to human-readable information

How to choose between normalization and denormalization?
* Depends on cost of reads/writes, how frequent data is updated
* Might depend on the user (very popular users are treated differently)
