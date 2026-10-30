
# Server Verifier SDK Guide

This document is a guide for building and using the Server Verifier SDK. The Server Verifier SDK provides the VP (Verifiable Presentation) and ZKP (Zero-Knowledge Proof) verification protocols of the OpenDID Verifier. The application server supplies data and storage through provider interfaces, and the SDK handles the protocol logic.

## Software Specifications
| Category      | Description                   |
|---------------|-------------------------------|
| OS            | Linux, macOS, Windows         |
| Language      | Java 21                       |
| Build System  | Gradle 9.2.1 (Gradle Wrapper) |

<br>

## Build Instructions
Generate a JAR file using the Gradle Wrapper of this SDK project.

1. Run the following command in the `source/did-verifier-sdk-server` folder.
```bash
./gradlew jar
```
2. Once the execution is complete, the file `did-verifier-sdk-server-1.0.0.jar` will be generated in the `build/libs` folder.

<br>

## How to Apply SDK
1. Copy `did-verifier-sdk-server-1.0.0.jar` and the OpenDID SDK libraries in the [libs](libs) folder to the `libs` folder of the app project.
2. Add the following dependencies to the `build.gradle` file of the app project.

```groovy
    implementation files('libs/did-verifier-sdk-server-1.0.0.jar')
    implementation files('libs/did-core-sdk-server-2.0.0.jar')
    implementation files('libs/did-crypto-sdk-server-2.0.0.jar')
    implementation files('libs/did-datamodel-sdk-server-2.0.0.jar')
    implementation files('libs/did-zkp-sdk-server-2.0.1.jar')
    implementation files('libs/did-common-sdk-server-2.0.0.jar')
    implementation 'com.google.code.gson:gson:2.10.1'
    implementation 'org.slf4j:slf4j-api:2.0.12'
```

3. Implement the provider interfaces in the `org.omnione.did.verifier.v1.provider` package.

| Interface                  | Role                                                            |
|----------------------------|-----------------------------------------------------------------|
| VerificationConfigProvider | Provides verification policies                                  |
| VerifierInfoProvider       | Provides the verifier's information, DID and DID document       |
| EcdhSessionProvider        | Creates, stores and removes E2E sessions and decrypts with them |
| StorageProvider            | Looks up DID documents, VC metadata and ZKP schemas/definitions |
| TransactionProvider        | Creates transaction IDs and looks up transactions               |
| NonceGenerator             | Generates nonces                                                |
| CryptoHelper               | Signature verification, multibase, key generation, ECDH and decryption |

4. Create a `VerifierService` with `VerifierSdkBuilder`. `build()` throws a `VerifierSdkException` if any provider is missing.

```java
VerifierService verifierService = new VerifierSdkBuilder()
    .configProvider(configProvider)
    .verifierInfoProvider(verifierInfoProvider)
    .sessionProvider(sessionProvider)
    .storageProvider(storageProvider)
    .transactionProvider(transactionProvider)
    .nonceGenerator(nonceGenerator)
    .cryptoHelper(cryptoHelper)
    .build();
```

For the full API, see the [Verifier SDK Server API](../../docs/api/VERIFIER_SDK-SERVER_API.md) document.
