---
title: "FOLIO Reporting with the LDP"
linkTitle: "FOLIO Reporting with the LDP"
weight: 650
---

{{% alert color="info"%}}
<b>Note:</b> This documentation covers [LDP v1.1.11](https://github.com/library-data-platform/ldp/tree/1.1.11) and [FOLIO Analytics v1.0](https://github.com/folio-org/folio-analytics/tree/release-1.0), which were released at approximately the same time as the FOLIO Honeysuckle release.
{{% /alert %}}

The [Library Data Platform (LDP)](https://youtu.be/SM1vq0zvxsY) is a project that develops an open-source analytics platform which is utilized by FOLIO for reporting and analytics. LDP software is used by both the [FOLIO](http://folio.org/) and [ReShare](https://projectreshare.org/) projects to provide statistical and operational analytics. For FOLIO, the LDP software extracts data from FOLIO modules into a relational database (also called “the LDP”), making it possible to engage in complex reporting and analytics using FOLIO data. While some FOLIO apps have built-in reports or the ability to download results from a search and filter workflow, the LDP supports larger and more complex queries that combine data from across the FOLIO apps.

## Data Flow from FOLIO to the LDP

The basic flow of data from FOLIO to the LDP is described in Figure 1 below. The LDP extracts data from the database used by FOLIO apps. From the LDP, data flows into two types of queries: derived tables (which simplify the FOLIO data and save the results back into the LDP) and report queries (which meet specific needs of report users).

![The LDP software extracts data from the FOLIO database and loads into the LDP database. The FOLIO Analytics repository stores derived table queries, which add derived tables to the LDP database, and report queries, which build reports forreporting end users. The LDP database can also be used to store non-FOLIO data in custom tables.](/img/FOLIOReportingDataflow.png)

Figure 1: The FOLIO Reporting Dataflow.

## Features of the LDP

* Open-source software and query development.
* Built on trusted database platforms.
* Ability to create custom tables to import non-FOLIO data.
* Historical tracking of FOLIO records.
* Ability to connect from a wide variety of reporting applications.
* Ability to perform cross-app FOLIO queries.
* Ability to query FOLIO data without impacting production application performance.
* Ability to query a simplified FOLIO data model through the use of derived tables.

## FOLIO Data Coverage in the LDP

As of [version 1.1.11](https://github.com/library-data-platform/ldp/tree/1.1.11), the LDP pulls data from the following FOLIO modules:

* mod-circulation-storage
    * /cancellation-reason-storage/cancellation-reasons
    * /fixed-due-date-schedule-storage/fixed-due-date-schedules
    * /loan-policy-storage/loan-policies
    * /loan-storage/loans
    * /loan-storage/loan-history
    * /patron-action-session-storage/patron-action-sessions
    * /patron-notice-policy-storage/patron-notice-policies
    * /request-policy-storage/request-policies
    * /request-preference-storage/request-preference
    * /request-storage/requests
    * /scheduled-notice-storage/scheduled-notices
    * /staff-slips-storage/staff-slips
* mod-configuration
    * /configurations/entries
* mod-email
    * /email
* mod-feesfines
    * /accounts
    * /comments
    * /feefines
    * /feefineactions
    * /lost-item-fees-policies
    * /manualblocks
    * /overdue-fines-policies
    * /owners
    * /payments
    * /refunds
    * /transfer-criterias
    * /transfers
    * /waives
* mod-courses
    * /coursereserves/copyrightstatuses
    * /coursereserves/courselistings
    * /coursereserves/courses
    * /coursereserves/coursetypes
    * /coursereserves/departments
    * /coursereserves/processingstatuses
    * /coursereserves/reserves
    * /coursereserves/roles
    * /coursereserves/terms
* mod-finance-storage
    * /finance-storage/budgets
    * /finance-storage/fiscal-years
    * /finance-storage/fund-types
    * /finance-storage/funds
    * /finance-storage/group-fund-fiscal-years
    * /finance-storage/groups
    * /finance-storage/ledgers
    * /finance-storage/transactions
* mod-inventory-storage
    * /alternative-title-types
    * /call-number-types
    * /classification-types
    * /contributor-name-types
    * /contributor-types
    * /electronic-access-relationships
    * /holdings-note-types
    * /holdings-storage/holdings*
    * /holdings-types
    * /identifier-types
    * /ill-policies
    * /instance-formats
    * /instance-note-types
    * /instance-relationship-types
    * /instance-statuses
    * /instance-storage/instance-relationships
    * /instance-storage/instances*
    * /instance-types
    * /item-damaged-statuses
    * /item-note-types
    * /item-storage/items*
    * /location-units/campuses
    * /location-units/institutions
    * /location-units/libraries
    * /loan-types
    * /locations
    * /material-types
    * /modes-of-issuance
    * /nature-of-content-terms
    * /service-points
    * /service-points-users
    * /statistical-code-types
    * /statistical-codes
* mod-invoice-storage
    * /invoice-storage/invoice-lines
    * /invoice-storage/invoices
    * /voucher-storage/voucher-lines
    * /voucher-storage/vouchers
* mod-orders-storage
    * /acquisitions-units-storage/memberships
    * /acquisitions-units-storage/units
    * /orders-storage/alerts
    * /orders-storage/order-invoice-relns
    * /orders-storage/order-templates
    * /orders-storage/pieces
    * /orders-storage/po-lines
    * /orders-storage/purchase-orders
    * /orders-storage/receiving-history
    * /orders-storage/reporting-codes
* mod-organizations-storage
    * /organizations-storage/addresses
    * /organizations-storage/categories
    * /organizations-storage/contacts
    * /organizations-storage/emails
    * /organizations-storage/interfaces
    * /organizations-storage/organizations
    * /organizations-storage/phone-numbers
    * /organizations-storage/urls
* mod-users
    * /addresstypes
    * /departments
    * /groups
    * /proxiesfor
    * /users

\* - The LDP supports [direct extraction](https://github.com/library-data-platform/ldp/blob/1.1.11/doc/Admin_Guide.md#6-direct-extraction) for these tables, rather than using the module APIs.
