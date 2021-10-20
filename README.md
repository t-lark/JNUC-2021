# JNUC-2021
## JNUC 2021 resource companion

### About This Repo

This repo is meant as a companion item to augment my JNUC 2021 virtual conference talk. This repo will be a very basic intro to using SQL, and is to be taken conceptually. No code, process, workflow, or otherwise is meant to be taken as a best practice, or even great advice. While I do have some basic skills in SQL, I am not an expert, and am still learning in my data journey. Please keep this in mind while consuming any data in this repo. It was recommended by Jamf that I write something up that is shareable to the audience on basic usage, and this the intended purpose of this repo. Thank you for taking the time to read this and if you wathced my presentation, another thank you in advance. If anyone would like to add to this in any way PRs are welcome.

### Basic Syntax Overview

SQL code can be written many different ways, and like any language it can adhere to different types of code styles. Code styles, linters, and similar things are a matter of opinion, and often used to bring standards to code. Whether the code is meant for an open source project, work related, community driven, or another use case, having a standardized style helps keeps things unified. I will be focusing on reading the data in this repo with `SELECT` statements since this is the most common way any IT and Operations person would use a SQL based data tool. There are a lot more things one can do besides consuming data, but to keep this on topic with my JNUC 2021 presentation, all of my slides, code examples, and talking points are around just consuming data.

[Here](https://docs.snowflake.com/en/sql-reference/constructs.html) is the documentation around `SELECT` in Snowflake.

> SQL is not case sensitive 
> 
> You do not need double quotes in your path to the table, but the built in editor will add them 
> 
> I am using both with and with out quotes to simply illustrate both options are fine to use

example:
```sql
SELECT * FROM FOO;

-- and in lowercase

select * from foo;
```
In the above example, the code is exactly the same, with the exception that the top example is in all uppercase characters, and the bottom example is in all lowercase. Both examples will work, but ideally you will want to choose to use one or the other. So, all of your code is at least unified in a similar fashion. This makes reading the code later on, or if another human were to read it, easier to consume.

#### Basic Anatomy of a SELECT statement

When using SQL to `SELECT` data from tables, columns and rows, here is a basic query to count Jamf Webhook Events:

```sql
// can be a comment
-- can be a coment as well 
// in this code example, there is a actually some sub queries which are selecting the count (number of rows)
// returned from a table
SELECT (
    -- get the number of API Operation webhooks in the last hour
    SELECT COUNT(*) from "DB"."SCHEMA"."JAMF_APIOPS"
    -- filter by timestamp in epoch compared to the dateadd function, limited to hours from the current timestamp
    where TO_TIMESTAMP(WEBHOOK:eventTimestamp::INTEGER/1000) > dateadd(hours, -1, current_timestamp())
    -- we want to name the returned data in a column labeled APIOPS so we used as <the-name-you-want>
    ) as APIOPS,
    (
    SELECT COUNT(*) from "DB"."SCHEMA"."JAMF_CHECKIN"
    where TO_TIMESTAMP(WEBHOOK:eventTimestamp::INTEGER/1000) > dateadd(hours, -1, current_timestamp())
    ) as CHECKINS,
    (
    SELECT COUNT(*) from "DB"."SCHEMA"."JAMF_INVENTORY"
    where TO_TIMESTAMP(WEBHOOK:eventTimestamp::INTEGER/1000) > dateadd(hours, -1, current_timestamp())
    ) as INVENTORIES,
    (
    SELECT COUNT(*) FROM "DB"."SCHEMA"."JAMF_POLICIES"
    where TO_TIMESTAMP(WEBHOOK:eventTimestamp::INTEGER/1000) > dateadd(hours, -1, current_timestamp())
    ) as POLICIES,
    (
    SELECT COUNT(*) FROM "DB"."SCHEMA".."JAMF_CHECKIN"
    where TO_TIMESTAMP(WEBHOOK:eventTimestamp::INTEGER/1000) > dateadd(hours, -1, current_timestamp())
    AND EVENT:trigger = 'enrollmentComplete'
    ) as ENROLLMENT
    -- a ; ends the query statement
    ;
```
results

| APIOPS | CHECKINS | INVENTORIES | POLICIES |
|-------- | -------- | ----------- | -------- |
| 0 | 3081 | 190 | 405 |

Breaking down the exmaple above, we can look at each part and give some examples:

`SELECT FOO, BAR, BAZ FROM DB.SCHEMA.EXAMPLE;` will `SELECT` the columns `FOO`, `BAR`, and `BAZ` from a table named `EXAMPLE` in the Database and Schema path.

`SELECT * FROM DB.SCHEMA.EXAMPLE;` will return all the data from the table named `EXAMPLE` as `*` is considered a wildcard, much like other programming languages that also use wildcards

`SELECT COUNT(*) FROM DB.SCHEMA.EXAMPLE;` will return the total row count of the table `EXAMPLE`

```sql
SELECT 
    FOO
  , BAR
  FROM FROM DB.SCHEMA.EXAMPLE
  WHERE FOO = '1' AND BAR != 1
  ;
```
In the above example we are selecting the columns `FOO` and `BAR` from `EXAMPLE`, but only returning results where `FOO` as a string value of 1, and `BAR` does not have an integeter value of 1. I will briefly go over data types in a later section.

Writing queries in multiple lines can help when they get complex, you are out of your depth, or trying to debug something that isn't working:

```sql
SELECT 
    FOO
  --, BAR
  FROM FROM DB.SCHEMA.EXAMPLE
  WHERE FOO = '1' --AND BAR != 1
  ;
```
In the above example, I split out my query in my editor, IDE, or with in the Snowflake Worksheets interface in a manner where I was not getting the proper results or I had a syntax error, I can easily comment out my lines of code and get the last known working state of my query. This is just a general tip which I have found useful when learning how to write SQL code.

When using `FROM` you want to use the full path of your table. In Snowflake there are so many ways to store and manage your data I cannot possibly go over all of them, but the general concept is you have a database, that database has schemas, with in those schemas you can have tables. So, for the styanx it is simply `<DB-NAME>.<SCHEMA-NAME>.<TABLE-NAME>`

The `WHERE` clause in SQL is used to filter out results to get the exact data you need. You can use a `WHERE` clause in many different ways, and you can use them in conjuntion with other operators like `AND` or `OR` in your query to meet multiple conditions.

Queries are ended by a semicolon in code. This is how you terminate the end of your query, `SELECT * FROM DB.SCHEMA.FOO;`

### Data Types and Type Casting

Like many programming languages out there, `SQL` has data types and type casting, to see the full list of data types documenation click [here](https://docs.snowflake.com/en/sql-reference/intro-summary-data-types.html). If you have used other programming lanauges you are probably familiar with data types, some quick examples are, but not limited to:
* strings
* integers
* boolean
* lists or arrays
* dictionaries
* floats

In Snowflake you can type cast by simply using `::type` in your `SELECT` statements, for example:

```sql
SELECT 
    FOO::varchar
  , BAR::integer
  FROM FROM DB.SCHEMA.EXAMPLE
  WHERE FOO = '1' AND BAR != 1
  ;
```
In the above query I am type casing `FOO` as a `VARCHAR` data type, and `BAR` as an `INTEGER` data type. You might have noticed in my Jamf webhooks query example at the beginning of this doc, I am type casting the EPOCH date stamp as an integer and dividing it by 1000, to account for the different type of epoch time that is natively used from the Jamf Pro webhook and the Snowflake data platform. However, there are other built in types you can also leverage, lets take another look at that query:

```sql
SELECT (
    SELECT COUNT(*) from "DB"."SCHEMA"."JAMF_APIOPS"
    where TO_TIMESTAMP(WEBHOOK:eventTimestamp::VARCHAR::TIMESTAMP_LTZ) > dateadd(hours, -1, current_timestamp())
    ) as APIOPS,
    (
    SELECT COUNT(*) from "DB"."SCHEMA"."JAMF_CHECKIN"
    where TO_TIMESTAMP(WEBHOOK:eventTimestamp::VARCHAR::TIMESTAMP_LTZ) > dateadd(hours, -1, current_timestamp())
    ) as CHECKINS,
    (
    SELECT COUNT(*) from "DB"."SCHEMA"."JAMF_INVENTORY"
    where TO_TIMESTAMP(WEBHOOK:eventTimestamp::VARCHAR::TIMESTAMP_LTZ) > dateadd(hours, -1, current_timestamp())
    ) as INVENTORIES,
    (
    SELECT COUNT(*) FROM "DB"."SCHEMA"."JAMF_POLICIES"
    where TO_TIMESTAMP(WEBHOOK:eventTimestamp::VARCHAR::TIMESTAMP_LTZ) > dateadd(hours, -1, current_timestamp())
    ) as POLICIES;
```
results

| APIOPS | CHECKINS | INVENTORIES | POLICIES |
|-------- | -------- | ----------- | -------- |
| 0 | 3081 | 190 | 405 |

In a Jamf Webhook, the data is shipped in a JSON format. Snowflake can store JSON data natively as a [variant data type](https://docs.snowflake.com/en/sql-reference/data-types-semistructured.html#variant). So, one could take the Epoch time stamp that is shipped from a Jamf webhook, stored in Snowflake as a `VARIANT` data type, and type cast it to a `VARCHAR` (a string essentially) and then since [TIMESTAMP_LTZ](https://docs.snowflake.com/en/sql-reference/data-types-datetime.html#timestamp-ltz-timestamp-ntz-timestamp-tz) can convert a string type data into an actual date string, I can get away with using this over using math. Prety neat!

### Variant Data 

> note: variant data uses double quotes, string/varchar data uses single quotes. This might confuse you at first, this definitely confused me in the beginning


Since almost all API data and all Webhook data can be consumed/shipped as XML or JSON, you will likely be working with `VARIANT` data types. This is probably extremely common in the IT and Operations world. From my own anecdotal experience, almost every REST API I have worked with, and every App that can ship event hooks, has almost always been in JSON, sometimes in XML.

Example Webhook from Jamf (taken from their developer resources web page):
```json
    "event": {
        "computer": {
            "alternateMacAddress": "72:00:01:DD:A0:B9",
            "building": "Apple Park",
            "department": "Executive",
            "deviceName": "Tim's MacBook Pro",
            "emailAddress": "tim.cook@company.com",
            "jssID": 13,
            "macAddress": "60:03:08:A3:64:9D",
            "model": "13-inch Retina MacBook Pro (Late 2013)",
            "osBuild": "16G29",
            "osVersion": "10.15.7",
            "phone": "555-472-9829",
            "position": "CEO",
            "realName": "Tim Cook",
            "room": "1",
            "serialNumber": "C02M23PGFH50",
            "udid": "EBBFF74D-C6B7-5599-93A9-19E8BDEEFE32",
            "userDirectoryID": "-1",
            "username": "tim.cook"
        },
        "trigger": "CLIENT_CHECKIN",
        "username": "Tim Cook"
    },
    "webhook": {
        "eventTimestamp": 1553550875590,
        "id": 7,
        "name": "Webhook Documentation",
        "webhookEvent": "ComputerCheckIn"
    }
```

Assuming we have the raw JSON data in a column named `JSON_DATA` stored from a data ingest, we can look at executing this query:

```sql
SELECT 
  JSON_DATA:event.computer.serialNumber as SERIAL_NUMBER
, JSON_DATA:event.computer.osVersion as OS
FROM "JAMF_EVENTSDB"."TESTING"."JAMF_EVENTS_CHECKINS"
WHERE JSON_DATA:event.computer.username = 'tim.cook'
;
```
results:
| SERIAL_NUMBER |	OS |
| --- | --- |
|"C02M23PGFH50" |	"10.15.7" |
|"C02M23PGFH50"|"10.15.7" |
|"C02M23PGFH50" |	"10.15.7" |

Since Snowflake can natively parse `VARIANT` data and JSON the syntax is quite simple. To start to parse down keys in a JSON doc, simpley add `:` (colon) to your query to tell the SQL interpertter that we are now in variant data land. From there the same dot `.` syntax is fine to go down the next level of keys, like you would parse JSON or dictionary data in other languages. However, you might have noticed the double quotes around my data, which indicates that is a variant data type in Snowflake. To remove those we can do, you guessed it, type casting! 

example:
```sql
SELECT 
  JSON_DATA:event.computer.serialNumber::varchar as SERIAL_NUMBER
, JSON_DATA:event.computer.osVersion::varchar as OS
FROM "JAMF_EVENTSDB"."TESTING"."JAMF_EVENTS_CHECKINS"
WHERE JSON_DATA:event.computer.username = 'tim.cook'
;
```
results:
| SERIAL_NUMBER |	OS |
| --- | --- |
| C02M23PGFH50 |	10.15.7 |
| C02M23PGFH50 | 10.15.7  |
| C02M23PGFH50 |	 10.15.7 |

So, now we no longer have quotes and the data type is now a `VARCHAR` (string) and can be maniuplated in more ways now that it is a string if we chose to do so. Finally, there is another way to accomplish the same outcome of the same query, but with a slightly different syntax. This is a personal choice, either of these methods work just fine, but depending on your programming background you may prefer to use a different syntax.

```sql
SELECT 
  JSON_DATA['event']['computer']['serialNumber']::string as SERIAL_NUMBER
, JSON_DATA['event']['computer']['osVersion']::string as OS
FROM "JAMF_EVENTSDB"."TESTING"."JAMF_EVENTS_CHECKINS"
WHERE JSON_DATA:event.computer.username = 'tim.cook'
;
```
results:
| SERIAL_NUMBER |	OS |
| --- | --- |
| C02M23PGFH50 |	10.15.7 |
| C02M23PGFH50 | 10.15.7  |
| C02M23PGFH50 |	 10.15.7 |

You may find this type of syntax more familar if you have used say Python dictionaries or say hashes in Ruby. I also used the type cast of `::string` instead of `::varchar` and those two type are synonymous with each other, they are the same thing. 

### Key Takeaways

This tiny intro to using SQL is such a small piece of information in the greather schema of data and SQL, that this should get you a tiny idea of where to start. To avoid writing an entire novel, I will stop here. This was meant to augment my JNUC 2021 virtual conference talk which I will link once it is publicly available. The scope was to focus in on basic `SELECT` syntax and working with JSON data, as that will be the bulk of what you work with when consuming data from the Jamf Pro API, or a Jamf Pro webhook event. 

No one is born with this knowledge, and skills take time and effort to develop. If you have some experience in any OOP language, or even basic SQL concepts, you should be able to get onto a path of learning how to use SQL in cloud data platforms, like Snowflake. Do not feel discouraged if you have limits on what you can do in the beginning, or if you have issues grasphing how it works. We all do, we all have to take the time to learn it. So, my best advice I can give to any beginner, as a beginner myself, is to just keep at it. 
