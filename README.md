[![Build Status](https://travis-ci.org/englishtown/vertx-mod-jersey.png)](https://travis-ci.org/englishtown/vertx-mod-jersey)

# vertx-mod-jersey

Allows creating JAX-RS jersey resources that will handle incoming http requests to vert.x.


## Vertx Resource Injection

The javax.ws.rs.core.Context annotation can be used to inject vert.x objects into a resource constructor, field,
or method parameter.  Supported vert.x objects include
* org.vertx.java.core.http.HttpServerRequest
* org.vertx.java.core.http.HttpServerResponse
* org.vertx.java.core.streams.ReadStream<org.vertx.java.core.http.HttpServerRequest>
* org.vertx.java.core.Vertx
* org.vertx.java.platform.Container

To inject custom objects, you must provide one or more binders in the configuration.  See the integration test test_postJson() for an example.


### Example Resource Method
```java
@GET
@Produces(MediaType.APPLICATION_JSON)
public void getQuery(
        @Suspended final AsyncResponse response,
        @Context ContainerRequest jerseyRequest,
        @Context HttpServerRequest vertxRequest,
        @Context Vertx vertx,
        @Context Container container) {

    vertx.runOnLoop(new Handler<Void>() {
        @Override
        public void handle(Void aVoid) {
            response.resume("Hello World!");
        }
    });
}
```

## Configuration

The vertx-mod-jersey module configuration is as follows:

```json
{
    "host": "<host>",
    "port": <port>,
    "receive_buffer_size": <receive_buffer_size>,
    "max_body_size": <max_body_size>,
    "base_path": "<base_path>",
    "resources": ["<resources>"],
    "features": ["<features>"],
    "binders": ["<binders>"],
    "backlog_size": <backlog_sze>
}
````

* `host` - The host or ip address to listen at for connections. `0.0.0.0` means listen at all available addresses.
Default is `0.0.0.0`
* `port` -  The port to listen at for connections. Default is `80`.
* `receive_buffer_size` - The int receive buffer size.  The value is optional.
* `max_body_size` - The int max body size allowed.  The default value is 1MB.
* `base_path` - The base path jersey responds to.  Default is `/`.
* `resources` - An array of package names to inspect for resources.
* `features` - An array of feature classes to inject.  For example: `"org.glassfish.jersey.jackson.JacksonFeature"`
* `binders` - An array of HK2 binder classes to configure injection bindings.
* `backlog_size` - An int that sets the http server backlog size.  The default value is 10,000

### Examples
#### Simple

```json
{
    "resources": ["com.englishtown.vertx.jersey.resources"]
}
```

#### All settings

```json
{
    "host": "localhost",
    "port": 8080,
    "base_path": "/rest",
    "resources": ["com.englishtown.vertx.jersey.resources", "com.englishtown.vertx.jersey.resources2"],
    "features": ["org.glassfish.jersey.jackson.JacksonFeature"],
    "binders": ["com.englishtown.vertx.jersey.AppBinder"]
}
```

## How to use

Due to the way vert.x module class loaders work, your best bet is to include vertx-mod-jersey in your mod.json.  Directly deploying the vertx-mod-jersey module will run into class loader problems when reading your JAX-RS resources.

```json
{
    "includes": "com.englishtown~vertx-mod-jersey~2.3.0-final"
}
```

The vertx-mod-jersey jar (plus its dependencies javax.ws.rs-api, javax.inject, jersey-server, etc.) should be added to your project with scope "provided".

You have 3 ways to start the Jersey Server:

1. In your mod.json file, make the start Verticle JerseyModule (`"main": "com.englishtown.vertx.jersey.JerseyModule"`).
2. In your own Verticle specified in mod.json `"main"`, create an instance of the JerseyServer and initialize similarly to how JerseyModule does.
3. Use vertx-mod-whenjersey, this uses When.java and simplifies the process.


Use #1 if you don't have anything else to do at application start.  Use #2 if you need to deploy other modules at start.

Note: if you are using vertx-mod-hk2, ensure you are using the same version as included in vertx-mod-jersey.
