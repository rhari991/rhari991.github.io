---
title: "Custom converters for Retrofit Part 2: Custom request converters"
excerpt: "Request converters let you modify how data is serialized into the body of network requests."
last_modified_at: 2018-02-07 10:00:00-04:00
categories:
  - android
tags:
  - java
  - android 
  - retrofit
  - converter
---

As mentioned in the first part of this series, request converters are responsible for creating the body of the actual request that is sent across the network.

A custom request converter should be used used when you need more control over how a request is created. For example, you may want to add custom encryption to your requests before they are sent. A request converter specifies how an object should be serialized into a [`RequestBody`](https://square.github.io/okhttp/3.x/okhttp/okhttp3/RequestBody.html) object that Retrofit can send across the network.

Similar to response converters, a request converter should impement the [`Converter<?, RequestBody>`](https://square.github.io/retrofit/2.x/retrofit/) interface where `?` is the type of the model object that is being serialized. The `convert()` method of this interface defines the conversion logic. Let's look at an example.

Have a look at the [endpoint](https://pastebin.com/api#2) of the Pastebin API responsible for creating new pastes. It requires the request body to be form encoded and has multiple optional arguments. Ideally, a caller should be able to provide any number of the optional arguments in an easy manner. There are several possible ways to do this :

- Define methods containing the required argument as well as all possible combinations of the optional arguments as shown below :

<script src="https://gist.github.com/rhari991/03adf3109571931febab53de5c1122a7.js"></script>

Unfortunately, this is a bad idea for endpoints containing a large number of arguments since it litters the API interface with very similar methods.

- Retrofit provides an option for this usecase through the [`@FieldMap'](https://square.github.io/retrofit/2.x/retrofit/retrofit2/http/FieldMap.html) annotation. This requires you to add each argument as a key/value pair in a map, and then pass that map to Retrofit as shown below.

<script src="https://gist.github.com/rhari991/35bf23fa7f0f86e79b3b07db4a43dd0c.js"></script>

While this does reduce the number of lines that are needed to represent the optional arguments, it also puts a requirement on the caller to remember the keys to which each optional argument maps. Also, it does not ensure that the required arguments are present in the request before it is sent.

- A more elegant way to do this is to use a combination of the Builder pattern and a custom request converter. The builder pattern is perfect for optional arguments as the methods are self-descriptive. The built object can be passed to a request converter, which builds a request body using the provided arguments.

Let's look at how this can be implemented.

<script src="https://gist.github.com/rhari991/1253cfd2a10cad2dde1af4deeb286da9.js"></script>

As shown above, the builder class accepts and validates the required argument through it's constructor, while the instance methods are used to provide the optional arguments.

Rather than url encode the arguments ourselves, we can use the handy [`FormBody`](https://square.github.io/okhttp/3.x/okhttp/okhttp3/FormBody.html) class that is a part of OkHttp. It is a subclass of `RequestBody` and automatically encodes the key/value pairs provided to it. So we simply need to return an instance of `FormBody` containing the provided arguments.

<script src="https://gist.github.com/rhari991/3e961f3f9797d69706ee4176ddda44e9.js"></script>

In lines 8 - 19, we simply check if the caller provided each of the optional arguments, and if so, add them to the 	`FormBody` instance.

Now the API caller can use the endpoint in a simple and intuitive manner.

<script src="https://gist.github.com/rhari991/67afd7d850e58e806b39386aa13b0f16.js"></script>

In the last part of this series, we will look at how converters should be registered with Retrofit so that they can be used when needed.