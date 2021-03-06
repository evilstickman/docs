---
title: Backand Documentation

language_tabs:
  - shell: cURL
  - javascript

toc_footers:
  - <a href='https://www.backand.com'>Sign Up for a free app</a>
  - <a href='https://github.com/backand'>View our samples on GitHub</a>

includes:
  - backand_features
  - backand_dashboard
  - sdk
  - platform_specific
  - common_use_cases
  - integrations

search: true
---

# Documentation Overview

##What is Backand?
Backand's goal is to free up the front end development of web applications by providing a rich, robust, and scalable back end with minimal impact on the development process. The key feature offered by Backand is the [ORM](http://en.wikipedia.org/wiki/Object-relational_mapping), but most application back ends require more than just object management. Backand provides you with most of what your application needs automatically, offering features like tracking data changes, logging, role-based security, back-office connectivity, and much more. You can even create your own server side actions with JavaScript, running custom server side queries and easily integrating with third party services.

###Backand ORM
Backand [ORM](http://en.wikipedia.org/wiki/Object-relational_mapping) automatically provides you with a REST API to perform [CRUD](http://en.wikipedia.org/wiki/Create,_read,_update_and_delete) operations against your database. You can connect an existing database you already have, or create a new database with the Backand dashboard. When you create a database at Backand it is automatically populated in Amazon's AWS Relational Database Service (AWS RDS), providing an isolated database server that you can scale automatically, or even use in other applications entirely independent of Backand. Even though Backand focuses on software services as opposed to platform services, when you register a database with Backand it is truly your database.

###SQL and NoSQL - The Best of Both Worlds
Through the flexibility of Backand's API, you have the ability to work with your data at whatever level you desire. You can perform basic [CRUD](http://en.wikipedia.org/wiki/Create,_read,_update_and_delete) operations in a couple ways. In one way you can mimic a SQL database by simply returning a shallow representation of the object, with foreign key references remaining as simple IDs in the response data. However, you can also perform deep queries that resolve all of the underlying data objects into a single set of response data, giving you the level of object detail that you often see with NoSQL databases. With a simple parameter change, you can switch between the two patterns at will!

## Using cURL with Backand
```shell
# Retrieve a list of items from your app
# This call stores the MASTER_KEY and USER_KEY into environment variables
# Following this pattern will enhance the security and reliability of any console
# calls made to the API
curl https://api.backand.com/1/objects/items -u $MASTER_KEY:$USER_KEY

# You can perform the same task using URL parameters:
curl https://api.backand.com/1/objects/items?authorization=basic+$MASTER_KEY:$USER_KEY
```

All calls made by our API can also be made on the command line using cURL. You simply need to either obtain a token to authenticate as a user with your application, provide a token allowing anonymous access to your app, or use your app's master key to override the authentication mechanism. Review our information on [authentication](#authentication) for more info on the specific endpoints to use for each approach.

<aside class="warning">Using the "master key" approach is very risky, as it completely bypasses the configured security in your application. We recommend only doing this while troubleshooting and exploring on the server - you should <strong>never</strong> use this technique on the client side unless you are fully aware of the consequences.</aside>

## Using our SDK with Backand
```javascript
// Initialize the SDK
backand.init({
  appName: 'APP_NAME',
  anonymousToken: 'ANONYMOUS_TOKEN'
});

// Retrieve a list of records from the server
backand.object.getList('users')
  .then((response) => {
      console.log(response);
  })
  .catch(function(error){
      console.log(error);
  });
```

We also offer a full-featured SDK that you can use to communicate with your Backand application. Getting started with the SDK is as simple as configuring your application access details, initializing the SDK, and calling a `getList()` function for one of the objects in your system. You configure the SDK with your application's `APP_NAME` and `ANONYMOUS_TOKEN`. Once you've called `init()` with these values, the SDK will use this data to automaticaly manage authentication headers for API requests to your app's REST API. See our [Vanilla SDK documentation](#vanilla-sdk) for more details.

## Response format
```JSON
// Sample response
{
  "totalRows": 2,
  "data": [
    {
      "__metadata": {
        "id": "1",
        "fields": {
          "id": {
            "type": "int",
            "unique": true
          },
          "items": {
            "collection": "items",
            "via": "user"
          },
          "email": {
            "type": "string"
          },
          "firstName": {
            "type": "string"
          },
          "lastName": {
            "type": "string"
          }
        },
        "descriptives": {

        },
        "dates": {

        }
      },
      "id": 1,
      "items": null,
      "email": "matt@backand.com",
      "firstName": "Matt",
      "lastName": "Billock"
    },
    {
      "__metadata": {
        "id": "2",
        "fields": {
          "id": {
            "type": "int",
            "unique": true
          },
          "items": {
            "collection": "items",
            "via": "user"
          },
          "email": {
            "type": "string"
          },
          "firstName": {
            "type": "string"
          },
          "lastName": {
            "type": "string"
          }
        },
        "descriptives": {

        },
        "dates": {

        }
      },
      "id": 2,
      "items": null,
      "email": "start@backand.io",
      "firstName": "Start",
      "lastName": "Backand"
    }
  ]
}
```
Our SDK will always respond to your requests with well-formed JSON representing the results of an action taken. We include metadata that can be used to parse the response where applicable, and this metadata is provided with each object in your system. Responses from the SDK wrap the HTTP response in a promise, allowing you to handle the results in an asynchronous manner.
