# Getting Started with BigQuery via the Command Line #

BigQuery aims to be a full-featured data platform and interactive analysis tool, and currently supports data import, export, and query. This is a brief guide to help you get started using BigQuery via the [`bq` command-line interface (CLI) tool](http://code.google.com/p/google-bigquery-tools/source/browse/#git%2Fbq).

Your data in BigQuery is represented as collections of **tables**, each of which is a structured collection of rows. Tables are grouped into **datasets**, which are an organizational tool and the primary unit of access control. Datasets themselves are attached to **projects**, which is the abstraction for a user, principal, or group shared by all Cloud Platform services. Operations performed by BigQuery are represented by **jobs**. For now, we'll just concentrate on tables and jobs.

This document is a quick tutorial on how to use some of the most common commands supported by the BigQuery CLI.

The BigQuery CLI has built-in help: you can simply type

`$ bq help`

for general help, or

`$ bq help <command name>`

for more detailed help on a specific command.

# Logging in #

You can use the `bq` tool without doing any setup, but it's convenient to set a few default options (such as your default project). You can do that easily by running:

`$ bq init`

This command will create an OAuth token in the file `~/.bigquery.token`, which will remain valid indefinitely.  It will also ask you to pick a default project so you don't have to specify a project id for each operation.  This can be overridden later with the `--project_id` flag.

# Working with Datasets #

Datasets are the fundamental unit of access control. Right now, you can gloss over these details and just put everything in one dataset. It's convenient to call that dataset `workspace`, as this is the default dataset that will be used by our web UI.

There are four important commands for managing your datasets: `list, create, describe, and delete`. We can create a dataset named `workspace`:

`$ bq mk -d workspace`

(If a dataset already exists with that name, you'll see an error.) You can see all of your datasets:

```
$ bq ls -d
   datasetId
 -------------
  dataset1234
  ...
  other_ds
  workspace
```

You can also delete a dataset:

`$ bq rm workspace`

This will always confirm your choice, unless you pass the `-f` flag.

However, if there are any tables attached to that dataset, the CLI will raise an error explaining that you can't delete a dataset with tables still attached. Passing the `-r` flag will also remove the tables in that dataset.

# Setting a default dataset #

If you find yourself working primarily in a single dataset (such as you will in the rest of the examples), add the `--dataset_id=workspace` flag to your `.bigqueryrc` file:

`$ echo dataset_id=workspace >> ~/.bigqueryrc`

If you set a default dataset\_id (or set it via the command line with `--dataset_id=`) you can leave out the dataset name when working with table identifiers in the rest of the examples.

# Working with tables #

Tables support all four of the same operations as datasets, as well as one more: `dump`, which shows you the schema for that table and its contents. Table names are only unique within a dataset, so in order to reference a table, you must also include the dataset name. In Bigquery, table references are of the form `<dataset>.<table>`.

Let's see a few examples:

```
$ bq mk workspace.table1
Table 'table "guid1234567:workspace.table1"' successfully created.
$ bq ls workspace
  tableId    
 --------- 
  table1  
$ bq rm workspace.table1
rm: remove table 'guid1234567:workspace.table1'? (y/N)
```

If you set a default dataset in your `.bigqueryrc` file, then the workspace. prefix is optional.

Of course, tables aren't very exciting until we put something in them.  So, on to the next section...

# Loading data into tables #

To load data into a BigQuery table, you'll need to first upload a CSV file to Google Storage. The examples here use gsutil, the Google Storage command-line interface.  (If you don't have gsutil yet, see the installation instructions.) You'll need to create your own bucket to store any CSV files in Google Storage, which you can do using the gsutil mb command. Below we'll use `gs://mybucket` wherever you need to insert your own bucket name.

To load data into a BigQuery table, use the `bq load` command.

To get you started, we have provided some sample inputs in the Google Storage bucket `gs://bigquery_samples` that already have the correct permissions.

Now let's run `bq load` to load data into a table:

```
$ bq load workspace.presidents gs://Bigquery_samples/presidents.csv Number:integer,Name,TookOffice,LeftOffice,Party
```

The schema we are using in this example is simple -- all fields are of type string, except for "Number".  Note that if you have a complex schema, you may want to store it in a file instead.  Example:

```
$ gsutil cp gs://Bigquery_samples/presidents_schema.json .
$ bq load workspace.presidents gs://bigquery_samples/presidents.csv presidents_schema.json
```

You now have a BigQuery table that holds the contents of that CSV file.  Let's check out the status of this table:

```
$ bq show workspace.presidents
{
  "creationTime": "1319219449136",
  "datasetId": "workspace",
  "id": "myproject:workspace.presidents",
  "kind": "bigquery#table",
  "lastModifiedTime": "1319219449136",
  "projectId": "myproject",
  "schema": {
    "fields": [
      {
        "mode": "REQUIRED",
        "name": "Number",
        "type": "INTEGER"
      },
      {
        "mode": "REQUIRED",
        "name": "Name",
        "type": "STRING"
      },
      {
        "mode": "REQUIRED",
        "name": "TookOffice",
        "type": "STRING"
      },
      {
        "mode": "REQUIRED",
        "name": "LeftOffice",
        "type": "STRING"
      },
      {
        "mode": "REQUIRED",
        "name": "Party",
        "type": "STRING"
      }
    ]
  },
  "selfLink": "https://www.googleapis.com/bigquery/v2beta1/projects/myproject/datasets/workspace/tables/presidents",
  "tableId": "presidents"
}
```

This output shows the schema for this table--in this case, five fields, one integer and four strings.

If you want to load your own data into Bigquery, you'll need to create a bucket (if you haven't already) and upload your data to Google Storage:

```
$ gsutil mb gs://mybucket
$ gsutil cp mydata.csv gs://mybucket
```


# Querying tables #

To run a query in Bigquery, use the `bq query` command:

```
$ bq query 'SELECT Name FROM workspace.presidents'
+------------------------+
|          name          |
+------------------------+
| George Washington      |
| John Adams             |
| Thomas Jefferson       |
| James Madison          |
| James Monroe           |
| . . .                  |
+------------------------+
```

This output shows the result schema (one string field called `Name`) as well as the actual result (the names of all 44 US presidents).

If tabular data is not what you want, you can use the `--format= flag` to request `CSV, JSON, pretty JSON`, or sparse output.  Example:


```
$ bq --format=json query 'SELECT name,party from workspace.presidents LIMIT 5'   
[{"party":"no party","name":"George Washington"},{"party":"Federalist","name":"John Adams"},{"party":"Democratic-Republican","name":"Thomas Jefferson"},{"party":"Democratic-Republican","name":"James Madison"},{"party":"Democratic-Republican","name":"James Monroe"}]
```

```
$ bq --format=csv query 'SElECT name,party from workspace.presidents LIMIT 5'   
name,party
George Washington,no party
John Adams,Federalist
Thomas Jefferson,Democratic-Republican
James Madison,Democratic-Republican
James Monroe,Democratic-Republican
```

By default, the results of your query aren't saved.  If you want to save the results of your query into another table, pass the `--destination_table` flag.  Example:

`$ bq query --destination_table=workspace.presidents_names 'SELECT Name FROM workspace.presidents'`

We can get the schema for this table as above:

`$ bq show workspace.presidents_names`

You can also delete that table when you're done with it:

`$ bq rm -t workspace.presidents_names`

Of course, you can also do more interesting queries:

`$ bq query --destination_table=workspace.shortest 'SELECT name, INTEGER(RIGHT(LeftOffice, 4)) - INTEGER(RIGHT(TookOffice, 4)) AS YearsInOffice FROM workspace.presidents ORDER BY YearsInOffice LIMIT 10'`

```
+------------------------+---------------+
|          name          | YearsInOffice |
+------------------------+---------------+
| James A. Garfield      |             0 |
| William Henry Harrison |             0 |
| Zachary Taylor         |             1 |
| John F. Kennedy        |             2 |
| Warren G. Harding      |             2 |
| Gerald Ford            |             3 |
| Millard Fillmore       |             3 |
| Martin Van Buren       |             4 |
| Franklin Pierce        |             4 |
| John Adams             |             4 |
+------------------------+---------------+
```

For a list of all the syntax supported in BigQuery queries, you can look at the BigQuery query reference.

# Extracting data from BigQuery #

Finally, we can extract data from a BigQuery table into a Google Storage CSV file--the opposite of the `load` operation. Just run the `bq extract` command as follows, and optionally use `gsutil` to copy the extracted data to your local disk:

```
$ bq extract workspace.presidents gs://mybucket/out.csv
$ gsutil cp gs://mybucket/out.csv .
```

# Shell Mode #

`bq` supports a REPL mode (REPL stands for "read-eval-print loop"), complete with tab command completion, readline support, and implicit queries.  Example:

```
$ bq shell
Welcome to BigQuery! (Type help for more information.)
guid426183296600> ls
   datasetId    
 -------------- 
  test_dataset  
  workspace     
guid426183296600> ls workspace
  tableId    
 ---------- 
  shortest  
  table1    
guid426183296600> select name from workspace.shortest limit 3;   
+------------------------+
|          name          |
+------------------------+
| James A. Garfield      |
| William Henry Harrison |
| Zachary Taylor         |
+------------------------+
```

# Personalizing bq #

If you're a regular `bq` user, you’ll find yourself growing tired of passing certain flags.  You can customize bq by adding persistent command line flags to your `~/.bigqueryrc` file.

If you find yourself working exclusively within a single dataset, you might want to pass both `project_id` and `dataset_id` via your `.bigqueryrc` file:

```
project_id=guid426183296600
dataset_id=test_dataset
```

You can override any of the values in .bigqueryrc by passing the flag explicitly.  Example: you can override dataset\_id by passing `--dataset_id=other_dataset`.

If you're debugging something, you might find the following useful:

`apilog=/tmp/bqlog`

You can also specify an alternate .bigqueryrc file by setting the `BIGQUERYRC` environment variable, or passing `--bigqueryrc=`.

# Asynchronous Operation #

Most of the features of `bq` are actually implemented upon an asynchronous substrate, but it hides some of that from you for convenience.  When you're automating interaction with Bigquery, this is probably not what you want.  You can instruct `bq` to be asynchronous with some special flags.

This is best illustrated with an example asynchronous `load` job:

```
$ JOB=$(bq --nosync load \
    presidents gs://bigquery_samples/presidents.csv \  
    presidents_schema.json | cut -f5 -d\ )
# do a bunch of other stuff...
# then wait until the query is done:
$ bq wait $JOB
$ bq query 'select party,tookoffice,name from workspace.presidents'
```

# Multiple Accounts #

If you're using multiple accounts (such as your Google Apps account and your gmail.com account), you can do so two ways:

1. Pass a unique filename to `--bigqueryrc=` for each identity, and in each file specify a different value of `--credential_fil`e. (The bq init command inserts the `--credential_file` for you if that's how your `.bigqueryrc` is created.) This will share nothing between the accounts. Example:

```
$ bq init --credential_file=$HOME/.bigquery.token-corp --bigqueryrc=$HOME/.bigqueryrc-corp
$ bq init --credential_file=$HOME/.bigquery.token-ext --bigqueryrc=$HOME/.bigqueryrc-ext
$ bq --bigqueryrc=$HOME/.bigqueryrc-ext query ...
```

2. Pass a unique filename to `--credential_file=` for each of your identities. This will mean they share all other settings in your `.bigqueryrc`. Example:

```
$ bq init --credential_file=$HOME/.bigquery.token-corp
$ bq init --credential_file=$HOME/.bigquery.token-ext
$ bq --credential_file=$HOME/.bigquery.token-ext query ...
```

If no `--bigqueryrc` flag is set, the default value of `~/.bigqueryrc` will apply unless you have set the `BIGQUERYRC=` environment variable.

# Getting Help #

Having problems?  Join our [bigquery-discuss mailing list](http://groups.google.com/group/bigquery-discuss), post your question there, and we'd be glad to help you.