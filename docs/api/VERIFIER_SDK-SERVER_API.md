---
puppeteer:
    pdf:
        format: A4
        displayHeaderFooter: true
        landscape: false
        scale: 0.8
        margin:
            top: 1.2cm
            right: 1cm
            bottom: 1cm
            left: 1cm
---

Verifier SDK Server API
==

- Subject: Verifier SDK Server API
- Author: OpenDID
- Date: 2026-10-20
- Version: v1.0.0

| Version | Date | Changes |
|---------|------|---------|
| v1.0.0 | 2026-10-20 | Initial release |


<div style="page-break-after: always;"></div>

# Table of Contents
- [1. APIs](#1-apis)
    - [1.1 VerifierService](#11-verifierservice)
        - [1.1.1 requestVpOffer](#111-requestvpoffer)
        - [1.1.2 requestStaticVpOffer](#112-requeststaticvpoffer)
        - [1.1.3 requestVerifyProfile](#113-requestverifyprofile)
        - [1.1.4 verifyPresentation](#114-verifypresentation)
        - [1.1.5 decryptVp](#115-decryptvp)
        - [1.1.6 confirmVerification (VP)](#116-confirmverification-vp)
        - [1.1.7 confirmVerification (VP or Proof)](#117-confirmverification-vp-or-proof)
        - [1.1.8 requestZkpProofRequestProfile](#118-requestzkpproofrequestprofile)
        - [1.1.9 verifyZkpProof](#119-verifyzkpproof)
        - [1.1.10 decryptZkpProof](#1110-decryptzkpproof)
- [2. Interfaces](#2-interfaces)
    - [2.1 VerificationConfigProvider](#21-verificationconfigprovider)
    - [2.2 VerifierInfoProvider](#22-verifierinfoprovider)
    - [2.3 EcdhSessionProvider](#23-ecdhsessionprovider)
    - [2.4 StorageProvider](#24-storageprovider)
    - [2.5 TransactionProvider](#25-transactionprovider)
    - [2.6 NonceGenerator](#26-noncegenerator)
    - [2.7 CryptoHelper](#27-cryptohelper)
- [3. Data Classes](#3-data-classes)
    - [3.1 VpOfferPayload](#31-vpofferpayload)
    - [3.2 VpVerificationRequest](#32-vpverificationrequest)
    - [3.3 VerificationConfirmResult](#33-verificationconfirmresult)
    - [3.4 ZkpVerificationResult](#34-zkpverificationresult)
    - [3.5 ReqE2e](#35-reqe2e)
    - [3.6 ZkpVerificationRequest](#36-zkpverificationrequest)

<div style="page-break-after: always;"></div>

## Overview

This document defines the Server SDK API for implementing the **VP (Verifiable Presentation) and ZKP (Zero-Knowledge Proof) verification** protocol in the OpenDID ecosystem.

### Key Features
- VP Offer generation for Dynamic/Static QR code flows
- VerifyProfile creation with E2E key exchange support
- VP verification pipeline: E2E decryption → signature verification → VC status check
- Claim extraction and verification result generation
- ZKP anonymous proof request and verification

<br>

<div style="page-break-after: always;"></div>

# 1. APIs

## 1.1 VerifierService

The `VerifierService` interface is the SDK facade. Obtain an instance via `VerifierSdkBuilder`. All seven providers are required; `build()` throws `VerifierSdkException` (`SDK_CONFIGURATION_ERROR`) if any of them is missing.

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

<br>

### 1.1.1 requestVpOffer

### Class Name
`VerifierService`

### Function Name
`requestVpOffer`

### Function Introduction
`Generates a VP Offer Payload for a Dynamic QR code flow. The returned payload includes an offerId (created by TransactionProvider.createTransactionId) used to correlate the holder's response, and validUntil set to the current time plus the policy's validityDuration (seconds).`

### Input Parameters

| Parameter | Type | Description | **M/O** | **Note** |
|-----------|------|-------------|---------|----------|
| policyId | String | Verification policy identifier | M | |
| device | String | Terminal/device identifier | M | |
| service | String | Service identifier | M | |
| locked | boolean | Whether the offer is locked | M | `true`: passcode required |

### Output Parameters

| Type | Description | **M/O** | **Note** |
|------|-------------|---------|----------|
| VpOfferPayload | VP Offer payload | M | Includes offerId. [Link](#31-vpofferpayload) |

### Function Declaration

```java
VpOfferPayload requestVpOffer(
    String policyId,
    String device,
    String service,
    boolean locked
)
```

### Function Usage
```java
VpOfferPayload offer = verifierService.requestVpOffer(
    "policy-login",
    "kiosk-01",
    "login-service",
    false
);
String offerId = offer.getOfferId(); // embed in QR code
```

<br>

### 1.1.2 requestStaticVpOffer

### Class Name
`VerifierService`

### Function Name
`requestStaticVpOffer`

### Function Introduction
`Generates a VP Offer Payload for a Static QR code flow. No offerId or validUntil is included; the same QR code is reused across multiple sessions.`

### Input Parameters

| Parameter | Type | Description | **M/O** | **Note** |
|-----------|------|-------------|---------|----------|
| policyId | String | Verification policy identifier | M | |
| device | String | Terminal/device identifier | M | |
| service | String | Service identifier | M | |
| locked | boolean | Whether the offer is locked | M | `true`: passcode required |

### Output Parameters

| Type | Description | **M/O** | **Note** |
|------|-------------|---------|----------|
| VpOfferPayload | VP Offer payload | M | offerId and validUntil are null. [Link](#31-vpofferpayload) |

### Function Declaration

```java
VpOfferPayload requestStaticVpOffer(
    String policyId,
    String device,
    String service,
    boolean locked
)
```

<br>

### 1.1.3 requestVerifyProfile

### Class Name
`VerifierService`

### Function Name
`requestVerifyProfile`

### Function Introduction
`Creates a VerifyProfile that describes the verification requirements for a given policy. The SDK generates a verifierNonce with NonceGenerator and puts it, together with reqE2e, in the profile's process. The calling application must sign the returned profile before sending it to the holder.`

### Input Parameters

| Parameter | Type | Description | **M/O** | **Note** |
|-----------|------|-------------|---------|----------|
| policyId | String | Verification policy identifier | M | |
| profileId | String | Unique profile identifier | M | UUID recommended |
| reqE2e | ReqE2e | Verifier's E2E key exchange information | M | e.g. from `EcdhSessionProvider.createSession(txId)`. [Link](#35-reqe2e) |

### Output Parameters

| Type | Description | **M/O** | **Note** |
|------|-------------|---------|----------|
| VerifyProfile | Verification profile without Proof | M | Application must add Proof |

### Function Declaration

```java
VerifyProfile requestVerifyProfile(
    String policyId,
    String profileId,
    ReqE2e reqE2e
)
```

### Function Usage
```java
// Verifier's E2E key exchange information (nonce, curve, publicKey, cipher, padding)
ReqE2e reqE2e = sessionProvider.createSession(txId);

VerifyProfile profile = verifierService.requestVerifyProfile(
    "policy-login",
    UUID.randomUUID().toString(),
    reqE2e
);
// Sign the profile and set its Proof (profile.setProof(...)) before returning to holder
```

<br>

### 1.1.4 verifyPresentation

### Class Name
`VerifierService`

### Function Name
`verifyPresentation`

### Function Introduction
`Performs the full VP verification pipeline: E2E decryption → auth type check → nonce check → VpManager verification (VP/VC signatures, schema, filter, validity period, holder-subject match) → VC status check. The VP is always decrypted with the E2E session of txId; plain (unencrypted) VP submission is not supported.`

### Input Parameters

| Parameter | Type | Description | **M/O** | **Note** |
|-----------|------|-------------|---------|----------|
| request | VpVerificationRequest | VP verification request | M | [Link](#32-vpverificationrequest) |

### Output Parameters

| Type | Description | **M/O** | **Note** |
|------|-------------|---------|----------|
| String | Verified VP JSON string | M | |

### Function Declaration

```java
String verifyPresentation(VpVerificationRequest request)
```

### Function Usage
```java
VpVerificationRequest request = VpVerificationRequest.builder()
    .txId(txId)
    .encVp(encVp)
    .iv(iv)
    .encHolderPublicKey(encHolderPublicKey)
    .verifierNonce(verifierNonce)
    .filter(policyFilter)
    .build();

String vpJson = verifierService.verifyPresentation(request);
```

<br>

### 1.1.5 decryptVp

### Class Name
`VerifierService`

### Function Name
`decryptVp`

### Function Introduction
`Decrypts an E2E-encrypted VP independently of the full verification flow. Use when decryption and verification are handled in separate steps.`

### Input Parameters

| Parameter | Type | Description | **M/O** | **Note** |
|-----------|------|-------------|---------|----------|
| txId | String | Transaction ID for E2E session lookup | M | |
| encHolderPublicKey | String | Holder's E2E public key | M | Multibase encoded (not encrypted) |
| encVp | String | E2E-encrypted VP | M | Multibase encoded |
| iv | String | Initial vector | M | Multibase encoded |

### Output Parameters

| Type | Description | **M/O** | **Note** |
|------|-------------|---------|----------|
| String | Decrypted VP JSON string | M | |

### Function Declaration

```java
String decryptVp(
    String txId,
    String encHolderPublicKey,
    String encVp,
    String iv
)
```

<br>

### 1.1.6 confirmVerification (VP)

### Class Name
`VerifierService`

### Function Name
`confirmVerification`

### Function Introduction
`Extracts the holder DID, submitted VCs and claims from a verified VP and returns the verification result (the SDK does not store it). When verified is false, a failure result is returned without parsing the VP. Use this overload for standard VP verification flows.`

### Input Parameters

| Parameter | Type | Description | **M/O** | **Note** |
|-----------|------|-------------|---------|----------|
| txId | String | Transaction ID | M | |
| vpJson | String | Verified VP JSON string | M | |
| verified | boolean | Whether verification succeeded | M | |

### Output Parameters

| Type | Description | **M/O** | **Note** |
|------|-------------|---------|----------|
| VerificationConfirmResult | Verification confirmation result | M | [Link](#33-verificationconfirmresult) |

### Function Declaration

```java
VerificationConfirmResult confirmVerification(
    String txId,
    String vpJson,
    boolean verified
)
```

### Function Usage
```java
VerificationConfirmResult result = verifierService.confirmVerification(txId, vpJson, true);

String holderDid = result.getHolderDid();
Map<String, Object> claims = result.getExtractedClaims();
```

<br>

### 1.1.7 confirmVerification (VP or Proof)

### Class Name
`VerifierService`

### Function Name
`confirmVerification`

### Function Introduction
`Returns the verification result for a verified VP or ZKP Proof according to the offer type (the SDK does not store it). For VerifyProofOffer, vpOrProofJson is not parsed and extractedClaims contains only {"zkpVerificationResult": "Successful"}. Use this overload when the offer type must be specified explicitly.`

### Input Parameters

| Parameter | Type | Description | **M/O** | **Note** |
|-----------|------|-------------|---------|----------|
| txId | String | Transaction ID | M | |
| vpOrProofJson | String | Verified VP or Proof JSON string | M | |
| verified | boolean | Whether verification succeeded | M | |
| offerType | OfferType | Offer type | M | `VerifyProofOffer`: ZKP Proof, any other value: VP |

### Output Parameters

| Type | Description | **M/O** | **Note** |
|------|-------------|---------|----------|
| VerificationConfirmResult | Verification confirmation result | M | [Link](#33-verificationconfirmresult) |

### Function Declaration

```java
VerificationConfirmResult confirmVerification(
    String txId,
    String vpOrProofJson,
    boolean verified,
    OfferType offerType
)
```

<br>

### 1.1.8 requestZkpProofRequestProfile

### Class Name
`VerifierService`

### Function Name
`requestZkpProofRequestProfile`

### Function Introduction
`Creates a ProofRequestProfile defining the ZKP verification request. The SDK generates the ProofRequest nonce and an E2E key pair, and saves the E2E session with EcdhSessionProvider.saveSession. The returned profile does not include a Proof; the calling application must sign it and set the Proof before sending it to the holder.`

### Input Parameters

| Parameter | Type | Description | **M/O** | **Note** |
|-----------|------|-------------|---------|----------|
| request | ProofRequestProfileRequest | Profile request metadata | M | Uses `txId`, `profileId` and `reqE2e` (`curve`, `cipher`, `padding`) |
| policy | ZkpPolicy | ZKP policy configuration | M | Uses `title`, `description`, `language`, `name`, `version`, `requestedAttributes`, `requestedPredicates` |

### Output Parameters

| Type | Description | **M/O** | **Note** |
|------|-------------|---------|----------|
| ProofRequestProfile | ZKP proof request profile | M | Proof not included. Store `getProfile().getProofRequest()` for [verifyZkpProof](#119-verifyzkpproof) |

### Function Declaration

```java
ProofRequestProfile requestZkpProofRequestProfile(
    ProofRequestProfileRequest request,
    ZkpPolicy policy
)
```

<br>

### 1.1.9 verifyZkpProof

### Class Name
`VerifierService`

### Function Name
`verifyZkpProof`

### Function Introduction
`Decrypts a submitted ZKP Proof, checks the submitted nonce against the stored proofRequest, and verifies the Proof with the ZKP library using the stored nonce.`

- `request.proofRequest` must be the ProofRequest that the verifier stored from the ProofRequestProfile created in [1.1.8](#118-requestzkpproofrequestprofile), not a value supplied by the holder.
- `request.nonce` must equal the nonce of `proofRequest`. If either nonce is missing, `request.nonce` is not a number, or the values differ, `VerifierSdkException` (`SDK_INVALID_NONCE`) is thrown. Verification uses the stored nonce.
- A Proof with no identifiers is rejected with `VerifierSdkException` (`SDK_INVALID_PROOF`).
- If the ZKP library reports the Proof as invalid, a result with `verified=false` and `errorMessage` is returned instead of an exception.
- The Credential Schema and Credential Definition of each identifier are fetched with `StorageProvider.getZKPCredential` / `getZKPCredentialDefinition`.
- Decryption errors are thrown as `SDK_E2E_SESSION_NOT_FOUND` / `SDK_DECRYPTION_FAILED`, and other unexpected errors as `SDK_SIGNATURE_VERIFICATION_FAILED`.

### Input Parameters

| Parameter | Type | Description | **M/O** | **Note** |
|-----------|------|-------------|---------|----------|
| request | ZkpVerificationRequest | ZKP verification request | M | [Link](#36-zkpverificationrequest) |

### Output Parameters

| Type | Description | **M/O** | **Note** |
|------|-------------|---------|----------|
| ZkpVerificationResult | ZKP verification result | M | [Link](#34-zkpverificationresult) |

### Function Declaration

```java
ZkpVerificationResult verifyZkpProof(ZkpVerificationRequest request)
```

### Function Usage
```java
ZkpVerificationRequest request = ZkpVerificationRequest.builder()
    .txId(txId)
    .encProof(encProof)
    .iv(iv)
    .accE2e(accE2e)
    .nonce(submittedNonce)             // nonce submitted by the holder
    .proofRequest(storedProofRequest)  // ProofRequest stored by the verifier
    .build();

ZkpVerificationResult result = verifierService.verifyZkpProof(request);
if (!result.isVerified()) {
    String reason = result.getErrorMessage();
}
```

<br>

### 1.1.10 decryptZkpProof

### Class Name
`VerifierService`

### Function Name
`decryptZkpProof`

### Function Introduction
`Decrypts an E2E-encrypted ZKP Proof independently of the full verification flow.`

### Input Parameters

| Parameter | Type | Description | **M/O** | **Note** |
|-----------|------|-------------|---------|----------|
| txId | String | Transaction ID for E2E session lookup | M | |
| encProof | String | Encrypted Proof | M | Multibase encoded |
| iv | String | Initial vector | M | Multibase encoded |
| accE2e | ZkpVerificationRequest.AccE2e | Holder's accept E2E information | M | `accE2e.publicKey` is used for decryption. [Link](#36-zkpverificationrequest) |

### Output Parameters

| Type | Description | **M/O** | **Note** |
|------|-------------|---------|----------|
| Proof | Decrypted Proof object | M | `org.omnione.did.zkp.datamodel.proof.Proof` |

### Function Declaration

```java
Proof decryptZkpProof(
    String txId,
    String encProof,
    String iv,
    ZkpVerificationRequest.AccE2e accE2e
)
```

<br>

# 2. Interfaces

## 2.1 VerificationConfigProvider

### Declaration

```java
public interface VerificationConfigProvider {
    VerificationPolicy getPolicy(String policyId);
    boolean existsPolicy(String policyId);
}
```

### Description
`Supplies verification policies by policy ID. Implement this interface to load policies from a database, configuration file, or remote service.`

### Usage
```java
@Component
public class DbVerificationConfigProvider implements VerificationConfigProvider {

    @Override
    public VerificationPolicy getPolicy(String policyId) {
        return policyRepository.findById(policyId)
            .map(this::toVerificationPolicy)
            .orElseThrow(() -> new VerifierSdkException(SDK_POLICY_NOT_FOUND));
    }

    @Override
    public boolean existsPolicy(String policyId) {
        return policyRepository.existsById(policyId);
    }
}
```

<br>

## 2.2 VerifierInfoProvider

### Declaration

```java
public interface VerifierInfoProvider {
    ProviderDetail getVerifierInfo();
    String getVerifierDidDocument();
    String getVerifierDid();
}
```

### Description
`Provides verifier information: the verifier's ProviderDetail (used as the verifier of VerifyProfile and ProofRequestProfile), its DID Document (JSON string) and its DID. Signing keys are not handled by this interface.`

<br>

## 2.3 EcdhSessionProvider

### Declaration

```java
public interface EcdhSessionProvider {
    ReqE2e createSession(String txId);
    void saveSession(String txId, KeyPairInfo keyPair, ReqE2e reqE2e);
    ReqE2e getSession(String txId);
    void removeSession(String txId);
    boolean existsSession(String txId);
    String decrypt(String txId, String encHolderPublicKey, String encVp, String iv);
}
```

### Description
`Creates and manages E2E encryption sessions (ephemeral ECDH key pairs) per transaction and decrypts E2E-encrypted data. decrypt performs ECDH with the holder's public key, derives the session key and decrypts the data; it is used by decryptVp, verifyPresentation, decryptZkpProof and verifyZkpProof. The SDK does not store sessions and provides no default implementation; implement it with in-memory, Redis or DB storage.`

<br>

## 2.4 StorageProvider

### Declaration

```java
import org.omnione.did.zkp.datamodel.definition.CredentialDefinition;
import org.omnione.did.zkp.datamodel.schema.CredentialSchema;

public interface StorageProvider {
    String findDidDocument(String did);
    String getVcMeta(String vcId);
    boolean existsDidDocument(String did);
    CredentialSchema getZKPCredential(String schemaId);
    CredentialDefinition getZKPCredentialDefinition(String credDefId);
}
```

### Description
`Provides the data needed for protocol verification: DID Documents (JSON string) of holders and issuers, VC metadata (JSON string) and ZKP Credential Schemas/Definitions. Implement it against a blockchain, database or other storage. Policies and filters are provided by VerificationConfigProvider, not by this interface.`

- If `findDidDocument` returns `null`, VP verification fails with `SDK_DID_DOCUMENT_NOT_FOUND`.
- The VC status is read from the `status` (or `vcStatus`, `credentialStatus.status`) field of the `getVcMeta` result; `REVOKED`, `INACTIVE` and `EXPIRED` are rejected with `SDK_INVALID_VC`. If `getVcMeta` returns `null`, the status check is skipped for that VC.

<br>

## 2.5 TransactionProvider

### Declaration

```java
public interface TransactionProvider {
    String createTransactionId();
    Long getTransactionId(String txId);
}
```

### Description
`Creates and looks up transaction IDs. createTransactionId returns a new transaction ID (UUID recommended), which the SDK uses as the offerId of a Dynamic VP Offer. getTransactionId returns the transaction's database primary key (Long) for a transaction ID. The SDK does not store transactions; the application manages them.`

<br>

## 2.6 NonceGenerator

### Declaration

```java
public interface NonceGenerator {
    String generateNonce(int length);

    default String generateNonce() {
        return generateNonce(16);
    }
}
```

### Description
`Generates cryptographically secure, Base64-encoded nonces of the given byte length. generateNonce() (16 bytes) creates the verifierNonce embedded in VerifyProfile, which is validated during VP submission.`

### Usage
```java
NonceGenerator nonceGenerator = length -> {
    byte[] bytes = new byte[length];
    new SecureRandom().nextBytes(bytes);
    return Base64.getUrlEncoder().withoutPadding().encodeToString(bytes);
};
```

<br>

## 2.7 CryptoHelper

### Declaration

```java
public interface CryptoHelper {
    boolean verifySignature(String publicKey, String signature, byte[] data);
    String sha256(byte[] data);
    byte[] decodeMultibase(String multibase);
    String encodeBase64(byte[] data);
    byte[] decodeBase64(String base64);
    KeyPairInfo generateKeyPair(String curve);
    String generateNonce(int length);
    String encodeMultibase(byte[] data);
    byte[] generateSharedSecret(byte[] holderPublicKey, byte[] verifierPrivateKey, String curve);
    byte[] deriveSessionKey(byte[] sharedSecret, byte[] nonce, String cipherType);
    byte[] decrypt(byte[] encData, byte[] sessionKey, byte[] iv, String cipherType, String paddingType);
    KeyPairInfo generateEcKeyPair(String curve);
}
```

### Description
`Provides cryptographic utilities: signature verification, SHA-256 hashing, Base64/Multibase encoding, E2E key pair and nonce generation, ECDH shared secret generation, session key derivation (KDF) and symmetric decryption. Within the SDK, generateEcKeyPair and generateNonce are used when creating a ProofRequestProfile; the other methods are available to the application (e.g. its EcdhSessionProvider implementation). Implement it with a crypto library such as did-crypto-sdk-server.`

<br>

# 3. Data Classes

## 3.1 VpOfferPayload

### Declaration

```java
@Getter
@Builder
public class VpOfferPayload {
    private String offerId;
    private OfferType type;
    private PresentMode mode;
    private String device;
    private String service;
    private List<String> endpoints;
    private String validUntil;
    @Builder.Default
    private Boolean locked = false;
}
```

### Property

| Parameter | Type | Description | **M/O** | **Note** |
|-----------|------|-------------|---------|----------|
| offerId | String | Offer identifier | O | Null for static QR |
| type | OfferType | Offer type | M | `VerifyOffer` \| `VerifyProofOffer`. The SDK offer methods set `VerifyOffer` |
| mode | PresentMode | Presentation mode | M | `DIRECT` \| `INDIRECT` \| `PROXY` (display names `Direct`, `Indirect`, `Proxy`) |
| device | String | Device identifier | M | |
| service | String | Service identifier | M | |
| endpoints | List\<String\> | Verifier API endpoints | O | Taken from the policy. Required for Direct mode |
| validUntil | String | Offer expiration time | O | ISO-8601. Null for static QR |
| locked | Boolean | Whether the offer is locked | O | Default: false. `true`: passcode required |

<br>

## 3.2 VpVerificationRequest

### Declaration

```java
@Getter
@Builder
public class VpVerificationRequest {
    private String vp;
    @Builder.Default
    private Boolean encrypted = false;
    private String encVp;
    private String iv;
    private String encHolderPublicKey;
    private String txId;
    private String serverToken;
    private String verifierNonce;
    private Integer requiredAuthType;
    private Filter filter;
}
```

### Property

| Parameter | Type | Description | **M/O** | **Note** |
|-----------|------|-------------|---------|----------|
| txId | String | Transaction identifier | M | Used to look up the E2E session |
| vp | String | Plain VP JSON | O | Not used by `verifyPresentation` |
| encrypted | Boolean | E2E encryption flag | O | Default: false. Not used by `verifyPresentation` (the VP is always decrypted) |
| encVp | String | Encrypted VP | M | Multibase encoded |
| iv | String | Initial vector | M | Multibase encoded |
| encHolderPublicKey | String | Holder's E2E public key | M | Multibase encoded (not encrypted) |
| verifierNonce | String | Nonce from VerifyProfile | M | Must equal the VP's `verifierNonce` (otherwise `SDK_INVALID_NONCE`) |
| serverToken | String | Authentication token | O | Not used by the SDK |
| requiredAuthType | Integer | Required auth type bitmask | O | `null` or 0x00000000: no restriction. The SDK checks only 0x00000006 (PIN or BIO); other values are not checked by the SDK |
| filter | Filter | Claim filter from policy | O | Applied in VpManager verification when set |

<br>

## 3.3 VerificationConfirmResult

### Declaration

```java
@Getter
@Builder
public class VerificationConfirmResult {
    private String txId;
    private Boolean verified;
    private String holderDid;
    private Map<String, Object> submittedVcs;
    private Map<String, Object> extractedClaims;
    private Instant verifiedAt;
    private String errorMessage;
    private String errorCode;
}
```

### Property

| Parameter | Type | Description | **M/O** | **Note** |
|-----------|------|-------------|---------|----------|
| txId | String | Transaction identifier | M | |
| verified | Boolean | Verification success flag | M | |
| holderDid | String | Holder's DID | O | Not set for `VerifyProofOffer` |
| submittedVcs | Map\<String, Object\> | Submitted VCs (VC ID → VC object) | O | Not set for `VerifyProofOffer` |
| extractedClaims | Map\<String, Object\> | Extracted claims (code → value) | O | For `VerifyProofOffer`: `{"zkpVerificationResult": "Successful"}` |
| verifiedAt | Instant | Verification timestamp | M | |
| errorMessage | String | Failure reason | O | Present when `verified=false` |
| errorCode | String | Error code | O | Not set by the SDK |

<br>

## 3.4 ZkpVerificationResult

### Declaration

```java
@Getter
@Builder
public class ZkpVerificationResult {
    private String txId;
    private boolean verified;
    private String errorMessage;
    private Map<String, String> revealedAttributes;
    private List<VerifiedPredicate> verifiedPredicates;
    private String holderDid;
    private String proofJson;
    private long verifiedAt;

    @Getter
    @Builder
    public static class VerifiedPredicate {
        private String type;
        private String name;
        private String value;
        private boolean verified;
    }
}
```

### Property

| Parameter | Type | Description | **M/O** | **Note** |
|-----------|------|-------------|---------|----------|
| txId | String | Transaction identifier | M | |
| verified | boolean | ZKP proof validity | M | `false` when the ZKP library reports the Proof as invalid |
| errorMessage | String | Failure reason | O | Present when `verified=false` |
| revealedAttributes | Map\<String, String\> | Disclosed attribute values | O | Set when `verified=true`; the current implementation returns an empty map |
| verifiedPredicates | List\<VerifiedPredicate\> | Verified predicates | O | Not set by the current implementation |
| holderDid | String | Holder's DID | O | Not set by the current implementation |
| proofJson | String | Verified Proof JSON | O | Present when `verified=true` |
| verifiedAt | long | Verification time | M | Unix timestamp (milliseconds) |

**VerifiedPredicate**

| Parameter | Type | Description | **M/O** | **Note** |
|-----------|------|-------------|---------|----------|
| type | String | Predicate type | M | `>=`, `<=`, `>`, `<` |
| name | String | Attribute name | M | |
| value | String | Comparison value | M | |
| verified | boolean | Whether the predicate was verified | M | |

<br>

## 3.5 ReqE2e

### Declaration

```java
@Getter
@Builder
public class ReqE2e {
    private String nonce;
    private String curve;
    private String publicKey;
    private String cipher;
    private String padding;
}
```

### Property

| Parameter | Type | Description | **M/O** | **Note** |
|-----------|------|-------------|---------|----------|
| nonce | String | Nonce for E2E session key derivation | M | 16 bytes |
| curve | String | Key curve | M | `Secp256r1` or `Secp256k1` |
| publicKey | String | Verifier's ephemeral E2E public key | M | Multibase encoded |
| cipher | String | Cipher algorithm | M | e.g. `AES-256-CBC` |
| padding | String | Padding scheme | M | e.g. `PKCS5` |

When used in `ProofRequestProfileRequest`, only `curve`, `cipher` and `padding` are read; the SDK generates `publicKey` and `nonce`.

<br>

## 3.6 ZkpVerificationRequest

### Declaration

```java
@Getter
@Builder
public class ZkpVerificationRequest {
    private String txId;
    private String encProof;
    private String iv;
    private AccE2e accE2e;
    private String nonce;
    private Object proofRequest;  // org.omnione.did.zkp.datamodel.proofrequest.ProofRequest
    private List<ProofVerifyParam> proofVerifyParams;

    @Getter
    @Builder
    public static class AccE2e {
        private String publicKey;
        private String iv;
        private Object proof;
    }
}
```

### Property

| Parameter | Type | Description | **M/O** | **Note** |
|-----------|------|-------------|---------|----------|
| txId | String | Transaction identifier | M | Used to look up the E2E session |
| encProof | String | E2E-encrypted ZKP Proof | M | Multibase encoded |
| iv | String | Initial vector | M | Multibase encoded. Used for decryption |
| accE2e | AccE2e | Holder's accept E2E information | M | |
| nonce | String | Proof nonce submitted by the holder | M | Decimal number string. Must equal the nonce of `proofRequest` |
| proofRequest | Object | ProofRequest stored by the verifier | M | `org.omnione.did.zkp.datamodel.proofrequest.ProofRequest`. Must be the value stored from the ProofRequestProfile, not a holder-supplied value |
| proofVerifyParams | List\<ProofVerifyParam\> | Proof verification parameters | O | Not used by `verifyZkpProof`; Schemas and Definitions are fetched through `StorageProvider` |

**AccE2e**

| Parameter | Type | Description | **M/O** | **Note** |
|-----------|------|-------------|---------|----------|
| publicKey | String | Holder's E2E public key | M | Multibase encoded. Used for decryption |
| iv | String | Initial vector | O | Not used by the SDK |
| proof | Object | Holder's signature over AccE2e | O | Not verified by the SDK |
