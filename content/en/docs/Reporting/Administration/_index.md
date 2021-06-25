---
title: "Administration"
linkTitle: "Administration"
date: 2021-06-24
weight: 10
tags: ["parenttopic"]
---

The Library Data Platform (LDP) is designed to run in different scenarios with only a few requirements. In order for the LDP to connect to FOLIO, it needs read-only access to Okapi and the FOLIO database. The LDP can be hosted and administered locally by an institution’s own staff or by contracting hosting services out to a third party. You can find specific [system requirements](https://github.com/library-data-platform/ldp/blob/release-1.1/doc/Admin_Guide.md#software) in the well-maintained LDP Github repository.

## Install and configure the LDP

For detailed information about how to set up and configure the LDP, please review the particular guides in its repository. There you also will find the latest versions and fixes of the LDP.  

### Administration Guide
* [Overview](https://github.com/library-data-platform/ldp/blob/1.1.11/doc/Admin_Guide.md#1-overview): brief introduction and summary
* [System requirements](https://github.com/library-data-platform/ldp/blob/1.1.11/doc/Admin_Guide.md#2-system-requirements): software and hardware requirements
* [Installation](https://github.com/library-data-platform/ldp/blob/1.1.11/doc/Admin_Guide.md#3-installation): installing prerequisites and building LDP
* [Database configuration](https://github.com/library-data-platform/ldp/blob/1.1.11/doc/Admin_Guide.md#4-database-configuration): setting up and configuring the database
* [Server configuration](https://github.com/library-data-platform/ldp/blob/1.1.11/doc/Admin_Guide.md#5-server-configuration): setting up, configuring, starting and updating the server
* [Direct extraction](https://github.com/library-data-platform/ldp/blob/1.1.11/doc/Admin_Guide.md#6-direct-extraction): setting up direct extraction from FOLIOs database
* [Data privacy](https://github.com/library-data-platform/ldp/blob/1.1.11/doc/Admin_Guide.md#7-data-privacy): options to anonymize personal data
* [Reference](https://github.com/library-data-platform/ldp/blob/1.1.11/doc/Admin_Guide.md#reference): reference of the configuration file: ldpconf.json

### Configuration Guide
* [Scheduling full updates](https://github.com/library-data-platform/ldp/blob/1.1.11/doc/Config_Guide.md#1-scheduling-full-updates): configuring full updates  
* [Foreign keys](https://github.com/library-data-platform/ldp/blob/1.1.11/doc/Config_Guide.md#2-foreign-keys): enabling capability to infer foreign keys
* [Reference](https://github.com/library-data-platform/ldp/blob/1.1.11/doc/Config_Guide.md#reference): reference of the configuration file: dbconfig.general

## Setting up derived tables
To let report users take full advantage of the community-developed [report queries](../querying/#using-queries-from-the-folio-analytics-repository) as well as having a good starting point for [ad hoc querying](../querying/#ad-hoc-querying-using-ldp-tables), it is strongly recommended to set up a nightly update on [derived tables](https://github.com/folio-org/folio-analytics/tree/release-1.0/sql/derived_tables). Note that the use of *views* and *materialized views* is not supported in LDP databases and may cause the LDP software to be unable to perform data updates.

You can find instructions in Github on how to set up [FOLIO Reporting Derived Tables (Guide)](https://github.com/folio-org/folio-analytics/blob/release-1.0/sql/derived_tables/README.md). 

## Data privacy 

The LDP is designed to support GDPR and other data privacy requirements. Administrators can exclude a predefined set of tables. 

See the [Anonymization Guide](https://github.com/library-data-platform/ldp/blob/1.1.11/doc/Admin_Guide.md#7-data-privacy) for information on how to activate and configure these features. 

The Reporting SIG has identified and documented attributes that contain potential personal data:

* [Users module](https://wiki.folio.org/display/RPT/Potential+personal+data%3A+List+of+FOLIO+attributes?src=contextnavpagetreemode): tables that will not loaded in LDP when anonymization is turned on
* [Organizations module](https://wiki.folio.org/display/RPT/Potential+personal+data+in+mod-organizations-storage?src=contextnavpagetreemode): potential personal data in organizations module
