---
title: "Querying the LDP"
linkTitle: "Querying"
date: 2021-06-24
weight: 30
tags: ["parenttopic"]
---

## What is the FOLIO Analytics repository

The FOLIO Analytics repository contains reports and other analytics developed for FOLIO and designed to run on the Library Data Platform. Reports developed for FOLIO are stored as code written in Structured Query Language (SQL). The SQL queries can be opened inside of database querying software to pull data out of the LDP.

You can read an introduction to the repository at its [overarching README file](https://github.com/folio-org/folio-analytics/blob/release-1.0/README.md) on GitHub.  The file describes the following:

* The two types of SQL queries to be found in the repository:
    * **Report queries**, which you can copy and paste to run on your LDP.
    * **Derived table queries**, which simplify and speed up report queries. A *derived table* is simply a table created using data from one or more other tables. These queries are managed behind the scenes through LDP administration so that they run automatically and generate the derived tables in the database.
* Documentation for the queries.
* Examples of the software queries can be run in.
* How the queries are organized in the repository.

The first section below outlines how to make use of the report queries in the FOLIO Analytics repository to generate reports on FOLIO data. Other sections of the documentation cover [administration of derived tables](../administration/#setting-up-derived-tables) and [use of derived tables for ad hoc querying](#ldp-specific-query-guidance).

## Using queries from the FOLIO Analytics repository

Report queries in the FOLIO Analytics repository are laid out in a particular structure that will make it easy for you to find the various areas you need as you build your knowledge of SQL. 

* **Introductory comments.** Queries may start with a short block of text that is not part of the query. These “comments” often describe basic components of the query and give a brief description of the purpose.
* **Parameters.** To make things easier for the user, queries typically include a “parameters” section at the top that allows one to easily specify the values needed for filtering on a field.  
* **Subqueries.** Queries may include several groups of smaller queries, which may be referred to as subqueries. These subqueries help simplify and rearrange different parts of the database to make the final query easier. (Note: The official term for these parts of the query is *common table expressions*, or *CTEs*. Calling them subqueries is meant to emphasize the role they play in the larger query.)
* **Main query.** The main query determines the final look of the report. Under the keyword `SELECT`, you will see a list of the fields that will show up in the final report. After the keyword `FROM`, there is a list of the tables the fields are coming from. The keyword `WHERE` specifies the filters that should be applied to limit the rows in the report. Additional keywords may appear after the `WHERE` keyword to further customize the output of the report. Comments can appear throughout the query to provide instructions or clarifications.

### Locating queries in the repository

Report queries are stored in the sql/report_queries folder of the repository. To see an overview of the report queries in the repository, review the [README file in the report_queries folder](https://github.com/folio-org/folio-analytics/blob/release-1.0/sql/report_queries/README.md). You can also browse the subdirectories in the [sql/report_queries](https://github.com/folio-org/folio-analytics/tree/release-1.0/sql/report_queries) folder. Each subdirectory contains one or more SQL queries, along with documentation that describes the purpose and output of the queries.

### Running queries in a database query tool

Once you have located a desired report query, you can perform the following steps to run the query and generate a report:

1. Copy the query code from GitHub.
1. Open a database query tool.
1. Connect to the LDP.
1. Paste the SQL query code into a local file.
1. Run the SQL query.
1. Export the query results in the desired format.

The following section demonstrates this workflow using [DBeaver](https://dbeaver.io/), a database querying tool that has a free community edition and is available for Windows, Mac OSX and Linux operating systems.

### Example of running a query using DBeaver

#### Copy the query code from GitHub

1. In the **sql** folder of [the FOLIO Analytics Repository](https://github.com/folio-org/folio-analytics/tree/release-1.0/sql), click on the **report_queries** folder. 
1. Click on the subfolder for the report you are interested in. For this example, open the [ACRL Circulation query file](https://github.com/folio-org/folio-analytics/blob/release-1.0/sql/report_queries/acrl/circulation/acrl_circulation.sql) by clicking first on the **acrl** subdirectory and then on the **circulation** subdirectory, and finally clicking on the **acrl_circulation.sql** file.
1. To open the query file directly, click on the **raw** button located in the upper right corner of the file preview box.
1. To copy the query code, type Ctrl-A (Cmd-A on Mac) to highlight all of the text, followed by Ctrl-C (Cmd-C on Mac) to copy the text.

#### Open a database query tool
1. Install the [DBeaver community edition](https://dbeaver.io/download/) corresponding to your operating system.
1. Open DBeaver.

#### Connect to the LDP
1. To add your LDP connection, click on the **New Database Connection** button toward the top of the Database Navigator tab. It should look like an electrical plug with a plus sign.
1. In the **Select your database** window that pops up, click on the **PostgreSQL** symbol and then click **Next**.
1. Fill out the **connection dialog**:
    * You will need to get the following information from your local LDP administrator:
        * Host (typically looks like a URL, like ldp.institution.edu)
        * Port (typically 5432)
        * Database name
        * User name and password
        * SSL mode (will likely be “require”)
    * Note that a hosted LDP reporting database is currently available for the FOLIO community.  It provides access to data from the FOLIO reference environment folio-snapshot and is updated hourly. For login info, please review the [Reporting SIG wiki documentation](https://wiki.folio.org/display/RPT/FOLIO+Reporting+Reference+Environment).
1. In addition to the first page of connection details, you must click on the SSL tab to select “require” under **SSL mode**.
1. Finally, expand **Connection Settings** in the sidebar on the left and select the **Initialization** subheading. In the settings on the right, make sure the **Auto-commit** check box is selected.
1. When you are done setting up the connection, you can double click on the connection name in the **Database Navigator** tab to connect to the database.

#### Paste the SQL query code into a local file

1. To create a new script file, either click on the **New SQL Editor** button in the toolbar (it will look like a document with a plus sign) or select **New SQL** script from the **SQL Editor menu**.
1. If you have multiple databases, DBeaver may prompt you to select the one you want to query. Select the correct database and click on select. The new script window should show up on the right, with the script editor on the top and the results window (currently empty) on the bottom.
1. Paste the copied query code from GitHub into the script editor. (Once you paste in your copy of the script, you can change it however you want. This is your copy of the SQL.  Read more about [tailoring queries](#tailoring-queries) below, and note that you should pay special attention to the “parameters” section in your query.)
1. To save the query, select “Save As” from the “File” menu and navigate to your preferred directory, using a filename with “.sql” as the file extension.

#### Run the SQL query

1. To run the query, either click on the **Execute SQL Script** button on the left side of the script editor (it should be the third button from the top and look like a document with a “play” symbol inside of it) or select **Execute SQL Script** from the **SQL Editor** menu.
1. The results will hopefully then appear in the **results panel** below the script. Note: When querying parts of the database with a lot of data, like the inventory tables, there may be a long delay before results are returned.


#### Export the query results as a CSV

1. To export the results, right-click inside the results table and select **Export data…**.
1. Complete the data export wizard:
    1. Select “CSV” as the data transfer target type and click **Next >**.
    1. Adjust any data transfer settings and click **Next >**.
    1. Adjust any output settings (e.g., output directory, file name pattern) and click **Next >**.
    1. Confirm settings and click **Start** to export the file.

### Tailoring queries

Many queries allow you to specify the correct values for report filters by editing the “parameters” sections at the top of the query. **You should always review and update the parameters before running your query.** If you do not update the values in the parameters section, the default values will be used, and in many cases these may be inappropriate for the data in your LDP.

To edit the parameter values, all you need to do is type in the measure value of interest between the single quotation marks at the start of one or more parameter lines. The values must be typed in exactly as they appear in the database. Possible values might be suggested in query comments (although those examples may not be in use at your institution).  If you do not want to filter the data, you can remove anything between the single quotation marks.

To further tailor the query, consult the [introduction to this section](#using-queries-from-the-folio-analytics-repository) above to identify the different sections you may wish to review and modify. One edit you may want to make would be to add your own comments to guide yourself and others.  Comment text will be gray (in DBeaver) and will not affect how the query runs. There are two ways to make comments. Typing `--` will create a comment out of the rest of that particular line of the file.  To create a comment that spans multiple lines, use `/*` at the beginning of the comment and `*/` at the end of the comment.

Another edit you may want to make would be to remove a field from the report. To do this, you can look for the `SELECT` keyword in the main query section (which is usually where you will see the longest list of field names). Inside the list of fields after that `SELECT` keyword, you can delete the lines that list fields you do not want to include. Just make sure that the last item in the list, the one that appears right before the `FROM` keyword, is not followed by a comma. 

For more advanced query writing techniques, refer to the documentation on [ad hoc querying using LDP tables](#ad-hoc-querying-using-ldp-tables).

### Troubleshooting queries

#### What to do if you do not get any results

* Check to see if a parameter was entered by the report writer that does not apply.  If not, seek out additional training here or through your local (or the larger) FOLIO community. 

#### What to do if there are errors

* Check to see if the error message indicates that a derived table is missing. These derived tables are stored in a schema called **folio_reporting**, so that schema name followed by a table name might appear in the error message. If a derived table is missing, contact your LDP administrator.
* If you edited the query, check to make sure you don’t have a comma appearing at the end of a list, like the list of the fields that occurs after the `SELECT` keyword.
* If you can’t determine what is causing the error, consult with your local LDP administrator or either your local or the larger [FOLIO Reporting community](../support).

## Ad hoc querying using LDP tables

If the shared queries do not meet your needs, you can also develop your own “ad hoc” (or as needed) queries to pull data from the LDP. In addition to creating queries with different fields and table connections than the shared queries, ad hoc queries make it possible to connect FOLIO to other custom tables available in your local LDP. Because the LDP is built on standard relational database software, you can build ad hoc LDP queries in the same way you would build queries for any other database, such as writing an SQL query and using a database query tool to run the query. 


### Learning SQL

To develop ad hoc queries, you will need to write query scripts using Structured Query Language (SQL). The Reporting SIG has compiled a few resources for learning SQL (Table 1).

Table 1: A compiled list of SQL training resources.

|Training Resource|Description|
|---|---|
|Stanford courses: <ul><li>[Relational Algebra](https://lagunita.stanford.edu/courses/DB/RA/SelfPaced/about)</li><li>[Introduction to SQL](https://lagunita.stanford.edu/courses/DB/RA/SelfPaced/about)</li></ul>|These courses include an introduction to the algebraic query language that provides the formal foundations of SQL (Structured Query Language), as well as a basic introduction to SQL.|
|[The Data School: Learn Introductory SQL Concepts](https://dataschool.com/learn-sql/introduction/)|An interactive tutorial with an approachable style. The tutorial has built-in SQL evaluation, so you don’t need to set up a separate database tool to try the exercises.|
|[Select Star SQL](https://selectstarsql.com/)|An interactive book that teaches SQL concepts using real-world datasets and problems. The book has built-in SQL evaluation, so you don’t need to set up a separate database tool to try the exercises.|
|[SQL Murder Mystery](https://github.com/NUKnightLab/sql-mysteries)|The SQL Murder Mystery is designed to be both a self-directed lesson to learn SQL concepts and commands and a fun game for experienced SQL users to solve an intriguing crime. They also have a walkthrough for SQL beginners.|
|CodeAcademy:<ul><li>[Learn SQL course](https://www.codecademy.com/learn/learn-sql)</li><li>[SQL Commands](https://www.codecademy.com/articles/sql-commands)</li></ul>|A course called Learn SQL and a list of SQL Commands. Without a Pro account, course features are limited.|
|[Linked In Learning](https://www.linkedin.com/learning/)|Linked In Learning provides access to several courses on SQL at many levels of expertise. Requires a paid subscription.|


### LDP-specific query guidance

After learning how to use SQL, there are a few resources that outline specifics of how the LDP organizes FOLIO data.

* **[The LDP User Guide](https://github.com/library-data-platform/ldp/blob/1.1.11/doc/User_Guide.md).** This guide includes details about writing SQL that works for the LDP data model; note especially the sections describing the data model, JSON queries, and the differences between the relational attributes and JSON fields. The guide also includes a section that describes the [historical data functionality within the LDP](https://github.com/library-data-platform/ldp/blob/1.1.11/doc/User_Guide.md#5-historical-data), which allows users to compose queries that explore how FOLIO data records change over time.
* **[SchemaSpy](https://glintcore.net:8443/ldp/schemaspy/).** The Reporting SIG maintains a SchemaSpy installation attached to our LDP reference environment, which pulls data from the FOLIO snapshot reference environment. SchemaSpy gives a concise list of LDP tables and fields and can be helpful when developing queries, if your local LDP uses the same software version as the LDP reference environment.
* **[FOLIO Schema Parser](https://docs.google.com/spreadsheets/d/1m_Cq_GmZX37gJPEjVWt9eOLXskUjSLUb-8KapWj0SIw/edit#gid=1511890017).** The Reporting SIG’s lightweight FOLIO Schema Parser automatically populates a spreadsheet using FOLIO’s data schema documentation, connecting FOLIO fields to LDP tables and fields. It can be helpful as a tool for quickly looking up what fields are available from FOLIO apps and what LDP tables include those fields.
* **[FOLIO Analytics shared derived tables](https://github.com/folio-org/folio-analytics/tree/release-1.0/sql/derived_tables).** The derived tables developed for the LDP (found in the **folio_reporting** schema of the LDP) often serve as a better starting point for ad hoc queries than the FOLIO tables in the **public** schema. The derived tables combine and simplify the original FOLIO tables in ways that make query development much easier. You should work with your local LDP administrator to determine how your local LDP is using derived tables (e.g., what FOLIO Analytics release you are using, how frequently the derived tables are updated).

### Sharing ad hoc queries

If your ad hoc query might be of use to other institutions, we encourage you to consider submitting it to the folio-analytics repository. Our [contributing guidelines](https://github.com/folio-org/folio-analytics/blob/release-1.0/CONTRIBUTING.md) describe the requirements for new contributions to the repository.

### Tips for using DBeaver to write an ad hoc query

Developing ad hoc queries using DBeaver follows a similar workflow to the [example workflow](#example-of-running-a-query-using-dbeaver) above. You can either start with an existing report query or derived table query and modify it for your own uses, or you can write the SQL code from scratch.

An example of a simple ad hoc query might be:

```
SELECT
    group_name,
    COUNT(user_id) AS num_users
FROM
    folio_reporting.users_groups
GROUP BY
    group_name
;
```

This code specifies that the report should contain two columns: `group_name` and a column that stores a calculation of the count of values in the `user_id` column, which should appear in the query with the label “num_users.” The code then specifies that these columns are coming from the `folio_reporting.users_groups` derived table. Finally, it specifies that the data from the original table should be separated into separate groups using values from the `group_name` column, so that the `num_users` calculation is done separately for each group. The result is a table where each value of `group_name` is matched with a count of the number of users in that group.

As you are writing your query file in DBeaver, you may find it helpful to browse the LDP using the [Database Navigator](https://dbeaver.com/docs/wiki/Database-Navigator/) tab. For example, you can expand the connection, then expand the **Schemas**, then expand the **folio_reporting** schema, then expand **Tables** to see the available derived tables. Each table can be expanded to see its available columns. To browse the data in a table, right-click on a table and select **View Data**. Use the same procedure to browse the tables and columns available in the **public** schema.
