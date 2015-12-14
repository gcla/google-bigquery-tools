<h1>BigQuery Python App Engine Example:<br /><i>Top 100 SF and Fantasy According to NPR</i></h1>





# Introduction #

Google BigQuery provides REST APIs that you can call from a variety of application platforms, including Google App Engine, to create your own custom tool for working with big data sets.  _Top 100 SF_ is a BigQuery example that runs on Python App Engine and illustrates the steps to configure an application and make queries.

This example assumes that you already know how to create a Python App Engine application.  If you need to brush up on this, please see [Getting Started: Python](http://code.google.com/appengine/docs/python/gettingstarted/) in the App Engine documentation.  You should also have some familiarity with the structured query language, [SQL](https://code.google.com/apis/bigquery/docs/query-reference.html).  We will use [OAuth2](http://code.google.com/p/google-api-python-client/wiki/OAuth2) for authentication and authorization, but you don't need to know the details of that protocol to make this example work -- the libraries take care of most of the auth work for you.

## Getting the Code ##

The source code for this example is located in [the samples directory](http://code.google.com/p/google-bigquery-tools/source/browse/#git%2Fsamples%2Fpython%2Fappengine-bq-join) of this project's `git` repository.

This code does not include all the supporting libraries required.  Please see the section below, **[Supporting Libraries](#Supporting_Libraries.md)**, for more information about installing the additional libraries.

## Getting the Data ##

This sample depends on you creating three tables in your own project's dataset.  The table data is include in the source code, located in the `tables/` subdirectory.  `tables/` also includes the table schema as both text and json formats.  The contents of the .txt files is handy for the UI, and the .json files are handy for the `bq` CLI.  If you have never uploaded data to BigQuery before, please see the [Hello BigQuery](http://code.google.com/apis/bigquery/docs/hello_bigquery.html#createtable) example for creating a table.

# Setup #
## API Project ##
To get started, you will need a project on the [Google apis console](https://code.google.com/apis/console) that is configured for BigQuery.

**During the Limited Availability Preview, you must have an invitation to use the BigQuery API.**  You can request an invitation via [this survey form](https://docs.google.com/spreadsheet/viewform?formkey=dGl4TUlob1RDRndMWVpIb21ORmJPZWc6MA).

Once you have received your invitation via email, you can use BigQuery in an existing API project or in a new project.

The following steps are the same as you might have followed for the [Hello BigQuery tutorial](http://code.google.com/apis/bigquery/docs/hello_bigquery.html) using the command line `bq` tool.  You can use the same project if you like.

To create a new BigQuery project:
  1. **Open the Google APIs Console.**
  1. **Click on the project dropdown button** on the top left of the page to create a new Google API project<br />![http://google-bigquery-tools.googlecode.com/git/samples/python/appengine-bq-join/images/image01.png](http://google-bigquery-tools.googlecode.com/git/samples/python/appengine-bq-join/images/image01.png)
  1. **Click 'Create'**
  1. **Click on 'Services'**
  1. Find _Google BigQuery API_ in the list (_might not be visible during the Limited Availability Preview_)
  1. Activate the _Google BigQuery API_ in the Services pane by clicking the Status button to toggle it to On.  **During the Limited Availability period this switch may not be visible on your project, but if you were invited to participate then your project will be enabled for BigQuery as if you had clicked the Status button to 'On'.**
  1. Find _Google+ API_ in the list (_might not be visible during the Limited Availability Preview_)
  1. Activate the Google+ API. This is not required for the BigQuery API, but we do use it in _Top 100 SF_

## Supporting Libraries ##
In addition to the App Engine SDK, you will also need the [Google API Client Library for Python](http://code.google.com/p/google-api-python-client/) (installation instructions [here](http://code.google.com/p/google-api-python-client/wiki/Installation)).  BigQuery is a supported API even though it may not be [listed](http://code.google.com/p/google-api-python-client/wiki/SupportedApis) yet.  This also installs:
  * [oauth2client](http://code.google.com/p/google-api-python-client/wiki/OAuth2Client) library for authorization and authentication via OAuth2.
  * [enable-app-engine-project](http://code.google.com/p/google-api-python-client/wiki/GoogleAppEngine) utility which copies all the required libraries into your app development directory.  This is going to be your best friend!  Copying or linking all the libraries by hand is tedious and error-prone, and this 'enable' app does all the copying for you.

# Running _Top 100 SF_ on the Development App Engine #
So, you've downloaded the source code to this example and you'd like to see it run in the development app engine server.  Here are the remaining tasks:
  1. Download the example source code.  Let's say you've kept this in its default development directory, `appengine-bq-join`
  1. Install the [Supporting Libraries](#Supporting_Libraries.md) as above.
  1. Run `enable-app-engine-project appengine-bq-join` to add all the required supporting library source code to the development directory.
  1. To configure your new or existing project for running _Top 100 SF_ on the development App Engine server, `dev_appserver.py`, you will also need to approve your development server for oauth callbacks.  This will allow _Top 100 SF_ to make authorized requests on behalf of other users. For more information see [Google APIs Console help](http://code.google.com/apis/console-help/).
    1. Go to the [Google APIs Console](https://code.google.com/apis/console).
    1. Click **API Access**.
    1. Click **Create an OAuth 2.0 client ID**.
    1. In the **Product Name** field enter `Top_ 100_SF`. Click **Next**
    1. Choose Application type **Web Application**
    1. Choose **http** protocol.
    1. Fill hostname with **`localhost:8080`**.
      * The Redirect URI should be `http://localhost:8080/oauth2callback`
    1. Click **Create client ID.**
    1. Go to **API Access** and you should see a box titled **Client ID for web applications**.
  1. Open for editing: `appengine-bq-join/client_secrets.json`
  1. Copy the value for **Client ID, Client secret** and **Redirect URIs** from the Google APIs console **Client ID for web applications** to `client_secrets.json`.  The results should look like this:
```
{
  "web": {
    "client_id": "999999999.apps.googleusercontent.com",
    "client_secret": "SOMESECRETSTRING",
    "redirect_uris": ["http://localhost:8080/oauth2callback"],
    "auth_uri": "https://accounts.google.com/o/oauth2/auth",
    "token_uri": "https://accounts.google.com/o/oauth2/token"
  }
}
```
  1. **Save `client_secrets.json`**
  1. **Open for editing**: `appengine-bq-join/main.py`
  1. **Find** `PROJECT_ID = 'MAKE_THIS_YOURS'` and **change** `MAKE_THIS_YOURS` into your project id.  There are two valid values for this id, which you can see in this picture:<br />![http://google-bigquery-tools.googlecode.com/git/samples/python/appengine-bq-join/images/image00.png](http://google-bigquery-tools.googlecode.com/git/samples/python/appengine-bq-join/images/image00.png)
  1. **Find** the following object and change `MAKE_THIS_YOURS` into the appropriate table names:
```
      tables = {'author':'MAKE_THIS_YOURS',
                'title': 'MAKE_THIS_YOURS',
                'genre': 'MAKE_THIS_YOURS'}
```
> > Please **change** these tables to match your own versions.
> > The format should be similar to `'author':'my_dataset.top100sf_author'` where `my_dataset` is the name of the dataset holding the author, title, and genre tables and `top100sf_author` is the name of the table holding the corresponding data.  This will be used in the query.
  1. **Save `appengine-bq-join/main.py`**
  1. Start up the development server with the _Top 100 SF_ app:
> > `dev_appserver.py appengine-bq-join`
  1. Open `http://localhost:8080` in your browser