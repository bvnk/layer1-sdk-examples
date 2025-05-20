# Java 8 Examples

## Requirements

These examples assume you have added the Java 8 SDK's to your build. For example:

```groovy
dependencies {
    implementation "com.layer1.clients:java8-digital:1.0.0"
}
```

Source available here at:
* https://github.com/bvnk/layer1-sdk-java8-digital
* https://github.com/bvnk/layer1-sdk-java8-trade
* https://github.com/bvnk/layer1-sdk-java-auth

## Setting up the client

You need to create a new API key set on the Layer1 console and provide the public key to be used for signature verification.

To generate a new key pair, you can use the following commands. The contents of public.pem would be provided to Layer1, and the private.pem would be used in your application to sign requests.

```bash
openssl genrsa -out private.pem 2048
openssl rsa -in private.pem -pubout -out public.pem
```

Once you have the client ID and keys, you can set up the client as follows, replacing the content of the signingPrivateKey with the contents of private.pem. You also need to use the URLs specific to your Layer1 environment (self-hosted, sandbox, etc)

```java
Layer1ClientConfiguration layer1ClientConfiguration = Layer1DefaultClientConfiguration.builder()
                .clientId("20ff4abc-8f1f-499f-8720-365a73e6f1f5")
                .clientSecret("xxx")
                .signingPrivateKey("-----BEGIN PRIVATE KEY-----\n" +
                        "xxx\n" +
                        "-----END PRIVATE KEY-----\n")
                .basePath("https://api.sandbox.layer1.com")
                .tokenUrl("https://auth.sandbox.layer1.com/auth/realms/bvnk/protocol/openid-connect/token")
                .build();

Layer1DigitalClient layer1DigitalClient = new Layer1DigitalClient(layer1ClientConfiguration);
```

## Example Requests

Asset pool IDs are largely constant depending on your implementation needs, but these are usually pre-configured and don't change between requests.

### Create a new address by network

```java
AddressApi addressApi = new AddressApi(layer1DigitalClient);
CreateAddressRequest createAddressRequest = new CreateAddressRequest();
createAddressRequest.setNetwork("ETHEREUM");
createAddressRequest.setAssetPoolId(UUID.fromString("b53b3740-e5bc-4e4c-addf-168858497741"));
createAddressRequest.setReference("my-user-id");
Address address = addressApi.createAddress(createAddressRequest).execute();
System.out.println(address);
```

### Create a new set of addresses by asset

```java
UUID assetPoolId = UUID.fromString("b53b3740-e5bc-4e4c-addf-168858497741");
String userId = "my-user-id";
AddressApi addressApi = new AddressApi(layer1DigitalClient);
CreateAddressRequest createAddressRequest = new CreateAddressRequest();
createAddressRequest.setAsset("USDT");
createAddressRequest.setAssetPoolId(assetPoolId);
createAddressRequest.setReference(userId);

addressApi.createAddress(createAddressRequest).execute(); // Triggers an async request to create new addresses for all supported chains

PaginatedResultAddress addresses = addressApi.listAddresses(UUID.fromString(assetPoolId), 0, 10).q(String.format("reference:%s", userId)).execute();

System.out.println(addresses);

```

### Send funds

```java
TransactionRequestApi transactionRequestApi = new TransactionRequestApi(layer1DigitalClient);

Participant destination = new Participant();
destination.setAmount(new BigDecimal("0.1"));
destination.setAddress("0x5DcA85d183A6D1D730992ED93D1A553803f9C661");

CreateTransactionRequest transactionRequest = new CreateTransactionRequest();
transactionRequest.setReference(UUID.randomUUID().toString()); // Replace with a unique reference for idempotency
transactionRequest.setAssetPoolId(UUID.fromString("b34a7a3e-bde5-4526-bf3b-b03c8e894fb0"));
transactionRequest.setNetwork("ETHEREUM");
transactionRequest.setAsset("ETH");
transactionRequest.setDestinations(Collections.singletonList(destination));

TransactionRequest result = transactionRequestApi.createTransaction(transactionRequest).execute();
System.out.println("Transaction Request ID: " + result.getRequestId());

```