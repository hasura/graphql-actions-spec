GraphQL
-------

# GraphQL Actions 

## Overview

A common usecase (in BFF) with GraphQL is calling an HTTP API for a mutation,
and using the response to trigger a Graphql query and returning that data to
the client as response. We call this an Action. This document standardizes the
client/server interface for Actions. This will allow building clients that are
not tied to the server implementation

Three aspects need standardizing

1. Error handling
2. Asynchronous execution
3. Consistent handling of read after writes

## Error handling

When the HTTP API returns a non 2XX status code or fails due to network errors,
an error must be added to the "errros" list in the response.

In the case of the HTTP API returning a non 2XX status code, the status code
is recommended to be provided in the errors document, For eg:

```
{
  "httpStatusCode": 401,
  "path": ["createUser"],
  "message": "HTTP Method call failed"
}
```

In addition if the HTTP server provides additional details for the error,
it is recommended that these details be added to the errors document.

For transient errors (Such as HTTP 503 Service Unavailable), the server should
retry the mutation. In such cases if the failure persists, the errors document
should containt the details of the last HTTP call

## Asynchronous execution of Actions

It is recommended that clients allow users to run actions asynchronously. If
the GraphQL server supports asynchronous execution of Actions, the client must
be able to request an actionId in response to the initiating mutation. Further
the client must be able to subscribe to updates on the action.

For example, to call an asynchronous mutation createUser, the client must
be able to do

```
mutation {
  createUser(name: "Ramesh") {
    actionId
  }
}

subscription ActionUpdates {
  actionUpdate(actionId: 123) {
    userId
    name 
  }
}
```

In the above example the server must send exactly one packet (for the
subscription) once the execution of mutation is completed followed by "no more
data" packet and then close the connection.

The packet sent must have the exact same format as the response would have been
if the mutation was synchronous in the first place.

## Consistent handling of read after writes

In a microservice based architecture it is possible that the mutation leads to
updates to data in multiple systems and the query response is also built by
fetching data from multiple systems. In these scenarios the GraphQL server must
makes sure that the response sent on the execution of an action (synchronously
or asynchronously) reflects the changes made to data in different systems.
