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
- Date: 2026-11-09
- Version: v3.0.0

| Version | Date | Changes |
|---------|------|---------|
| v1.0.0 | 2026-10-20 | Initial release |
| v3.0.0 | 2026-11-09 | Documented all public APIs and corrected signatures |


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
        - [1.1.11 getOfferService](#1111-getofferservice)
        - [1.1.12 getProfileService](#1112-getprofileservice)
        - [1.1.13 getVerificationService](#1113-getverificationservice)
        - [1.1.14 getConfirmService](#1114-getconfirmservice)
        - [1.1.15 getZkpProofVerificationProtocol](#1115-getzkpproofverificationprotocol)
    - [1.2 VerifierSdkBuilder](#12-verifiersdkbuilder)
        - [1.2.1 build](#121-build)
    - [1.3 VpOfferProtocol](#13-vpofferprotocol)
        - [1.3.1 requestVpOffer](#131-requestvpoffer)
        - [1.3.2 requestStaticVpOffer](#132-requeststaticvpoffer)
    - [1.4 VpProfileProtocol](#14-vpprofileprotocol)
        - [1.4.1 requestVerifyProfile](#141-requestverifyprofile)
    - [1.5 VpVerificationProtocol](#15-vpverificationprotocol)
        - [1.5.1 verifyPresentation](#151-verifypresentation)
        - [1.5.2 decryptVp](#152-decryptvp)
        - [1.5.3 validateAuthType](#153-validateauthtype)
        - [1.5.4 validateNonce](#154-validatenonce)
        - [1.5.5 validateVcStatuses](#155-validatevcstatuses)
    - [1.6 VerificationConfirmProtocol](#16-verificationconfirmprotocol)
        - [1.6.1 confirmVerification (VP)](#161-confirmverification-vp)
        - [1.6.2 confirmVerification (VP or Proof)](#162-confirmverification-vp-or-proof)
        - [1.6.3 extractHolderDid](#163-extractholderdid)
        - [1.6.4 extractSubmittedVcs](#164-extractsubmittedvcs)
        - [1.6.5 extractSubmittedVcsAsMap](#165-extractsubmittedvcsasmap)
        - [1.6.6 extractClaims](#166-extractclaims)
        - [1.6.7 extractFilteredClaims](#167-extractfilteredclaims)
    - [1.7 ZkpProofVerificationProtocol](#17-zkpproofverificationprotocol)
        - [1.7.1 requestZkpProofRequestProfile](#171-requestzkpproofrequestprofile)
        - [1.7.2 verifyZkpProof](#172-verifyzkpproof)
        - [1.7.3 decryptZkpProof](#173-decryptzkpproof)
    - [1.8 Implementation Classes](#18-implementation-classes)
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
    - [3.7 VerificationPolicy](#37-verificationpolicy)
    - [3.8 ProofRequestProfile](#38-proofrequestprofile)
    - [3.9 ZkpInnerProfile](#39-zkpinnerprofile)
    - [3.10 ProofRequestProfileRequest](#310-proofrequestprofilerequest)
    - [3.11 ZkpPolicy](#311-zkppolicy)
    - [3.12 KeyPairInfo](#312-keypairinfo)
    - [3.13 ProofVerifyParam](#313-proofverifyparam)
    - [3.14 OfferType](#314-offertype)
    - [3.15 PresentMode](#315-presentmode)
    - [3.16 VerifierSdkException](#316-verifiersdkexception)
    - [3.17 VerifierSdkErrorCode](#317-verifiersdkerrorcode)

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

The `VerifierService` interface is the SDK facade. Obtain an instance via `VerifierSdkBuilder`. All seven providers are required; `build()` throws `VerifierSdkException` (`SDK_CONFIGURATION_ERROR`) if any of them is missing. The facade delegates each call to one of five protocol interfaces, which are returned by [1.1.11](#1111-getofferservice)–[1.1.15](#1115-getzkpproofverificationprotocol). The SDK reports errors with `VerifierSdkException` ([3.16](#316-verifiersdkexception)); how exceptions thrown by provider implementations are handled is described for each method and in [2. Interfaces](#2-interfaces).

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

- `VerificationConfigProvider.getPolicy(policyId)` is called first. If it returns `null`, `VerifierSdkException` (`SDK_POLICY_NOT_FOUND`) is thrown; exceptions thrown by the provider are passed through unchanged.
- `TransactionProvider.createTransactionId()` is then called once, and its return value is used as `offerId` without a `null` check.
- `mode` is converted from the policy's `mode` with `PresentMode.fromDisplayName`. A value other than `Direct`, `Indirect` or `Proxy` throws `IllegalArgumentException`.
- `endpoints` is copied from the policy and `type` is always `VerifyOffer`. `validUntil` is an ISO-8601 UTC string (`Instant.toString()`).
- The SDK does not store the offer. The application must keep the `offerId` and the transaction state.

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

- The policy lookup, `mode`, `endpoints` and `type` are handled as in [1.1.1](#111-requestvpoffer). `TransactionProvider` is not called.

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

### Function Usage
```java
VpOfferPayload staticOffer = verifierService.requestStaticVpOffer(
    "policy-login",
    "kiosk-01",
    "login-service",
    false
);
```

<br>

### 1.1.3 requestVerifyProfile

### Class Name
`VerifierService`

### Function Name
`requestVerifyProfile`

### Function Introduction
`Creates a VerifyProfile that describes the verification requirements for a given policy. The SDK generates a verifierNonce with NonceGenerator and puts it, together with reqE2e, in the profile's process. The calling application must sign the returned profile before sending it to the holder.`

- If `VerificationConfigProvider.getPolicy(policyId)` returns `null`, `VerifierSdkException` (`SDK_POLICY_NOT_FOUND`) is thrown. Exceptions thrown by `VerificationConfigProvider`, `VerifierInfoProvider` and `NonceGenerator` are passed through unchanged.
- The profile is filled as follows: `type` = `VerifyProfile`, `title` = policy `policyName`, `description` = policy `description` (empty string if `null`), `encoding` = `UTF-8`, `language` = policy `language`, `profile.verifier` = `VerifierInfoProvider.getVerifierInfo()`, `profile.filter` = policy `filter`, `profile.process` = `endpoints` and `authType` of the policy `process` plus `reqE2e` and `verifierNonce`.
- `reqE2e` is copied field by field into the profile and must not be `null` (`NullPointerException`). The SDK does not create or store the E2E session in this flow; the application must keep the private key for `txId` (for example with `EcdhSessionProvider.createSession`) so that [verifyPresentation](#114-verifypresentation) can decrypt the VP.
- Store `profile.getProfile().getProcess().getVerifierNonce()` and pass it as `VpVerificationRequest.verifierNonce` when verifying the VP.

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
String verifierNonce = profile.getProfile().getProcess().getVerifierNonce(); // keep for verifyPresentation
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

- Decryption: as in [decryptVp](#115-decryptvp) (`SDK_E2E_SESSION_NOT_FOUND`, `SDK_DECRYPTION_FAILED`).
- Auth type: only when `requiredAuthType` is `0x00000006` (PIN or BIO), the fragment after `#` of the VP proof's `verificationMethod` must be `pin` or `bio` (case-insensitive). A missing proof or `verificationMethod` throws `SDK_INVALID_PROOF`; any other fragment throws `SDK_INVALID_AUTH_TYPE`. See [1.5.3](#153-validateauthtype).
- Nonce: if the VP has no `verifierNonce` or it differs from `request.verifierNonce`, `SDK_INVALID_NONCE` is thrown.
- VpManager: `StorageProvider.findDidDocument` is called for the holder DID and for the issuer DID of each VC; `null` throws `SDK_DID_DOCUMENT_NOT_FOUND`. A VP without VCs throws `SDK_INVALID_VP`, and a VpManager verification error throws `SDK_SIGNATURE_VERIFICATION_FAILED`.
- VC status: checked with `StorageProvider.getVcMeta` as described in [1.5.5](#155-validatevcstatuses); a revoked, inactive or expired VC throws `SDK_INVALID_VC`.
- Exceptions thrown by `StorageProvider` are passed through unchanged. On success the decrypted VP JSON is returned as is; the SDK neither removes the E2E session nor stores the result.

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

- If `EcdhSessionProvider.existsSession(txId)` returns `false`, `VerifierSdkException` (`SDK_E2E_SESSION_NOT_FOUND`) is thrown.
- Otherwise `EcdhSessionProvider.decrypt(txId, encHolderPublicKey, encVp, iv)` is called and its return value is returned unchanged. Any exception thrown by `decrypt` is converted to `SDK_DECRYPTION_FAILED`.
- The VP is neither parsed nor verified.

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

### Function Usage
```java
String vpJson = verifierService.decryptVp(txId, encHolderPublicKey, encVp, iv);
```

<br>

### 1.1.6 confirmVerification (VP)

### Class Name
`VerifierService`

### Function Name
`confirmVerification`

### Function Introduction
`Extracts the holder DID, submitted VCs and claims from a verified VP and returns the verification result (the SDK does not store it). When verified is false, a failure result is returned without parsing the VP. Use this overload for standard VP verification flows.`

- Equivalent to [1.1.7](#117-confirmverification-vp-or-proof) with `offerType` = `VerifyOffer`.
- `vpJson` is parsed with Gson; parsing errors are not converted to `VerifierSdkException`. If the VP has no `holder`, `SDK_INVALID_VP` is thrown.
- `submittedVcs` contains each VC of `verifiableCredential` that has an `id` ([1.6.5](#165-extractsubmittedvcsasmap)), and `extractedClaims` contains `credentialSubject.claims[].code` → `value` of all VCs ([1.6.6](#166-extractclaims)).
- When `verified` is `false`, the result has `verified=false` and `errorMessage` = `Verification failed`.

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

- When `verified` is `false`, a failure result (`verified=false`, `errorMessage` = `Verification failed`) is returned for any `offerType`.
- `VerifyProofOffer`: `vpOrProofJson` is not used. The result has `verified=true` and no `holderDid` or `submittedVcs`.
- Any other `offerType`, including `null`: processed as a VP as in [1.1.6](#116-confirmverification-vp).

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

### Function Usage
```java
VerificationConfirmResult result = verifierService.confirmVerification(
    txId,
    proofJson,
    true,
    OfferType.VerifyProofOffer
);
```

<br>

### 1.1.8 requestZkpProofRequestProfile

### Class Name
`VerifierService`

### Function Name
`requestZkpProofRequestProfile`

### Function Introduction
`Creates a ProofRequestProfile defining the ZKP verification request. The SDK generates the ProofRequest nonce and an E2E key pair, and saves the E2E session with EcdhSessionProvider.saveSession. The returned profile does not include a Proof; the calling application must sign it and set the Proof before sending it to the holder.`

- The ProofRequest is built from the policy: `name` and `version` are copied, `requestedAttributes` and `requestedPredicates` (JSON strings) are parsed into the ProofRequest (skipped when `null` or empty), and `nonce` is a random large integer generated by the SDK.
- The E2E key pair is created with `CryptoHelper.generateEcKeyPair(request.reqE2e.curve)` and the E2E nonce with `CryptoHelper.generateNonce(16)`. The profile's `reqE2e` takes `curve`, `cipher` and `padding` from the request and `publicKey` and `nonce` from these calls.
- `type` is `VerifyProfile`; `title`, `description` and `language` come from the policy, and `profile.verifier` from `VerifierInfoProvider.getVerifierInfo()`.
- `EcdhSessionProvider.saveSession(request.txId, keyPair, reqE2e)` is called last.
- The policy's `curve`, `cipher` and `padding` and the request's `policyId` and `verifierDid` are not used.
- Any exception during creation, including exceptions thrown by providers and invalid JSON in the policy, is converted to `VerifierSdkException` (`SDK_UNKNOWN_ERROR`).

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

### Function Usage
```java
ProofRequestProfileRequest request = ProofRequestProfileRequest.builder()
    .txId(txId)
    .profileId(UUID.randomUUID().toString())
    .reqE2e(ReqE2e.builder()
        .curve("Secp256r1")
        .cipher("AES-256-CBC")
        .padding("PKCS5")
        .build())
    .build();

ZkpPolicy policy = ZkpPolicy.builder()
    .title("Age verification")
    .language("en")
    .name("age-check")
    .version("1.0")
    .requestedPredicates(requestedPredicatesJson)
    .build();

ProofRequestProfile profile = verifierService.requestZkpProofRequestProfile(request, policy);
ProofRequest proofRequest = profile.getProfile().getProofRequest(); // keep for verifyZkpProof
// Sign the profile and set its Proof (profile.setProof(...)) before sending it to the holder
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
- The SDK does not remove the E2E session.

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

- If `EcdhSessionProvider.existsSession(txId)` returns `false`, `VerifierSdkException` (`SDK_E2E_SESSION_NOT_FOUND`) is thrown.
- Otherwise `EcdhSessionProvider.decrypt(txId, accE2e.publicKey, encProof, iv)` is called and the result is parsed with Gson as a `Proof`. Any exception in this step, including a `null` `accE2e`, is converted to `SDK_DECRYPTION_FAILED`.
- The nonce and the Proof are not verified.

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

### Function Usage
```java
Proof proof = verifierService.decryptZkpProof(txId, encProof, iv, accE2e);
```

<br>

### 1.1.11 getOfferService

### Class Name
`VerifierService`

### Function Name
`getOfferService`

### Function Introduction
`Returns the VpOfferProtocol to which requestVpOffer and requestStaticVpOffer are delegated, always the same instance. For a VerifierService created by VerifierSdkBuilder this is a VpOfferProtocolImpl.`

### Input Parameters
n/a

### Output Parameters

| Type | Description | **M/O** | **Note** |
|------|-------------|---------|----------|
| VpOfferProtocol | VP Offer protocol | M | [Link](#13-vpofferprotocol) |

### Function Declaration

```java
VpOfferProtocol getOfferService()
```

### Function Usage
```java
VpOfferProtocol offerProtocol = verifierService.getOfferService();
```

<br>

### 1.1.12 getProfileService

### Class Name
`VerifierService`

### Function Name
`getProfileService`

### Function Introduction
`Returns the VpProfileProtocol to which requestVerifyProfile is delegated, always the same instance. For a VerifierService created by VerifierSdkBuilder this is a VpProfileProtocolImpl.`

### Input Parameters
n/a

### Output Parameters

| Type | Description | **M/O** | **Note** |
|------|-------------|---------|----------|
| VpProfileProtocol | VerifyProfile protocol | M | [Link](#14-vpprofileprotocol) |

### Function Declaration

```java
VpProfileProtocol getProfileService()
```

### Function Usage
```java
VpProfileProtocol profileProtocol = verifierService.getProfileService();
```

<br>

### 1.1.13 getVerificationService

### Class Name
`VerifierService`

### Function Name
`getVerificationService`

### Function Introduction
`Returns the VpVerificationProtocol to which verifyPresentation and decryptVp are delegated, always the same instance. For a VerifierService created by VerifierSdkBuilder this is a VpVerificationProtocolImpl.`

### Input Parameters
n/a

### Output Parameters

| Type | Description | **M/O** | **Note** |
|------|-------------|---------|----------|
| VpVerificationProtocol | VP verification protocol | M | [Link](#15-vpverificationprotocol) |

### Function Declaration

```java
VpVerificationProtocol getVerificationService()
```

### Function Usage
```java
VpVerificationProtocol verificationProtocol = verifierService.getVerificationService();
```

<br>

### 1.1.14 getConfirmService

### Class Name
`VerifierService`

### Function Name
`getConfirmService`

### Function Introduction
`Returns the VerificationConfirmProtocol to which both confirmVerification methods are delegated, always the same instance. For a VerifierService created by VerifierSdkBuilder this is a VerificationConfirmProtocolImpl.`

### Input Parameters
n/a

### Output Parameters

| Type | Description | **M/O** | **Note** |
|------|-------------|---------|----------|
| VerificationConfirmProtocol | Verification confirm protocol | M | [Link](#16-verificationconfirmprotocol) |

### Function Declaration

```java
VerificationConfirmProtocol getConfirmService()
```

### Function Usage
```java
VerificationConfirmProtocol confirmProtocol = verifierService.getConfirmService();
```

<br>

### 1.1.15 getZkpProofVerificationProtocol

### Class Name
`VerifierService`

### Function Name
`getZkpProofVerificationProtocol`

### Function Introduction
`Returns the ZkpProofVerificationProtocol to which requestZkpProofRequestProfile, verifyZkpProof and decryptZkpProof are delegated, always the same instance. For a VerifierService created by VerifierSdkBuilder this is a ZkpProofVerificationProtocolImpl.`

### Input Parameters
n/a

### Output Parameters

| Type | Description | **M/O** | **Note** |
|------|-------------|---------|----------|
| ZkpProofVerificationProtocol | ZKP Proof verification protocol | M | [Link](#17-zkpproofverificationprotocol) |

### Function Declaration

```java
ZkpProofVerificationProtocol getZkpProofVerificationProtocol()
```

### Function Usage
```java
ZkpProofVerificationProtocol zkpProtocol = verifierService.getZkpProofVerificationProtocol();
```

<br>

## 1.2 VerifierSdkBuilder

`VerifierSdkBuilder` (`org.omnione.did.verifier.v1.config`) creates a `VerifierService`. Set the seven providers with the methods below and call [build](#121-build). Each method stores the given provider and returns the same builder; it does not check for `null`, so a missing provider is detected by `build()`. Calling a method again replaces the provider.

| Method | Type | Description | **M/O** |
|--------|------|-------------|---------|
| `VerifierSdkBuilder configProvider(VerificationConfigProvider configProvider)` | VerificationConfigProvider | Policy provider. [Link](#21-verificationconfigprovider) | M |
| `VerifierSdkBuilder verifierInfoProvider(VerifierInfoProvider verifierInfoProvider)` | VerifierInfoProvider | Verifier information provider. [Link](#22-verifierinfoprovider) | M |
| `VerifierSdkBuilder sessionProvider(EcdhSessionProvider sessionProvider)` | EcdhSessionProvider | E2E session provider. [Link](#23-ecdhsessionprovider) | M |
| `VerifierSdkBuilder storageProvider(StorageProvider storageProvider)` | StorageProvider | DID Document, VC metadata and ZKP data provider. [Link](#24-storageprovider) | M |
| `VerifierSdkBuilder transactionProvider(TransactionProvider transactionProvider)` | TransactionProvider | Transaction ID provider. [Link](#25-transactionprovider) | M |
| `VerifierSdkBuilder nonceGenerator(NonceGenerator nonceGenerator)` | NonceGenerator | Verifier nonce generator. [Link](#26-noncegenerator) | M |
| `VerifierSdkBuilder cryptoHelper(CryptoHelper cryptoHelper)` | CryptoHelper | Cryptographic helper. [Link](#27-cryptohelper) | M |

<br>

### 1.2.1 build

### Class Name
`VerifierSdkBuilder`

### Function Name
`build`

### Function Introduction
`Checks that all seven providers are set and creates a VerifierService (VerifierServiceImpl) with the default protocol implementations. Providers are not called during build, and each call creates a new instance.`

- The providers are checked in the order `configProvider`, `verifierInfoProvider`, `sessionProvider`, `storageProvider`, `transactionProvider`, `nonceGenerator`, `cryptoHelper`. The first missing one throws `VerifierSdkException` (`SDK_CONFIGURATION_ERROR`) with the message `<interface name> is required.` (for example `StorageProvider is required.`).

### Input Parameters
n/a

### Output Parameters

| Type | Description | **M/O** | **Note** |
|------|-------------|---------|----------|
| VerifierService | SDK facade | M | [Link](#11-verifierservice) |

### Function Declaration

```java
public VerifierService build()
```

### Function Usage
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

## 1.3 VpOfferProtocol

`VpOfferProtocol` creates VP Offer payloads. Obtain it with [getOfferService](#1111-getofferservice); the `VerifierService` methods of the same name delegate to it. The default implementation is `VpOfferProtocolImpl` ([1.8](#18-implementation-classes)).

<br>

### 1.3.1 requestVpOffer

### Class Name
`VpOfferProtocol`

### Function Name
`requestVpOffer`

### Function Introduction
`Generates a VP Offer Payload for a Dynamic QR code flow, with an offerId created by TransactionProvider.createTransactionId and a validUntil based on the policy's validityDuration.`

- Behavior and exceptions are the same as [1.1.1 requestVpOffer](#111-requestvpoffer). If the policy is not found, `VerifierSdkException` (`SDK_POLICY_NOT_FOUND`) is thrown.

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
VpOfferPayload offer = verifierService.getOfferService().requestVpOffer(
    "policy-login",
    "kiosk-01",
    "login-service",
    false
);
```

<br>

### 1.3.2 requestStaticVpOffer

### Class Name
`VpOfferProtocol`

### Function Name
`requestStaticVpOffer`

### Function Introduction
`Generates a VP Offer Payload for a Static QR code flow, without offerId and validUntil.`

- Behavior and exceptions are the same as [1.1.2 requestStaticVpOffer](#112-requeststaticvpoffer).

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

### Function Usage
```java
VpOfferPayload offer = verifierService.getOfferService().requestStaticVpOffer(
    "policy-login",
    "kiosk-01",
    "login-service",
    false
);
```

<br>

## 1.4 VpProfileProtocol

`VpProfileProtocol` creates the VerifyProfile of a VP flow. Obtain it with [getProfileService](#1112-getprofileservice); `VerifierService.requestVerifyProfile` delegates to it. The default implementation is `VpProfileProtocolImpl` ([1.8](#18-implementation-classes)).

<br>

### 1.4.1 requestVerifyProfile

### Class Name
`VpProfileProtocol`

### Function Name
`requestVerifyProfile`

### Function Introduction
`Creates a VerifyProfile without Proof for the given policy, with a verifierNonce generated by NonceGenerator and the given reqE2e in the profile's process.`

- Behavior and exceptions are the same as [1.1.3 requestVerifyProfile](#113-requestverifyprofile). Store the generated `verifierNonce` for VP verification, and sign the profile before sending it.

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
VerifyProfile profile = verifierService.getProfileService().requestVerifyProfile(
    "policy-login",
    UUID.randomUUID().toString(),
    reqE2e
);
```

<br>

## 1.5 VpVerificationProtocol

`VpVerificationProtocol` decrypts and verifies VPs. Obtain it with [getVerificationService](#1113-getverificationservice). `verifyPresentation` runs the full pipeline; `validateAuthType`, `validateNonce` and `validateVcStatuses` are single steps of it and can be called on a parsed VP (`org.omnione.did.data.model.vp.VerifiablePresentation`). The default implementation is `VpVerificationProtocolImpl` ([1.8](#18-implementation-classes)).

<br>

### 1.5.1 verifyPresentation

### Class Name
`VpVerificationProtocol`

### Function Name
`verifyPresentation`

### Function Introduction
`Performs the full VP verification pipeline: E2E decryption → auth type check → nonce check → VpManager verification → VC status check, and returns the verified VP JSON.`

- Behavior and exceptions are the same as [1.1.4 verifyPresentation](#114-verifypresentation).

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
String vpJson = verifierService.getVerificationService().verifyPresentation(request);
```

<br>

### 1.5.2 decryptVp

### Class Name
`VpVerificationProtocol`

### Function Name
`decryptVp`

### Function Introduction
`Decrypts an E2E-encrypted VP with the E2E session of txId and returns the VP JSON without verifying it.`

- Behavior and exceptions are the same as [1.1.5 decryptVp](#115-decryptvp).

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

### Function Usage
```java
String vpJson = verifierService.getVerificationService().decryptVp(txId, encHolderPublicKey, encVp, iv);
```

<br>

### 1.5.3 validateAuthType

### Class Name
`VpVerificationProtocol`

### Function Name
`validateAuthType`

### Function Introduction
`Checks the authentication type of the VP against requiredAuthType. This is the auth type step of verifyPresentation.`

- `requiredAuthType` is `null` or `0x00000000`: no check.
- `0x00000006` (PIN or BIO): the fragment after `#` of `vp.proof.verificationMethod` must be `pin` or `bio` (case-insensitive). If `proof` or `verificationMethod` is missing, `VerifierSdkException` (`SDK_INVALID_PROOF`) is thrown; if there is no fragment or it has another value, `SDK_INVALID_AUTH_TYPE` is thrown.
- Any other value: the SDK returns without checking. The application must check other auth types itself.

### Input Parameters

| Parameter | Type | Description | **M/O** | **Note** |
|-----------|------|-------------|---------|----------|
| vp | VerifiablePresentation | Parsed VP | M | `org.omnione.did.data.model.vp.VerifiablePresentation` |
| requiredAuthType | Integer | Required auth type bitmask | O | `null`, `0x00000000`, `0x00000006` or another value |

### Output Parameters
n/a

### Function Declaration

```java
void validateAuthType(VerifiablePresentation vp, Integer requiredAuthType)
```

### Function Usage
```java
VerifiablePresentation vp = new VerifiablePresentation();
vp.fromJson(vpJson);

verifierService.getVerificationService().validateAuthType(vp, 0x00000006);
```

<br>

### 1.5.4 validateNonce

### Class Name
`VpVerificationProtocol`

### Function Name
`validateNonce`

### Function Introduction
`Checks that the verifierNonce of the VP equals expectedNonce. This is the nonce step of verifyPresentation.`

- If the VP's `verifierNonce` is `null` or empty, or differs from `expectedNonce` (exact string comparison), `VerifierSdkException` (`SDK_INVALID_NONCE`) is thrown.
- `expectedNonce` is not checked for `null`; a `null` value causes `NullPointerException`.

### Input Parameters

| Parameter | Type | Description | **M/O** | **Note** |
|-----------|------|-------------|---------|----------|
| vp | VerifiablePresentation | Parsed VP | M | `org.omnione.did.data.model.vp.VerifiablePresentation` |
| expectedNonce | String | Expected verifier nonce | M | `verifierNonce` stored from the VerifyProfile |

### Output Parameters
n/a

### Function Declaration

```java
void validateNonce(VerifiablePresentation vp, String expectedNonce)
```

### Function Usage
```java
verifierService.getVerificationService().validateNonce(vp, storedVerifierNonce);
```

<br>

### 1.5.5 validateVcStatuses

### Class Name
`VpVerificationProtocol`

### Function Name
`validateVcStatuses`

### Function Introduction
`Checks the status of each VC in the VP with StorageProvider.getVcMeta. This is the last step of verifyPresentation.`

- A VP without VCs throws `VerifierSdkException` (`SDK_INVALID_VP`).
- VCs without `id` are skipped. `getVcMeta(vcId)` is called for each other VC; if it returns `null`, that VC is skipped.
- The status is read from `status`, `vcStatus` or `credentialStatus.status` of the metadata, in that order, and compared case-insensitively. `REVOKED`, `INACTIVE` and `EXPIRED` throw `SDK_INVALID_VC`; `ACTIVE`, other values and a missing status pass.
- Exceptions thrown by `StorageProvider` and errors parsing the metadata JSON are passed through unchanged. Signatures and validity periods are not checked in this step.

### Input Parameters

| Parameter | Type | Description | **M/O** | **Note** |
|-----------|------|-------------|---------|----------|
| vp | VerifiablePresentation | Parsed VP | M | `org.omnione.did.data.model.vp.VerifiablePresentation` |

### Output Parameters
n/a

### Function Declaration

```java
void validateVcStatuses(VerifiablePresentation vp)
```

### Function Usage
```java
verifierService.getVerificationService().validateVcStatuses(vp);
```

<br>

## 1.6 VerificationConfirmProtocol

`VerificationConfirmProtocol` builds the verification result and extracts the holder DID, VCs and claims from a VP. Obtain it with [getConfirmService](#1114-getconfirmservice). The extract methods take the VP as a Gson `com.google.gson.JsonObject` and do not verify it, so call them only for a VP verified with `verifyPresentation`. The default implementation is `VerificationConfirmProtocolImpl` ([1.8](#18-implementation-classes)).

<br>

### 1.6.1 confirmVerification (VP)

### Class Name
`VerificationConfirmProtocol`

### Function Name
`confirmVerification`

### Function Introduction
`Extracts the holder DID, submitted VCs and claims from a verified VP and returns the verification result. The SDK does not store it.`

- Behavior and exceptions are the same as [1.1.6 confirmVerification (VP)](#116-confirmverification-vp).

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
VerificationConfirmResult result = verifierService.getConfirmService().confirmVerification(txId, vpJson, true);
```

<br>

### 1.6.2 confirmVerification (VP or Proof)

### Class Name
`VerificationConfirmProtocol`

### Function Name
`confirmVerification`

### Function Introduction
`Returns the verification result for a verified VP or ZKP Proof according to the offer type. The SDK does not store it.`

- Behavior and exceptions are the same as [1.1.7 confirmVerification (VP or Proof)](#117-confirmverification-vp-or-proof).

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

### Function Usage
```java
VerificationConfirmResult result = verifierService.getConfirmService().confirmVerification(
    txId,
    vpJson,
    true,
    OfferType.VerifyOffer
);
```

<br>

### 1.6.3 extractHolderDid

### Class Name
`VerificationConfirmProtocol`

### Function Name
`extractHolderDid`

### Function Introduction
`Returns the holder DID (holder property) of the VP.`

- If the VP has no `holder`, `VerifierSdkException` (`SDK_INVALID_VP`) is thrown. The value is read as a string.

### Input Parameters

| Parameter | Type | Description | **M/O** | **Note** |
|-----------|------|-------------|---------|----------|
| vpObject | JsonObject | VP JSON object | M | `com.google.gson.JsonObject` |

### Output Parameters

| Type | Description | **M/O** | **Note** |
|------|-------------|---------|----------|
| String | Holder DID | M | |

### Function Declaration

```java
String extractHolderDid(JsonObject vpObject)
```

### Function Usage
```java
JsonObject vpObject = new Gson().fromJson(vpJson, JsonObject.class);
String holderDid = verifierService.getConfirmService().extractHolderDid(vpObject);
```

<br>

### 1.6.4 extractSubmittedVcs

### Class Name
`VerificationConfirmProtocol`

### Function Name
`extractSubmittedVcs`

### Function Introduction
`Returns the IDs of the VCs in the verifiableCredential array of the VP, in array order.`

- VCs without `id` are skipped. If the VP has no `verifiableCredential`, an empty list is returned.

### Input Parameters

| Parameter | Type | Description | **M/O** | **Note** |
|-----------|------|-------------|---------|----------|
| vpObject | JsonObject | VP JSON object | M | `com.google.gson.JsonObject` |

### Output Parameters

| Type | Description | **M/O** | **Note** |
|------|-------------|---------|----------|
| List\<String\> | VC ID list | M | Empty list if there are no VCs |

### Function Declaration

```java
List<String> extractSubmittedVcs(JsonObject vpObject)
```

### Function Usage
```java
List<String> vcIds = verifierService.getConfirmService().extractSubmittedVcs(vpObject);
```

<br>

### 1.6.5 extractSubmittedVcsAsMap

### Class Name
`VerificationConfirmProtocol`

### Function Name
`extractSubmittedVcsAsMap`

### Function Introduction
`Returns the VCs in the VP as a map of VC ID to VC object.`

- Each VC is converted from JSON to `Map` with Gson (JSON numbers become `Double`). VCs without `id` are skipped. If the VP has no `verifiableCredential`, an empty map is returned.

### Input Parameters

| Parameter | Type | Description | **M/O** | **Note** |
|-----------|------|-------------|---------|----------|
| vpObject | JsonObject | VP JSON object | M | `com.google.gson.JsonObject` |

### Output Parameters

| Type | Description | **M/O** | **Note** |
|------|-------------|---------|----------|
| Map\<String, Object\> | VC ID → VC object | M | Empty map if there are no VCs |

### Function Declaration

```java
Map<String, Object> extractSubmittedVcsAsMap(JsonObject vpObject)
```

### Function Usage
```java
Map<String, Object> vcs = verifierService.getConfirmService().extractSubmittedVcsAsMap(vpObject);
```

<br>

### 1.6.6 extractClaims

### Class Name
`VerificationConfirmProtocol`

### Function Name
`extractClaims`

### Function Introduction
`Extracts the claims of all VCs in the VP as a map of claim code to claim value.`

- Reads `credentialSubject.claims` of each VC and adds the entries that have both `code` and `value`. The value is read as a string; a value that is not a JSON primitive causes an exception from Gson.
- If the same code appears in more than one VC, the value of the later VC is kept. If the VP has no `verifiableCredential`, an empty map is returned.

### Input Parameters

| Parameter | Type | Description | **M/O** | **Note** |
|-----------|------|-------------|---------|----------|
| vpObject | JsonObject | VP JSON object | M | `com.google.gson.JsonObject` |

### Output Parameters

| Type | Description | **M/O** | **Note** |
|------|-------------|---------|----------|
| Map\<String, Object\> | Claim code → value | M | Values are `String` |

### Function Declaration

```java
Map<String, Object> extractClaims(JsonObject vpObject)
```

### Function Usage
```java
Map<String, Object> claims = verifierService.getConfirmService().extractClaims(vpObject);
```

<br>

### 1.6.7 extractFilteredClaims

### Class Name
`VerificationConfirmProtocol`

### Function Name
`extractFilteredClaims`

### Function Introduction
`Extracts claims in the same way as extractClaims and returns only the codes in requiredClaims.`

- Codes that are not in the VP are omitted without error. `requiredClaims` must not be `null` (`NullPointerException`).

### Input Parameters

| Parameter | Type | Description | **M/O** | **Note** |
|-----------|------|-------------|---------|----------|
| vpObject | JsonObject | VP JSON object | M | `com.google.gson.JsonObject` |
| requiredClaims | List\<String\> | Claim codes to return | M | |

### Output Parameters

| Type | Description | **M/O** | **Note** |
|------|-------------|---------|----------|
| Map\<String, Object\> | Claim code → value | M | Only codes in `requiredClaims` |

### Function Declaration

```java
Map<String, Object> extractFilteredClaims(
    JsonObject vpObject,
    List<String> requiredClaims
)
```

### Function Usage
```java
Map<String, Object> claims = verifierService.getConfirmService().extractFilteredClaims(
    vpObject,
    List.of("org.example.name", "org.example.birth")
);
```

<br>

## 1.7 ZkpProofVerificationProtocol

`ZkpProofVerificationProtocol` creates ZKP ProofRequestProfiles and decrypts and verifies ZKP Proofs. Obtain it with [getZkpProofVerificationProtocol](#1115-getzkpproofverificationprotocol). The default implementation is `ZkpProofVerificationProtocolImpl` ([1.8](#18-implementation-classes)).

<br>

### 1.7.1 requestZkpProofRequestProfile

### Class Name
`ZkpProofVerificationProtocol`

### Function Name
`requestZkpProofRequestProfile`

### Function Introduction
`Creates a ProofRequestProfile without Proof, generating the ProofRequest nonce and an E2E key pair, and saves the E2E session with EcdhSessionProvider.saveSession.`

- Behavior and exceptions are the same as [1.1.8 requestZkpProofRequestProfile](#118-requestzkpproofrequestprofile). Any error is thrown as `VerifierSdkException` (`SDK_UNKNOWN_ERROR`).

### Input Parameters

| Parameter | Type | Description | **M/O** | **Note** |
|-----------|------|-------------|---------|----------|
| request | ProofRequestProfileRequest | Profile request metadata | M | Uses `txId`, `profileId` and `reqE2e` (`curve`, `cipher`, `padding`). [Link](#310-proofrequestprofilerequest) |
| policy | ZkpPolicy | ZKP policy configuration | M | Uses `title`, `description`, `language`, `name`, `version`, `requestedAttributes`, `requestedPredicates`. [Link](#311-zkppolicy) |

### Output Parameters

| Type | Description | **M/O** | **Note** |
|------|-------------|---------|----------|
| ProofRequestProfile | ZKP proof request profile | M | Proof not included. [Link](#38-proofrequestprofile) |

### Function Declaration

```java
ProofRequestProfile requestZkpProofRequestProfile(
    ProofRequestProfileRequest request,
    ZkpPolicy policy
)
```

### Function Usage
```java
ProofRequestProfile profile = verifierService.getZkpProofVerificationProtocol()
    .requestZkpProofRequestProfile(request, policy);
```

<br>

### 1.7.2 verifyZkpProof

### Class Name
`ZkpProofVerificationProtocol`

### Function Name
`verifyZkpProof`

### Function Introduction
`Decrypts a submitted ZKP Proof, checks its nonce against the stored proofRequest and verifies the Proof with the ZKP library.`

- Behavior and exceptions are the same as [1.1.9 verifyZkpProof](#119-verifyzkpproof). An invalid Proof returns `verified=false`; request errors throw `VerifierSdkException`.

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
ZkpVerificationResult result = verifierService.getZkpProofVerificationProtocol().verifyZkpProof(request);
```

<br>

### 1.7.3 decryptZkpProof

### Class Name
`ZkpProofVerificationProtocol`

### Function Name
`decryptZkpProof`

### Function Introduction
`Decrypts an E2E-encrypted ZKP Proof with the E2E session of txId without verifying it.`

- Behavior and exceptions are the same as [1.1.10 decryptZkpProof](#1110-decryptzkpproof).

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

### Function Usage
```java
Proof proof = verifierService.getZkpProofVerificationProtocol().decryptZkpProof(txId, encProof, iv, accE2e);
```

<br>

## 1.8 Implementation Classes

The SDK's default implementations are in the `org.omnione.did.verifier.v1.core` package. `VerifierSdkBuilder.build()` creates them, so the application normally does not create them directly.

- `VerifierServiceImpl` implements `VerifierService`. See [1.1 VerifierService](#11-verifierservice).
- `VpOfferProtocolImpl` implements `VpOfferProtocol`. See [1.3 VpOfferProtocol](#13-vpofferprotocol).
- `VpProfileProtocolImpl` implements `VpProfileProtocol`. See [1.4 VpProfileProtocol](#14-vpprofileprotocol).
- `VpVerificationProtocolImpl` implements `VpVerificationProtocol`. See [1.5 VpVerificationProtocol](#15-vpverificationprotocol).
- `VerificationConfirmProtocolImpl` implements `VerificationConfirmProtocol`. See [1.6 VerificationConfirmProtocol](#16-verificationconfirmprotocol).
- `ZkpProofVerificationProtocolImpl` implements `ZkpProofVerificationProtocol`. See [1.7 ZkpProofVerificationProtocol](#17-zkpproofverificationprotocol).

Their only public members outside the interfaces are the constructors below. None of them checks its arguments for `null`.

| Class | Constructor | **Note** |
|-------|-------------|----------|
| VerifierServiceImpl | `VerifierServiceImpl(VerificationConfigProvider configProvider, VerifierInfoProvider verifierInfoProvider, EcdhSessionProvider sessionProvider, StorageProvider storageService, TransactionProvider transactionManager, NonceGenerator nonceGenerator, CryptoHelper cryptoHelper)` | Used by `build()`. Creates the five default protocol implementations |
| VerifierServiceImpl | `VerifierServiceImpl(VpOfferProtocol offerService, VpProfileProtocol profileService, VpVerificationProtocol verificationService, VerificationConfirmProtocol confirmService, ZkpProofVerificationProtocol zkpProofVerificationService)` | Uses the given protocol implementations. Use it to replace some protocols with custom implementations |
| VpOfferProtocolImpl | `VpOfferProtocolImpl(VerificationConfigProvider configProvider, TransactionProvider transactionManager)` |  |
| VpProfileProtocolImpl | `VpProfileProtocolImpl(VerificationConfigProvider configProvider, VerifierInfoProvider verifierInfoProvider, NonceGenerator nonceGenerator)` |  |
| VpVerificationProtocolImpl | `VpVerificationProtocolImpl(EcdhSessionProvider sessionProvider, StorageProvider storageService, CryptoHelper cryptoHelper)` | `cryptoHelper` is not used |
| VerificationConfirmProtocolImpl | `VerificationConfirmProtocolImpl()` |  |
| ZkpProofVerificationProtocolImpl | `ZkpProofVerificationProtocolImpl(EcdhSessionProvider sessionProvider, StorageProvider storageService, VerifierInfoProvider verifierInfoProvider, CryptoHelper cryptoHelper)` |  |

### Usage
```java
// customVerification: the application's own VpVerificationProtocol implementation
VerifierService verifierService = new VerifierServiceImpl(
    new VpOfferProtocolImpl(configProvider, transactionProvider),
    new VpProfileProtocolImpl(configProvider, verifierInfoProvider, nonceGenerator),
    customVerification,
    new VerificationConfirmProtocolImpl(),
    new ZkpProofVerificationProtocolImpl(sessionProvider, storageProvider, verifierInfoProvider, cryptoHelper)
);
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

### Methods

| Method | Description | **SDK Call** | **On Failure** |
|--------|-------------|--------------|----------------|
| `VerificationPolicy getPolicy(String policyId)` | Returns the policy of `policyId` as a [VerificationPolicy](#37-verificationpolicy). The SDK reads `policyName`, `description`, `language`, `mode`, `endpoints`, `validityDuration`, `filter` and `process` | Once in each `requestVpOffer`, `requestStaticVpOffer` and `requestVerifyProfile` call, with the `policyId` given by the application | Return `null` or throw. `null` → the SDK throws `SDK_POLICY_NOT_FOUND`. Exceptions are passed to the caller unchanged |
| `boolean existsPolicy(String policyId)` | Returns whether the policy exists | Not called by the SDK |  |

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

### Methods

| Method | Description | **SDK Call** | **On Failure** |
|--------|-------------|--------------|----------------|
| `ProviderDetail getVerifierInfo()` | Returns the verifier's `ProviderDetail` (`org.omnione.did.data.model.provider.ProviderDetail`: `did`, `certVcRef`, `name`, `description`, `logo`, `ref`). The SDK sets it unchanged as `profile.verifier` | Once in each `requestVerifyProfile` and `requestZkpProofRequestProfile` call | `null` is not checked. In `requestVerifyProfile` exceptions are passed through unchanged; in `requestZkpProofRequestProfile` they become `SDK_UNKNOWN_ERROR` |
| `String getVerifierDidDocument()` | Returns the verifier's DID Document as a JSON string | Not called by the SDK |  |
| `String getVerifierDid()` | Returns the verifier's DID | Not called by the SDK |  |

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

### Methods

| Method | Description | **SDK Call** | **On Failure** |
|--------|-------------|--------------|----------------|
| `ReqE2e createSession(String txId)` | Creates an ephemeral E2E key pair and nonce for `txId`, keeps the private key, nonce, curve, cipher and padding under `txId`, and returns them as [ReqE2e](#35-reqe2e) (`publicKey` and `nonce` Multibase encoded) | Not called by the SDK. The application calls it before `requestVerifyProfile` and passes the result as `reqE2e` | Up to the application |
| `void saveSession(String txId, KeyPairInfo keyPair, ReqE2e reqE2e)` | Stores the key pair and E2E settings generated by the SDK under `txId`. At least `keyPair.privateKey` and the `nonce`, `curve`, `cipher` and `padding` of `reqE2e` must be kept for `decrypt` | Once at the end of `requestZkpProofRequestProfile`, with `request.txId`, the key pair from `CryptoHelper.generateEcKeyPair` and the `reqE2e` of the returned profile | Throw an exception; the SDK converts it to `SDK_UNKNOWN_ERROR` |
| `ReqE2e getSession(String txId)` | Returns the stored E2E settings of `txId` | Not called by the SDK |  |
| `void removeSession(String txId)` | Deletes the session of `txId` | Not called by the SDK. The application removes the session when the transaction ends |  |
| `boolean existsSession(String txId)` | Returns whether a session exists for `txId` | First step of `decryptVp`, `verifyPresentation`, `decryptZkpProof` and `verifyZkpProof` | `false` → the SDK throws `SDK_E2E_SESSION_NOT_FOUND`. Exceptions are passed through unchanged, so return `false` for a missing session |
| `String decrypt(String txId, String encHolderPublicKey, String encVp, String iv)` | Decrypts the data with the session of `txId`: ECDH with the verifier private key and `encHolderPublicKey`, a session key derived from the shared secret and the session nonce, then symmetric decryption with `iv` and the session's cipher and padding. Returns the plaintext as a UTF-8 string. All inputs are Multibase encoded; `encHolderPublicKey` is the holder's public key (not encrypted) | After `existsSession` returns `true`. VP: `encHolderPublicKey`, `encVp` and `iv` of the request. ZKP: `accE2e.publicKey`, `encProof` and `iv` | Throw an exception; the SDK converts any exception to `SDK_DECRYPTION_FAILED`. The return value is not checked for `null` |

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

### Methods

| Method | Description | **SDK Call** | **On Failure** |
|--------|-------------|--------------|----------------|
| `String findDidDocument(String did)` | Returns the DID Document of `did` as a JSON string (parsed with `DidDocument.fromJson`) | `verifyPresentation`: once for the VP holder DID and once for the issuer DID of each VC | `null` → the SDK throws `SDK_DID_DOCUMENT_NOT_FOUND`. Exceptions are passed through unchanged |
| `String getVcMeta(String vcId)` | Returns the VC metadata of `vcId` as a JSON object string. Only the status field is read | `verifyPresentation` and `validateVcStatuses`: once for each VC that has an `id` | `null` → the status check of that VC is skipped. Exceptions and JSON parsing errors are passed through unchanged |
| `boolean existsDidDocument(String did)` | Returns whether the DID Document exists | Not called by the SDK |  |
| `CredentialSchema getZKPCredential(String schemaId)` | Returns the ZKP Credential Schema of `schemaId` | `verifyZkpProof`: once for each identifier of the Proof, with `identifiers[].schemaId` | `null` is not checked and is passed to the ZKP library. `VerifierSdkException` is passed through; other exceptions become `SDK_SIGNATURE_VERIFICATION_FAILED` |
| `CredentialDefinition getZKPCredentialDefinition(String credDefId)` | Returns the ZKP Credential Definition of `credDefId` | `verifyZkpProof`: once for each identifier of the Proof, with `identifiers[].credDefId` | Same as `getZKPCredential` |

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

### Methods

| Method | Description | **SDK Call** | **On Failure** |
|--------|-------------|--------------|----------------|
| `String createTransactionId()` | Returns a new unique transaction ID (UUID recommended) | Once per `requestVpOffer` call; the value becomes `offerId`. Not called by `requestStaticVpOffer` | Not checked for `null`. Exceptions are passed through unchanged |
| `Long getTransactionId(String txId)` | Returns the application's internal key (e.g. DB primary key) of `txId` | Not called by the SDK |  |

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

### Methods

| Method | Description | **SDK Call** | **On Failure** |
|--------|-------------|--------------|----------------|
| `String generateNonce(int length)` | Returns a cryptographically secure random value of `length` bytes as a string (Base64 recommended) | Through `generateNonce()`, with `length` = 16 | Exceptions are passed through unchanged |
| `default String generateNonce()` | Calls `generateNonce(16)`. Can be overridden | Once per `requestVerifyProfile` call; the value becomes `profile.process.verifierNonce` | Not checked. The value must not be empty because the VP's `verifierNonce` is compared with it |

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

### Methods

| Method | Description | **SDK Call** | **On Failure** |
|--------|-------------|--------------|----------------|
| `boolean verifySignature(String publicKey, String signature, byte[] data)` | Verifies `signature` over `data` with `publicKey` (Base64 or Multibase) and returns the result | Not called by the SDK. VP and VC signatures are verified by `VpManager` of did-core-sdk-server |  |
| `String sha256(byte[] data)` | Returns the SHA-256 hash of `data`, Base64 encoded | Not called by the SDK |  |
| `byte[] decodeMultibase(String multibase)` | Decodes a Multibase string | Not called by the SDK |  |
| `String encodeBase64(byte[] data)` | Encodes `data` in Base64 | Not called by the SDK |  |
| `byte[] decodeBase64(String base64)` | Decodes a Base64 string | Not called by the SDK |  |
| `KeyPairInfo generateKeyPair(String curve)` | Generates an EC key pair for `curve` (`Secp256r1`, `Secp256k1`); both keys Multibase encoded | Not called by the SDK |  |
| `String generateNonce(int length)` | Generates a random nonce of `length` bytes, Multibase encoded | `requestZkpProofRequestProfile`, with `length` = 16. The value becomes `reqE2e.nonce` of the profile and is passed to `saveSession` | Exceptions become `SDK_UNKNOWN_ERROR` |
| `String encodeMultibase(byte[] data)` | Encodes `data` in Multibase | Not called by the SDK |  |
| `byte[] generateSharedSecret(byte[] holderPublicKey, byte[] verifierPrivateKey, String curve)` | Computes the ECDH shared secret | Not called by the SDK |  |
| `byte[] deriveSessionKey(byte[] sharedSecret, byte[] nonce, String cipherType)` | Derives the symmetric session key for `cipherType` (e.g. `AES-256-CBC`) from the shared secret and nonce | Not called by the SDK |  |
| `byte[] decrypt(byte[] encData, byte[] sessionKey, byte[] iv, String cipherType, String paddingType)` | Decrypts `encData` with the session key (`paddingType` e.g. `PKCS5`) | Not called by the SDK |  |
| `KeyPairInfo generateEcKeyPair(String curve)` | Generates an EC key pair for E2E encryption; both keys Multibase encoded. `publicKey` is sent to the holder as `reqE2e.publicKey`, and the key pair is passed to `saveSession` | `requestZkpProofRequestProfile`, with `request.reqE2e.curve` | Exceptions become `SDK_UNKNOWN_ERROR`. A `null` return also ends in `SDK_UNKNOWN_ERROR` |

<br>

# 3. Data Classes

## 3.1 VpOfferPayload

### Declaration

```java
@Getter
@Builder
@NoArgsConstructor
@AllArgsConstructor
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
@NoArgsConstructor
@AllArgsConstructor
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
@NoArgsConstructor
@AllArgsConstructor
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
@NoArgsConstructor
@AllArgsConstructor
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
    @NoArgsConstructor
    @AllArgsConstructor
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
@NoArgsConstructor
@AllArgsConstructor
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
@NoArgsConstructor
@AllArgsConstructor
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
    @NoArgsConstructor
    @AllArgsConstructor
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

<br>

## 3.7 VerificationPolicy

### Declaration

```java
@Getter
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class VerificationPolicy {
    private String policyId;
    private String policyName;
    private String description;
    private Object logo;
    private String language;
    private String mode;
    private List<String> endpoints;
    private int validityDuration;
    private Filter filter;            // org.omnione.did.data.model.profile.Filter
    private VerifyProcess process;    // org.omnione.did.data.model.profile.verify.VerifyProcess
    private ProviderDetail verifier;  // org.omnione.did.data.model.provider.ProviderDetail
}
```

### Property

| Parameter | Type | Description | **M/O** | **Note** |
|-----------|------|-------------|---------|----------|
| policyId | String | Policy identifier | O | Not used by the SDK |
| policyName | String | Policy name | M | `title` of the VerifyProfile |
| description | String | Policy description | O | `description` of the VerifyProfile. Empty string if `null` |
| logo | Object | Logo image | O | Not used by the SDK |
| language | String | Language code | M | e.g. `ko`, `en`. `language` of the VerifyProfile |
| mode | String | Presentation mode | M | `Direct`, `Indirect` or `Proxy` ([PresentMode](#315-presentmode) display name). Other values make the offer methods throw `IllegalArgumentException` |
| endpoints | List\<String\> | Verifier API endpoints | O | `endpoints` of the VP Offer |
| validityDuration | int | Offer validity period (seconds) | M | `validUntil` of a Dynamic VP Offer = now + this value. Default: 0 |
| filter | Filter | Claim filter | M | `profile.filter` of the VerifyProfile. Not checked by the SDK |
| process | VerifyProcess | Base process | O | `endpoints` and `authType` are copied to `profile.process` of the VerifyProfile; `reqE2e` and `verifierNonce` are set by the SDK |
| verifier | ProviderDetail | Verifier information | O | Not used by the SDK; `VerifierInfoProvider.getVerifierInfo()` is used instead |

`VerificationConfigProvider.getPolicy` returns this class. When verifying the VP, pass the `authType` and `filter` of the same policy as `VpVerificationRequest.requiredAuthType` and `filter`.

<br>

## 3.8 ProofRequestProfile

### Declaration

```java
@Getter
@Setter
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class ProofRequestProfile {
    private String id;
    private String type;
    private String title;
    private String description;
    private String language;
    private ZkpInnerProfile profile;
    private Object proof;
}
```

### Property

| Parameter | Type | Description | **M/O** | **Note** |
|-----------|------|-------------|---------|----------|
| id | String | Profile identifier | M | `profileId` of the request |
| type | String | Profile type | M | Always `VerifyProfile` |
| title | String | Title | M | `title` of the ZkpPolicy |
| description | String | Description | O | `description` of the ZkpPolicy |
| language | String | Language | O | `language` of the ZkpPolicy |
| profile | ZkpInnerProfile | Inner profile | M | [Link](#39-zkpinnerprofile) |
| proof | Object | Verifier's signature | O | `null` in the SDK result. The application signs the profile and sets it with `setProof` |

<br>

## 3.9 ZkpInnerProfile

### Declaration

```java
@Getter
@Setter
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class ZkpInnerProfile {
    private Object verifier;            // ProviderDetail
    private ProofRequest proofRequest;  // org.omnione.did.zkp.datamodel.proofrequest.ProofRequest
    private ReqE2e reqE2e;
}
```

### Property

| Parameter | Type | Description | **M/O** | **Note** |
|-----------|------|-------------|---------|----------|
| verifier | Object | Verifier information | M | `ProviderDetail` from `VerifierInfoProvider.getVerifierInfo()` |
| proofRequest | ProofRequest | ZKP proof request | M | `org.omnione.did.zkp.datamodel.proofrequest.ProofRequest`. Keep it for `ZkpVerificationRequest.proofRequest` |
| reqE2e | ReqE2e | Verifier's E2E key exchange information | M | `publicKey` and `nonce` are generated by the SDK. [Link](#35-reqe2e) |

<br>

## 3.10 ProofRequestProfileRequest

### Declaration

```java
@Getter
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class ProofRequestProfileRequest {
    private String txId;
    private String policyId;
    private String profileId;
    private ReqE2e reqE2e;
    private String verifierDid;
}
```

### Property

| Parameter | Type | Description | **M/O** | **Note** |
|-----------|------|-------------|---------|----------|
| txId | String | Transaction identifier | M | Key of the E2E session saved with `saveSession`. Use the same value in `ZkpVerificationRequest.txId` |
| policyId | String | ZKP policy identifier | O | Not used by the SDK |
| profileId | String | Profile identifier | M | `id` of the ProofRequestProfile. UUID recommended |
| reqE2e | ReqE2e | E2E settings | M | Only `curve`, `cipher` and `padding` are read. [Link](#35-reqe2e) |
| verifierDid | String | Verifier DID | O | Not used by the SDK |

<br>

## 3.11 ZkpPolicy

### Declaration

```java
@Getter
@Setter
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class ZkpPolicy {
    private String policyId;
    private String title;
    private String description;
    private String language;
    private String name;
    private String version;
    private String requestedAttributes;
    private String requestedPredicates;
    private String curve;
    private String cipher;
    private String padding;
}
```

### Property

| Parameter | Type | Description | **M/O** | **Note** |
|-----------|------|-------------|---------|----------|
| policyId | String | Policy identifier | O | Not used by the SDK |
| title | String | Profile title | M | `title` of the ProofRequestProfile |
| description | String | Profile description | O | |
| language | String | Profile language | O | e.g. `ko`, `en` |
| name | String | ProofRequest name | M | `proofRequest.name` |
| version | String | ProofRequest version | M | `proofRequest.version` |
| requestedAttributes | String | Requested attributes (JSON) | O | JSON object of referent → `AttributeInfo` (`name`, `restrictions`), parsed into `proofRequest.requestedAttributes`. Ignored if `null` or empty |
| requestedPredicates | String | Requested predicates (JSON) | O | JSON object of referent → `PredicateInfo` (`name`, `pType` (`GE`, `LE`, `GT`, `LT`, `EQ`), `pValue`, `restrictions`), parsed into `proofRequest.requestedPredicates`. Ignored if `null` or empty |
| curve | String | E2E curve | O | Not used by the SDK (`ProofRequestProfileRequest.reqE2e` is used) |
| cipher | String | E2E cipher | O | Not used by the SDK |
| padding | String | E2E padding | O | Not used by the SDK |

<br>

## 3.12 KeyPairInfo

### Declaration

```java
@Getter
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class KeyPairInfo {
    private String publicKey;
    private String privateKey;
}
```

### Property

| Parameter | Type | Description | **M/O** | **Note** |
|-----------|------|-------------|---------|----------|
| publicKey | String | Public key | M | Multibase encoded. Sent to the holder as `reqE2e.publicKey` |
| privateKey | String | Private key | M | Multibase encoded. Kept only by the verifier for `EcdhSessionProvider.decrypt` |

Returned by `CryptoHelper.generateKeyPair` and `generateEcKeyPair`, and passed to `EcdhSessionProvider.saveSession`.

<br>

## 3.13 ProofVerifyParam

### Declaration

```java
@Getter
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class ProofVerifyParam {
    private Object credentialSchema;
    private Object credentialDefinition;
    private String schemaId;
    private String credDefId;
}
```

### Property

| Parameter | Type | Description | **M/O** | **Note** |
|-----------|------|-------------|---------|----------|
| credentialSchema | Object | Credential Schema | O | `org.omnione.did.zkp.datamodel.schema.CredentialSchema` |
| credentialDefinition | Object | Credential Definition | O | `org.omnione.did.zkp.datamodel.definition.CredentialDefinition` |
| schemaId | String | Schema ID | O | |
| credDefId | String | Credential Definition ID | O | |

`org.omnione.did.verifier.v1.model.data.ProofVerifyParam` is the element type of `ZkpVerificationRequest.proofVerifyParams`. `verifyZkpProof` does not use it; it fetches Schemas and Definitions through `StorageProvider`.

<br>

## 3.14 OfferType

### Declaration

```java
public enum OfferType {
    VerifyOffer,
    VerifyProofOffer
}
```

### Property

| Value | Description | **Note** |
|-------|-------------|----------|
| VerifyOffer | VP verification offer | Set by `requestVpOffer` and `requestStaticVpOffer` |
| VerifyProofOffer | ZKP Proof verification offer | `confirmVerification` does not parse the JSON |

<br>

## 3.15 PresentMode

### Declaration

```java
public enum PresentMode {
    DIRECT("Direct"),
    INDIRECT("Indirect"),
    PROXY("Proxy");

    public String getDisplayName();
    @JsonValue
    public String toString();
    public static PresentMode fromDisplayName(String displayName);
}
```

### Property

| Value | Display Name | Description |
|-------|--------------|-------------|
| DIRECT | Direct | The holder's app submits the VP directly to the verifier |
| INDIRECT | Indirect | The VP is passed to the verifier through the service device |
| PROXY | Proxy | The VP is passed through a proxy server |

`fromDisplayName` returns the constant for a display name (case-sensitive) and throws `IllegalArgumentException` for any other value. `toString()` returns the display name, which Jackson uses for serialization (`@JsonValue`); Gson serializes the constant name (e.g. `DIRECT`).

<br>

## 3.16 VerifierSdkException

### Declaration

```java
@Getter
public class VerifierSdkException extends RuntimeException {
    private VerifierSdkErrorCode errorCode;

    public VerifierSdkException(VerifierSdkErrorCode errorCode);
    public VerifierSdkException(VerifierSdkErrorCode errorCode, String message);
}
```

### Property

| Parameter | Type | Description | **M/O** | **Note** |
|-----------|------|-------------|---------|----------|
| errorCode | VerifierSdkErrorCode | Error code | M | [Link](#317-verifiersdkerrorcode) |

All SDK errors are thrown as this unchecked exception. The first constructor uses the default message of the error code, the second the given message. Provider implementations can throw it as well; how the SDK handles exceptions from each provider method is described in [2. Interfaces](#2-interfaces).

<br>

## 3.17 VerifierSdkErrorCode

### Declaration

```java
@Getter
public enum VerifierSdkErrorCode {
    // SDK_INVALID_DID, SDK_INVALID_PROOF, ..., SDK_UNKNOWN_ERROR

    private final String code;
    private final String message;
    private final int httpStatus;

    public String getCode();
    public String getMessage();
    public int getHttpStatus();
    public static String getMessageByCode(String code);
}
```

### Property

| Parameter | Type | Description | **M/O** | **Note** |
|-----------|------|-------------|---------|----------|
| code | String | Error code | M | `SSDKVRF` followed by six digits |
| message | String | Default error message | M | |
| httpStatus | int | Suggested HTTP status | M | e.g. 400, 404, 500 |

`getMessageByCode` returns the message of the given code string, or `Unknown error code: <code>` if no constant matches. The full list of codes is in [Verifier SDK Error Codes](../errorCode/VerifierSDKError.md).

