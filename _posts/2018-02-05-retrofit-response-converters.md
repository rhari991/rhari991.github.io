---
title: "Custom converters for Retrofit | Part 1 : Introduction and custom response converters"
excerpt: "Converters define the mapping between network requests/responses and model classes. Custom response converters can be used to handle edge cases in a server's API."
last_modified_at: 2018-02-05 10:00:00-04:00
categories:
  - android
tags:
  - java
  - android 
  - retrofit
  - converter
---

[Retrofit](https://github.com/square/retrofit) is an popular networking library for Java and Android applications. It is made up of several customizable components, such as converters and call adapters, which makes it adaptable to a number of scenarios. In this series of posts, we will look at how custom converters can be created.

Throughout this series, examples from the [Pasted library](https://github.com/rhari991/pasted) will be quoted. The library is a Java wrapper for the [Pastebin API](https://pastebin.com/api). The API is not very consistent with modern API specifications - for example, invalid requests sent by a client result a response code of 200 instead of 400. This makes it a great example to showcase the various customization options that Retrofit offers.

## What are Converters ?
Converters are the components of Retrofit that define the mapping between an application's model objects and the bits that are sent/received across the network. There are two types of converters :

* **Request Converters** define the format using which your data should be sent across the network. Retrofit already has in-built support for common formats such as form encoding and JSON encoding.

* **Response Converters** define how network responses should be deserialized into objects of a model class. For example, if a response is in JSON, then this is where it would be parsed into the Java object that is returned.

Retrofit uses [OkHttp's](https://github.com/square/okhttp) [`RequestBody`](https://square.github.io/okhttp/3.x/okhttp/okhttp3/RequestBody.html) and [`ResponseBody`](https://square.github.io/okhttp/3.x/okhttp/okhttp3/ResponseBody.html)classes to represent the data that is being transferred across the network. Converters transform model objects into these Retrofit objects and vice versa.

A common use case for custom converters is when an API has some sort of anomaly in the way it works, and so cannot be used directly with Retrofit. In such a case, a custom converter can handle the edge case and then delegate the remaining work to a standard converter.

## Writing a custom response converter
As mentioned earlier, response converters parse network responses into model objects that are returned to the caller. Retrofit offers a few [standard response converters](https://github.com/square/retrofit/tree/master/retrofit-converters) based on popular libraries such as Gson for JSON responses and SimpleXML for XML responses.

A custom response converter can be used when you need more control over the deserialization process. Some situations where this is required could be if your serialization format is not consistent or if the format varies slightly from the specification.

The response from the [trending pastes endpoint](https://pastebin.com/api#10) of the Pastebin API is a good example of this. This endpoint returns a series of XML objects enclosed in a pair of ```<paste>...</paste>``` tags as shown below :

<script src="https://gist.github.com/rhari991/4d3e031150aa70b50a26fb6ac608b27a.js"></script>

A natural return type to parse this into is ```List<Paste>```. However, note that this response is not valid XML, since there is no root element. This means that it cannot be parsed by an XML deserializer directly. Let's write a custom response parser to take care of this issue.

<script src="https://gist.github.com/rhari991/64a9d404d8ad69ede0ca6f2c6e6f41c3.js"></script>

A response converter should implement the [```Converter<ResponseBody, ?>```](https://square.github.io/retrofit/2.x/retrofit/) interface where ```?``` is the type of the model object that is returned by the converter. The ```convert()``` method receives the network response as a ```ResponseBody``` object and should contain the conversion logic.

In lines 11 and 12 of the above example, a string representation of the response is created after which a pair of ```<pastes>...</pastes>``` tags is appended to it to make the XML valid. This can now be passed safely to the XML parser ([SimpleXML](http://simple.sourceforge.net/home.php) in this case). 

However, ```SimpleXML``` cannot parse the response directly into a parameterized type like ```List<Paste>```. So, it is first parsed into a temporary wrapper object (```PasteGroup```). Then, the list is simply unwrapped and returned. This can be seen in lines 14 and 15. In this way we can ensure that the server response is parsed correctly.

This example also highlights an important advantage of using a converter - it abstracts any API-specific implementation details away from the caller. The caller only needs to know the return type of the endpoint while the hoops that we had to jump through to parse the server response remain hidden.

In the next part of this series, we shall see how to write custom request converters. Finally, we will see how converters are registered with Retrofit so that they can be used when required.







