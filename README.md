# Java Client for the Homematic IP Cloud

A Java wrapper for the RESP API of the Homematic IP Cloud.
Since there is no offical documentation i used the code of the [Python wrapper](https://github.com/coreGreenberet/homematicip-rest-api)
to get an idea of how the API works. Thanks to [coreGreenberet](https://github.com/coreGreenberet) for doing the great job of
reverse engineering. **Use this library at your own risk!**

# Loading the configuration and getting the current state
To load the configuration for the client from an `application.yml` you can use the class
`org.salex.hmip.client.HmIPProperties`. Include it into the configuration properties scan
and provide the following properties in your `application.yml`:

```yml
org:
  salex:
    hmip:
      client:
        access-point-sgtin: '{cipher}<some-encrypted-value>'
        pin: '{cipher}<some-encrypted-value>'
        device-id: '{cipher}<some-encrypted-value>'
        client-id: '{cipher}<some-encrypted-value>'
        client-name: '<some-plan-value>'
        client-auth-token: '{cipher}<some-encrypted-value>'
        auth-token: '{cipher}<some-encrypted-value>'
```
**You should not store any tokens or other secrets as plain text in the configuration!**

I use the [property encryption feature of Spring Cloud Config](https://cloud.spring.io/spring-cloud-static/spring-cloud-config/1.3.0.RELEASE/#_encryption_and_decryption) to decrypt the
secret configuration values, and I use the [Spring Cloud CLI for manual encryption](https://cloud.spring.io/spring-cloud-cli/reference/html/#_encryption_and_decryption)
of the values. You can provide the encryption key by setting the environment variable `ENCRYPT_KEY`.

With the loaded (and decrypted) properties you can easily create and use the client:

```java
final var properties = ... // Should be injected
HmIPConfiguration.builder()
    .properties(properties)
    .build()
    .map(config -> new HmIPClient(config))
    .flatMap(client -> client.loadCurrentState())
    .doOnError(error -> {
        LOG.error(String.format("Failed to load the current state: %s", error.getMessage()));
        ...
    })
    .subscribe(currentState -> {
        LOG.info("Successfully loaded the current state");
        ...
    });
```

# Registering a new client
To register a new client in the Homematic IP Cloud you can use the client library:
```java
final var clientName = ...
final var accessPointSGTIN = ...        
HmIPConfiguration.builder()
    .clientName(clientName)
    .accessPointSGTIN(accessPointSGTIN)
    .build()
    .flatMap(config -> config.registerClient())
    .doOnError(error -> {
        LOG.error(String.format("Failed to register new client: %s", error.getMessage()));
        ...
    }).subscribe(config -> {
        LOG.info("Successfully registered new client");
        LOG.info(String.format("Device ID: %s", config.getDeviceId()));
        LOG.info(String.format("Client ID: %s", config.getClientId()));
        LOG.info(String.format("Client Auth Token: %s", config.getClientAuthToken()));
        LOG.info(String.format("Auth Token: %s", config.getAuthToken()));
        ...
        // Encrypt these values and put them to the configuration of your client app for lates usage
    });
```
All you need is the SGTIN (Serialized Global Trade Item Number) of your access point, the PIN (if you have assigned one)
and a name for the client. You also have to acknowledge the client registration by pressing the "blue button" on your
access point, when you are prompted in the log.

# Work in Progress
The development of the library is 'work in progress', so actually only a few features are available. 