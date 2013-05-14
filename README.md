# Xively wrapper for Java (Xively4J)

## Overview

This is a RESTful Java client for accessing Xively API. It uses [Apache HttpComponent](http://hc.apache.org/) for handling HTTP requests while remaining decoupled.
This fully featured Java library parses domain objects to and from Xively JSON data format.

## Quickstart

To get started:

- Update `api.key` in `src/main/res/config.properties` with your Xively API key 
- Call `XivelyService.instance()` and you will get access to all operations

All configuration for the library is contained within `AppConfig` class.
These settings are loaded from `config.properties`, which override the defaults defined in `AppConfig`.

Simple timeouts and retries can be configured via `config.properties`, for example:

```Java
http.connectionTimeout=5000
```

The connection timeout can also be set dynamically by calling:

```Java
HttpClientBuilder.getInstance().setConnectionTimeout(5000);
```

## Main Class Summary

The _DomainObject_ is the model interface for all objects that can be directly accessed/modified via Xively RESTful API.
Each model has a equals, hashcode and deepEquals defined to streamline downstream processing.

All CRUD operations on _DomaimObject(s)_ are provided by `XivelyService`.

It is designed to be fluent, here are some examples:

```Java
XivelyService.instance().feed().create(<Feed object>);
XivelyService.instance().datastream(feedId).create(<Datastream object>);
XivelyService.instance().datastream(feedId).delete(datastreamId);
```

Each _DomainObject_ has a corresponding requester interface for accessing the API in `com.xively.client.http.api` package for all CRUD operations, and the access to these is made fluent via the user of the `XivelyService`.
Each of the requester will provide access to all available API endpoints for the corresponding resource.

Method calls take _DomainObject(s)_ as parameters and return _DomainObject_, therefore downstream application does not need to be concerned about parsing or the underlying HTTP request/response details.

***On success***, the requester implementation will return the object post CRUD operation:

- _create_ - returns the _DomainObject_ created, fully populated with fields generated post API call
- _get (read)_ - returns the _DomainObject_ retrieved
- _update_ - returns the updated _DomainObject_
- _delete_ - returns an empty _DomainObject_ with the ID only

***On failure***, the requester implementation will throw:

- `InvalidRequestException`, if the request is invalid
- `HttpException`, if the response status is not _2xx_

RESTful requests to Xively API are managed by `DefaultRequestHandler` and `DefaultResponseHandler` handles the responses.
Therefore, this client is fully decoupled from the HTTP client implementation.
Parsing to and from _DomainObjects_ to HTTP request/response body are encapsulated in `com.xively.client.http.util.ParseUtil`.
It may throw:

- `ParseToObjectException`, if the returned response cannot be parse into DomainObject implementations
- `ParseFromObjectException`, if the DomainObject implementation cannot be parse into specified data format (e.g. JSON)

Any exception thrown out of the library is a subclass of `XivelyClientException`.


## More Example

Retrieve a feed:

```Java
Feed feed = XivelyService.instance().feed().get(123);
// the returned object will be populate with fields generated by the API
```

Create several datapoints and then put them into the same datastream:

```Java
Datapoint dp1 = new Datapoint();
dp1.setAt("2013-01-01T00:00:00.000000Z");
dp1.setValue("123");

Datapoint dp2 = new Datapoint();
dp2.setAt("2013-01-02T00:00:00.000000Z");
dp2.setValue("456");

// assuming your API key has permission to write
// to the feed:123 and datastream:"test_stream0"
DatastreamRequester requester = XivelyService.instance().datapoint(123, "test_stream-0");
requester.create(dp1, dp2);

dp1.setValue("234");
requester.update(dp1);
```

## Generating Documentation

Run this to regenerate docs:

```Bash
javadoc -d doc/ \
	-sourcepath src/main/java/ \
	-subpackages com.xively.client \
	-stylesheetfile src/main/res/doc_stylesheet.css
```