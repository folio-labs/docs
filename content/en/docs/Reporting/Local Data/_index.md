---
title: "Adding Local Data"
linkTitle: "Local Data"
date: 2021-06-24
weight: 20
tags: ["parenttopic"]
---

As documented in the [user guide](https://github.com/library-data-platform/ldp/blob/1.1.11/doc/User_Guide.md#4-local-tables), it is also possible to load and create local data into your LDP. Besides using the local schema, you might consider setting up separate schemas for different user groups or purposes. 

## Using schemas
The concept of schemas allows you to organize tables and permissions within one database. In LDP we have initially four relevant schemas:

* public: contains all extracted tables and its current data from the bound FOLIO tenant
* [history](https://github.com/library-data-platform/ldp/blob/1.1.11/doc/User_Guide.md#5-historical-data): stores data that have been updated in the past or may no longer exist
* [folio_reporting](https://github.com/folio-org/folio-analytics/blob/release-1.0/sql/derived_tables/README.md): contains all [derived tables](https://github.com/folio-org/folio-analytics/tree/release-1.0/sql/derived_tables) created and supported by the community 
* [local](https://github.com/library-data-platform/ldp/blob/1.1.11/doc/User_Guide.md#4-local-tables): common area for report users to create or import own data

Beside existing schemas, you are free to create more schemas for your local needs (e.g., to provide data for different departments, to separate and protect sensitive data). 

Learn more about the schema concept and how to configure schemas using [Postgres’s Schema Documentation](https://www.postgresql.org/docs/11/ddl-schemas.html#DDL-SCHEMAS-PRIV).

For a granular set up of permissions see also the built-in [Roles](https://www.postgresql.org/docs/11/user-manag.html) and [Privileges](https://www.postgresql.org/docs/11/ddl-priv.html) concepts of Postgres. 

## Moving and loading data 
Loading and moving data into LDP is as simple as it is for databases in general.

For Postgres there are two common approaches:

* [COPY](https://www.postgresql.org/docs/11/sql-copy.html): SQL command for moving table data via csv files 
* [pg_dump](https://www.postgresql.org/docs/11/app-pgdump.html) / [pg_restore](https://www.postgresql.org/docs/11/app-pgrestore.html): Postgres command line tools for importing and exporting data 