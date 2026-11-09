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

| 버전 | 날짜 | 변경 내용 |
|------|------|---------|
| v1.0.0 | 2026-10-20 | 최초 작성 |
| v3.0.0 | 2026-11-09 | 공개 API 전체 문서화, 시그니처 정정 |


<div style="page-break-after: always;"></div>

# 목차
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
    - [1.8 구현 클래스](#18-구현-클래스)
- [2. 인터페이스](#2-인터페이스)
    - [2.1 VerificationConfigProvider](#21-verificationconfigprovider)
    - [2.2 VerifierInfoProvider](#22-verifierinfoprovider)
    - [2.3 EcdhSessionProvider](#23-ecdhsessionprovider)
    - [2.4 StorageProvider](#24-storageprovider)
    - [2.5 TransactionProvider](#25-transactionprovider)
    - [2.6 NonceGenerator](#26-noncegenerator)
    - [2.7 CryptoHelper](#27-cryptohelper)
- [3. 데이터 클래스](#3-데이터-클래스)
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

## 개요

이 문서는 OpenDID 생태계에서 **VP(Verifiable Presentation) 및 ZKP(Zero-Knowledge Proof) 검증** 프로토콜을 구현하기 위한 Server SDK API를 정의합니다.

### 주요 기능
- Dynamic/Static QR 코드 흐름을 위한 VP Offer 생성
- E2E 키 교환이 포함된 VerifyProfile 생성
- VP 검증 파이프라인: E2E 복호화 → 서명 검증 → VC 상태 확인
- 클레임 추출 및 검증 결과 생성
- ZKP 익명 증명 요청 및 검증

<br>

<div style="page-break-after: always;"></div>

# 1. APIs

## 1.1 VerifierService

`VerifierService` 인터페이스는 SDK Facade입니다. `VerifierSdkBuilder`를 통해 인스턴스를 획득합니다. 7개 Provider는 모두 필수이며, 하나라도 누락되면 `build()`에서 `VerifierSdkException`(`SDK_CONFIGURATION_ERROR`)이 발생합니다. Facade는 각 호출을 5개 프로토콜 인터페이스 중 하나에 위임하며, 이 인터페이스는 [1.1.11](#1111-getofferservice)–[1.1.15](#1115-getzkpproofverificationprotocol)로 얻을 수 있습니다. SDK는 오류를 `VerifierSdkException`([3.16](#316-verifiersdkexception))으로 알리며, Provider 구현체가 던진 예외의 처리 방식은 각 메서드와 [2. 인터페이스](#2-인터페이스)에 설명되어 있습니다.

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
`Dynamic QR 코드 흐름을 위한 VP Offer Payload를 생성합니다. 반환된 페이로드에는 Holder의 응답과 매칭하기 위한 offerId(TransactionProvider.createTransactionId로 생성)와, 현재 시각에 정책의 validityDuration(초)을 더한 validUntil이 포함됩니다.`

- 먼저 `VerificationConfigProvider.getPolicy(policyId)`를 호출합니다. `null`을 반환하면 `VerifierSdkException`(`SDK_POLICY_NOT_FOUND`)이 발생하며, Provider가 던진 예외는 그대로 전달됩니다.
- 이어서 `TransactionProvider.createTransactionId()`를 한 번 호출하고, 반환값을 `null` 확인 없이 `offerId`로 사용합니다.
- `mode`는 정책의 `mode`를 `PresentMode.fromDisplayName`으로 변환한 값입니다. `Direct`, `Indirect`, `Proxy` 외의 값이면 `IllegalArgumentException`이 발생합니다.
- `endpoints`는 정책에서 복사하며 `type`은 항상 `VerifyOffer`입니다. `validUntil`은 ISO-8601 UTC 문자열(`Instant.toString()`)입니다.
- SDK는 Offer를 저장하지 않습니다. `offerId`와 트랜잭션 상태는 애플리케이션이 관리해야 합니다.

### Input Parameters

| Parameter | Type | Description | **M/O** | **Note** |
|-----------|------|-------------|---------|----------|
| policyId | String | 검증 정책 식별자 | M | |
| device | String | 응대장치 식별자 | M | |
| service | String | 서비스 식별자 | M | |
| locked | boolean | Offer 잠김 여부 | M | `true`: passcode 필요 |

### Output Parameters

| Type | Description | **M/O** | **Note** |
|------|-------------|---------|----------|
| VpOfferPayload | VP Offer 페이로드 | M | offerId 포함. [Link](#31-vpofferpayload) |

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
String offerId = offer.getOfferId(); // QR 코드에 포함
```

<br>

### 1.1.2 requestStaticVpOffer

### Class Name
`VerifierService`

### Function Name
`requestStaticVpOffer`

### Function Introduction
`Static QR 코드 흐름을 위한 VP Offer Payload를 생성합니다. offerId와 validUntil이 포함되지 않으므로 동일한 QR 코드를 여러 세션에 재사용할 수 있습니다.`

- 정책 조회, `mode`, `endpoints`, `type`은 [1.1.1](#111-requestvpoffer)과 같이 처리합니다. `TransactionProvider`는 호출하지 않습니다.

### Input Parameters

| Parameter | Type | Description | **M/O** | **Note** |
|-----------|------|-------------|---------|----------|
| policyId | String | 검증 정책 식별자 | M | |
| device | String | 응대장치 식별자 | M | |
| service | String | 서비스 식별자 | M | |
| locked | boolean | Offer 잠김 여부 | M | `true`: passcode 필요 |

### Output Parameters

| Type | Description | **M/O** | **Note** |
|------|-------------|---------|----------|
| VpOfferPayload | VP Offer 페이로드 | M | offerId와 validUntil은 null. [Link](#31-vpofferpayload) |

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
`주어진 정책에 대한 검증 요구사항을 기술하는 VerifyProfile을 생성합니다. SDK는 NonceGenerator로 verifierNonce를 생성하여 reqE2e와 함께 프로파일의 process에 넣습니다. 호출 애플리케이션은 반환된 프로파일에 서명을 추가한 뒤 Holder에게 전달해야 합니다.`

- `VerificationConfigProvider.getPolicy(policyId)`가 `null`을 반환하면 `VerifierSdkException`(`SDK_POLICY_NOT_FOUND`)이 발생합니다. `VerificationConfigProvider`, `VerifierInfoProvider`, `NonceGenerator`가 던진 예외는 그대로 전달됩니다.
- 프로파일은 다음과 같이 채워집니다: `type` = `VerifyProfile`, `title` = 정책 `policyName`, `description` = 정책 `description`(`null`이면 빈 문자열), `encoding` = `UTF-8`, `language` = 정책 `language`, `profile.verifier` = `VerifierInfoProvider.getVerifierInfo()`, `profile.filter` = 정책 `filter`, `profile.process` = 정책 `process`의 `endpoints`, `authType`에 `reqE2e`와 `verifierNonce`를 더한 값.
- `reqE2e`는 필드 단위로 프로파일에 복사되며 `null`이면 안 됩니다(`NullPointerException`). 이 흐름에서 SDK는 E2E 세션을 생성하거나 저장하지 않으므로, [verifyPresentation](#114-verifypresentation)이 VP를 복호화할 수 있도록 애플리케이션이 `txId`의 개인키를 보관해야 합니다(예: `EcdhSessionProvider.createSession`).
- `profile.getProfile().getProcess().getVerifierNonce()`를 저장해 두었다가 VP 검증 시 `VpVerificationRequest.verifierNonce`로 전달합니다.

### Input Parameters

| Parameter | Type | Description | **M/O** | **Note** |
|-----------|------|-------------|---------|----------|
| policyId | String | 검증 정책 식별자 | M | |
| profileId | String | 고유 프로파일 식별자 | M | UUID 권장 |
| reqE2e | ReqE2e | Verifier의 E2E 키 교환 정보 | M | 예: `EcdhSessionProvider.createSession(txId)` 결과. [Link](#35-reqe2e) |

### Output Parameters

| Type | Description | **M/O** | **Note** |
|------|-------------|---------|----------|
| VerifyProfile | Proof가 없는 검증 프로파일 | M | 애플리케이션에서 Proof 추가 필요 |

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
// Verifier의 E2E 키 교환 정보 (nonce, curve, publicKey, cipher, padding)
ReqE2e reqE2e = sessionProvider.createSession(txId);

VerifyProfile profile = verifierService.requestVerifyProfile(
    "policy-login",
    UUID.randomUUID().toString(),
    reqE2e
);
String verifierNonce = profile.getProfile().getProcess().getVerifierNonce(); // verifyPresentation에서 사용하도록 보관
// 프로파일에 서명하여 Proof를 설정(profile.setProof(...))한 뒤 Holder에 반환
```

<br>

### 1.1.4 verifyPresentation

### Class Name
`VerifierService`

### Function Name
`verifyPresentation`

### Function Introduction
`VP 검증 전체 파이프라인(E2E 복호화 → 인증 타입 확인 → Nonce 확인 → VpManager 검증(VP/VC 서명, 스키마, 필터, 유효기간, Holder-Subject 일치) → VC 상태 확인)을 수행합니다. VP는 항상 txId의 E2E 세션으로 복호화되며, 평문(암호화되지 않은) VP 제출은 지원하지 않습니다.`

- 복호화: [decryptVp](#115-decryptvp)와 같습니다(`SDK_E2E_SESSION_NOT_FOUND`, `SDK_DECRYPTION_FAILED`).
- 인증 타입: `requiredAuthType`이 `0x00000006`(PIN 또는 BIO)인 경우에만 VP proof의 `verificationMethod`에서 `#` 뒤 값이 `pin` 또는 `bio`(대소문자 무시)여야 합니다. proof 또는 `verificationMethod`가 없으면 `SDK_INVALID_PROOF`, 그 밖의 값이면 `SDK_INVALID_AUTH_TYPE`이 발생합니다. [1.5.3](#153-validateauthtype) 참조.
- Nonce: VP에 `verifierNonce`가 없거나 `request.verifierNonce`와 다르면 `SDK_INVALID_NONCE`가 발생합니다.
- VpManager: Holder DID와 각 VC의 Issuer DID에 대해 `StorageProvider.findDidDocument`를 호출하며, `null`이면 `SDK_DID_DOCUMENT_NOT_FOUND`가 발생합니다. VC가 없는 VP는 `SDK_INVALID_VP`, VpManager 검증 오류는 `SDK_SIGNATURE_VERIFICATION_FAILED`로 발생합니다.
- VC 상태: [1.5.5](#155-validatevcstatuses)와 같이 `StorageProvider.getVcMeta`로 확인하며, 폐기·비활성·만료된 VC는 `SDK_INVALID_VC`로 거부됩니다.
- `StorageProvider`가 던진 예외는 그대로 전달됩니다. 성공하면 복호화된 VP JSON을 그대로 반환하며, SDK는 E2E 세션을 삭제하거나 결과를 저장하지 않습니다.

### Input Parameters

| Parameter | Type | Description | **M/O** | **Note** |
|-----------|------|-------------|---------|----------|
| request | VpVerificationRequest | VP 검증 요청 | M | [Link](#32-vpverificationrequest) |

### Output Parameters

| Type | Description | **M/O** | **Note** |
|------|-------------|---------|----------|
| String | 검증된 VP JSON 문자열 | M | |

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
`전체 검증 흐름과 독립적으로 E2E 암호화된 VP를 복호화합니다. 복호화와 검증을 별도 단계로 처리할 때 사용합니다.`

- `EcdhSessionProvider.existsSession(txId)`가 `false`를 반환하면 `VerifierSdkException`(`SDK_E2E_SESSION_NOT_FOUND`)이 발생합니다.
- 그렇지 않으면 `EcdhSessionProvider.decrypt(txId, encHolderPublicKey, encVp, iv)`를 호출해 반환값을 그대로 반환합니다. `decrypt`가 던진 예외는 모두 `SDK_DECRYPTION_FAILED`로 변환됩니다.
- VP를 파싱하거나 검증하지 않습니다.

### Input Parameters

| Parameter | Type | Description | **M/O** | **Note** |
|-----------|------|-------------|---------|----------|
| txId | String | E2E 세션 조회용 트랜잭션 ID | M | |
| encHolderPublicKey | String | Holder의 E2E 공개키 | M | Multibase 인코딩 (암호화되지 않음) |
| encVp | String | E2E 암호화된 VP | M | Multibase 인코딩 |
| iv | String | 초기화 벡터 | M | Multibase 인코딩 |

### Output Parameters

| Type | Description | **M/O** | **Note** |
|------|-------------|---------|----------|
| String | 복호화된 VP JSON 문자열 | M | |

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
`검증된 VP에서 Holder DID, 제출된 VC, 클레임을 추출하여 검증 결과를 반환합니다(SDK는 결과를 저장하지 않음). verified가 false이면 VP를 파싱하지 않고 실패 결과를 반환합니다. 표준 VP 검증 흐름에서 사용하는 오버로드입니다.`

- `offerType`을 `VerifyOffer`로 지정한 [1.1.7](#117-confirmverification-vp-or-proof)과 같습니다.
- `vpJson`은 Gson으로 파싱하며, 파싱 오류는 `VerifierSdkException`으로 변환되지 않습니다. VP에 `holder`가 없으면 `SDK_INVALID_VP`가 발생합니다.
- `submittedVcs`에는 `verifiableCredential` 중 `id`가 있는 VC가 담기고([1.6.5](#165-extractsubmittedvcsasmap)), `extractedClaims`에는 모든 VC의 `credentialSubject.claims[].code` → `value`가 담깁니다([1.6.6](#166-extractclaims)).
- `verified`가 `false`이면 `verified=false`, `errorMessage` = `Verification failed`인 결과를 반환합니다.

### Input Parameters

| Parameter | Type | Description | **M/O** | **Note** |
|-----------|------|-------------|---------|----------|
| txId | String | 트랜잭션 ID | M | |
| vpJson | String | 검증된 VP JSON 문자열 | M | |
| verified | boolean | 검증 성공 여부 | M | |

### Output Parameters

| Type | Description | **M/O** | **Note** |
|------|-------------|---------|----------|
| VerificationConfirmResult | 검증 확인 결과 | M | [Link](#33-verificationconfirmresult) |

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
`Offer 타입에 따라 검증된 VP 또는 ZKP Proof의 검증 결과를 반환합니다(SDK는 결과를 저장하지 않음). VerifyProofOffer인 경우 vpOrProofJson을 파싱하지 않으며, extractedClaims에는 {"zkpVerificationResult": "Successful"}만 포함됩니다. Offer 타입을 명시적으로 지정해야 할 때 사용하는 오버로드입니다.`

- `verified`가 `false`이면 `offerType`과 관계없이 실패 결과(`verified=false`, `errorMessage` = `Verification failed`)를 반환합니다.
- `VerifyProofOffer`: `vpOrProofJson`을 사용하지 않습니다. 결과는 `verified=true`이며 `holderDid`와 `submittedVcs`는 설정되지 않습니다.
- 그 밖의 `offerType`(`null` 포함): [1.1.6](#116-confirmverification-vp)과 같이 VP로 처리합니다.

### Input Parameters

| Parameter | Type | Description | **M/O** | **Note** |
|-----------|------|-------------|---------|----------|
| txId | String | 트랜잭션 ID | M | |
| vpOrProofJson | String | 검증된 VP 또는 Proof JSON 문자열 | M | |
| verified | boolean | 검증 성공 여부 | M | |
| offerType | OfferType | Offer 타입 | M | `VerifyProofOffer`: ZKP Proof, 그 외 값: VP |

### Output Parameters

| Type | Description | **M/O** | **Note** |
|------|-------------|---------|----------|
| VerificationConfirmResult | 검증 확인 결과 | M | [Link](#33-verificationconfirmresult) |

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
`ZKP 검증 요청을 정의하는 ProofRequestProfile을 생성합니다. SDK는 ProofRequest의 Nonce와 E2E 키쌍을 생성하고, EcdhSessionProvider.saveSession으로 E2E 세션을 저장합니다. 반환된 프로파일에는 Proof가 포함되지 않으므로, 호출 애플리케이션이 서명하여 Proof를 설정한 뒤 Holder에게 전달해야 합니다.`

- ProofRequest는 정책으로 구성합니다. `name`과 `version`은 그대로 복사하고, `requestedAttributes`와 `requestedPredicates`(JSON 문자열)는 ProofRequest로 파싱하며(`null`이거나 비어 있으면 생략), `nonce`는 SDK가 생성한 큰 난수 정수입니다.
- E2E 키쌍은 `CryptoHelper.generateEcKeyPair(request.reqE2e.curve)`로, E2E Nonce는 `CryptoHelper.generateNonce(16)`으로 생성합니다. 프로파일의 `reqE2e`는 요청의 `curve`, `cipher`, `padding`과 이 호출로 생성된 `publicKey`, `nonce`로 구성됩니다.
- `type`은 `VerifyProfile`이고, `title`, `description`, `language`는 정책에서, `profile.verifier`는 `VerifierInfoProvider.getVerifierInfo()`에서 가져옵니다.
- 마지막으로 `EcdhSessionProvider.saveSession(request.txId, keyPair, reqE2e)`를 호출합니다.
- 정책의 `curve`, `cipher`, `padding`과 요청의 `policyId`, `verifierDid`는 사용하지 않습니다.
- Provider가 던진 예외와 정책의 잘못된 JSON을 포함해 생성 중 발생한 모든 예외는 `VerifierSdkException`(`SDK_UNKNOWN_ERROR`)으로 변환됩니다.

### Input Parameters

| Parameter | Type | Description | **M/O** | **Note** |
|-----------|------|-------------|---------|----------|
| request | ProofRequestProfileRequest | 프로파일 요청 메타데이터 | M | `txId`, `profileId`, `reqE2e`(`curve`, `cipher`, `padding`) 사용 |
| policy | ZkpPolicy | ZKP 정책 설정 | M | `title`, `description`, `language`, `name`, `version`, `requestedAttributes`, `requestedPredicates` 사용 |

### Output Parameters

| Type | Description | **M/O** | **Note** |
|------|-------------|---------|----------|
| ProofRequestProfile | ZKP 증명 요청 프로파일 | M | Proof 미포함. [verifyZkpProof](#119-verifyzkpproof)를 위해 `getProfile().getProofRequest()`를 저장 |

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
    .title("연령 확인")
    .language("ko")
    .name("age-check")
    .version("1.0")
    .requestedPredicates(requestedPredicatesJson)
    .build();

ProofRequestProfile profile = verifierService.requestZkpProofRequestProfile(request, policy);
ProofRequest proofRequest = profile.getProfile().getProofRequest(); // verifyZkpProof에서 사용하도록 보관
// 프로파일에 서명하여 Proof를 설정(profile.setProof(...))한 뒤 Holder에 전달
```

<br>

### 1.1.9 verifyZkpProof

### Class Name
`VerifierService`

### Function Name
`verifyZkpProof`

### Function Introduction
`제출된 ZKP Proof를 복호화하고, 제출된 Nonce를 저장된 proofRequest의 Nonce와 비교한 뒤, 저장된 Nonce로 ZKP 라이브러리를 통해 Proof를 검증합니다.`

- `request.proofRequest`는 [1.1.8](#118-requestzkpproofrequestprofile)에서 생성한 ProofRequestProfile로부터 Verifier가 저장한 ProofRequest여야 하며, Holder가 제출한 값을 사용하면 안 됩니다.
- `request.nonce`는 `proofRequest`의 Nonce와 같아야 합니다. 둘 중 하나가 없거나, `request.nonce`가 숫자가 아니거나, 값이 다르면 `VerifierSdkException`(`SDK_INVALID_NONCE`)이 발생합니다. 검증에는 저장된 Nonce가 사용됩니다.
- identifiers가 없는 Proof는 `VerifierSdkException`(`SDK_INVALID_PROOF`)으로 거부됩니다.
- ZKP 라이브러리가 Proof를 유효하지 않다고 판단하면 예외 대신 `verified=false`와 `errorMessage`가 설정된 결과를 반환합니다.
- 각 identifier의 Credential Schema와 Credential Definition은 `StorageProvider.getZKPCredential` / `getZKPCredentialDefinition`으로 조회합니다.
- 복호화 오류는 `SDK_E2E_SESSION_NOT_FOUND` / `SDK_DECRYPTION_FAILED`로, 그 밖의 예기치 않은 오류는 `SDK_SIGNATURE_VERIFICATION_FAILED`로 발생합니다.
- SDK는 E2E 세션을 삭제하지 않습니다.

### Input Parameters

| Parameter | Type | Description | **M/O** | **Note** |
|-----------|------|-------------|---------|----------|
| request | ZkpVerificationRequest | ZKP 검증 요청 | M | [Link](#36-zkpverificationrequest) |

### Output Parameters

| Type | Description | **M/O** | **Note** |
|------|-------------|---------|----------|
| ZkpVerificationResult | ZKP 검증 결과 | M | [Link](#34-zkpverificationresult) |

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
    .nonce(submittedNonce)             // Holder가 제출한 Nonce
    .proofRequest(storedProofRequest)  // Verifier가 저장한 ProofRequest
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
`전체 검증 흐름과 독립적으로 E2E 암호화된 ZKP Proof를 복호화합니다.`

- `EcdhSessionProvider.existsSession(txId)`가 `false`를 반환하면 `VerifierSdkException`(`SDK_E2E_SESSION_NOT_FOUND`)이 발생합니다.
- 그렇지 않으면 `EcdhSessionProvider.decrypt(txId, accE2e.publicKey, encProof, iv)`를 호출하고 결과를 Gson으로 `Proof`에 파싱합니다. 이 단계의 모든 예외(`accE2e`가 `null`인 경우 포함)는 `SDK_DECRYPTION_FAILED`로 변환됩니다.
- Nonce와 Proof는 검증하지 않습니다.

### Input Parameters

| Parameter | Type | Description | **M/O** | **Note** |
|-----------|------|-------------|---------|----------|
| txId | String | E2E 세션 조회용 트랜잭션 ID | M | |
| encProof | String | 암호화된 Proof | M | Multibase 인코딩 |
| iv | String | 초기화 벡터 | M | Multibase 인코딩 |
| accE2e | ZkpVerificationRequest.AccE2e | Holder의 E2E 수락 정보 | M | 복호화에 `accE2e.publicKey` 사용. [Link](#36-zkpverificationrequest) |

### Output Parameters

| Type | Description | **M/O** | **Note** |
|------|-------------|---------|----------|
| Proof | 복호화된 Proof 객체 | M | `org.omnione.did.zkp.datamodel.proof.Proof` |

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
`requestVpOffer와 requestStaticVpOffer가 위임되는 VpOfferProtocol을 반환합니다. 항상 같은 인스턴스를 반환하며, VerifierSdkBuilder로 생성한 VerifierService에서는 VpOfferProtocolImpl입니다.`

### Input Parameters
n/a

### Output Parameters

| Type | Description | **M/O** | **Note** |
|------|-------------|---------|----------|
| VpOfferProtocol | VP Offer 프로토콜 | M | [Link](#13-vpofferprotocol) |

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
`requestVerifyProfile이 위임되는 VpProfileProtocol을 반환합니다. 항상 같은 인스턴스를 반환하며, VerifierSdkBuilder로 생성한 VerifierService에서는 VpProfileProtocolImpl입니다.`

### Input Parameters
n/a

### Output Parameters

| Type | Description | **M/O** | **Note** |
|------|-------------|---------|----------|
| VpProfileProtocol | VerifyProfile 프로토콜 | M | [Link](#14-vpprofileprotocol) |

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
`verifyPresentation과 decryptVp가 위임되는 VpVerificationProtocol을 반환합니다. 항상 같은 인스턴스를 반환하며, VerifierSdkBuilder로 생성한 VerifierService에서는 VpVerificationProtocolImpl입니다.`

### Input Parameters
n/a

### Output Parameters

| Type | Description | **M/O** | **Note** |
|------|-------------|---------|----------|
| VpVerificationProtocol | VP 검증 프로토콜 | M | [Link](#15-vpverificationprotocol) |

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
`두 confirmVerification 메서드가 위임되는 VerificationConfirmProtocol을 반환합니다. 항상 같은 인스턴스를 반환하며, VerifierSdkBuilder로 생성한 VerifierService에서는 VerificationConfirmProtocolImpl입니다.`

### Input Parameters
n/a

### Output Parameters

| Type | Description | **M/O** | **Note** |
|------|-------------|---------|----------|
| VerificationConfirmProtocol | 검증 확인 프로토콜 | M | [Link](#16-verificationconfirmprotocol) |

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
`requestZkpProofRequestProfile, verifyZkpProof, decryptZkpProof가 위임되는 ZkpProofVerificationProtocol을 반환합니다. 항상 같은 인스턴스를 반환하며, VerifierSdkBuilder로 생성한 VerifierService에서는 ZkpProofVerificationProtocolImpl입니다.`

### Input Parameters
n/a

### Output Parameters

| Type | Description | **M/O** | **Note** |
|------|-------------|---------|----------|
| ZkpProofVerificationProtocol | ZKP Proof 검증 프로토콜 | M | [Link](#17-zkpproofverificationprotocol) |

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

`VerifierSdkBuilder`(`org.omnione.did.verifier.v1.config`)는 `VerifierService`를 생성합니다. 아래 메서드로 7개 Provider를 설정한 뒤 [build](#121-build)를 호출합니다. 각 메서드는 전달받은 Provider를 저장하고 같은 빌더를 반환하며 `null` 여부를 확인하지 않으므로, 누락된 Provider는 `build()`에서 확인됩니다. 같은 메서드를 다시 호출하면 Provider가 교체됩니다.

| Method | Type | Description | **M/O** |
|--------|------|-------------|---------|
| `VerifierSdkBuilder configProvider(VerificationConfigProvider configProvider)` | VerificationConfigProvider | 정책 제공자. [Link](#21-verificationconfigprovider) | M |
| `VerifierSdkBuilder verifierInfoProvider(VerifierInfoProvider verifierInfoProvider)` | VerifierInfoProvider | Verifier 정보 제공자. [Link](#22-verifierinfoprovider) | M |
| `VerifierSdkBuilder sessionProvider(EcdhSessionProvider sessionProvider)` | EcdhSessionProvider | E2E 세션 제공자. [Link](#23-ecdhsessionprovider) | M |
| `VerifierSdkBuilder storageProvider(StorageProvider storageProvider)` | StorageProvider | DID Document, VC 메타데이터, ZKP 데이터 제공자. [Link](#24-storageprovider) | M |
| `VerifierSdkBuilder transactionProvider(TransactionProvider transactionProvider)` | TransactionProvider | 트랜잭션 ID 제공자. [Link](#25-transactionprovider) | M |
| `VerifierSdkBuilder nonceGenerator(NonceGenerator nonceGenerator)` | NonceGenerator | Verifier Nonce 생성기. [Link](#26-noncegenerator) | M |
| `VerifierSdkBuilder cryptoHelper(CryptoHelper cryptoHelper)` | CryptoHelper | 암호화 헬퍼. [Link](#27-cryptohelper) | M |

<br>

### 1.2.1 build

### Class Name
`VerifierSdkBuilder`

### Function Name
`build`

### Function Introduction
`7개 Provider가 모두 설정되었는지 확인하고, 기본 프로토콜 구현체로 구성된 VerifierService(VerifierServiceImpl)을 생성합니다. build 중에는 Provider를 호출하지 않으며, 호출할 때마다 새 인스턴스를 생성합니다.`

- Provider는 `configProvider`, `verifierInfoProvider`, `sessionProvider`, `storageProvider`, `transactionProvider`, `nonceGenerator`, `cryptoHelper` 순으로 확인하며, 처음 누락된 Provider에서 `<인터페이스 이름> is required.` 메시지(예: `StorageProvider is required.`)와 함께 `VerifierSdkException`(`SDK_CONFIGURATION_ERROR`)이 발생합니다.

### Input Parameters
n/a

### Output Parameters

| Type | Description | **M/O** | **Note** |
|------|-------------|---------|----------|
| VerifierService | SDK Facade | M | [Link](#11-verifierservice) |

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

`VpOfferProtocol`은 VP Offer 페이로드를 생성합니다. [getOfferService](#1111-getofferservice)로 얻으며, 같은 이름의 `VerifierService` 메서드가 이 인터페이스에 위임합니다. 기본 구현체는 `VpOfferProtocolImpl`입니다([1.8](#18-구현-클래스)).

<br>

### 1.3.1 requestVpOffer

### Class Name
`VpOfferProtocol`

### Function Name
`requestVpOffer`

### Function Introduction
`Dynamic QR 코드 흐름을 위한 VP Offer Payload를 생성합니다. offerId는 TransactionProvider.createTransactionId로 생성하고, validUntil은 정책의 validityDuration으로 계산합니다.`

- 동작과 예외는 [1.1.1 requestVpOffer](#111-requestvpoffer)와 같습니다. 정책이 없으면 `VerifierSdkException`(`SDK_POLICY_NOT_FOUND`)이 발생합니다.

### Input Parameters

| Parameter | Type | Description | **M/O** | **Note** |
|-----------|------|-------------|---------|----------|
| policyId | String | 검증 정책 식별자 | M | |
| device | String | 응대장치 식별자 | M | |
| service | String | 서비스 식별자 | M | |
| locked | boolean | Offer 잠김 여부 | M | `true`: passcode 필요 |

### Output Parameters

| Type | Description | **M/O** | **Note** |
|------|-------------|---------|----------|
| VpOfferPayload | VP Offer 페이로드 | M | offerId 포함. [Link](#31-vpofferpayload) |

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
`Static QR 코드 흐름을 위한 VP Offer Payload를 생성합니다. offerId와 validUntil은 포함되지 않습니다.`

- 동작과 예외는 [1.1.2 requestStaticVpOffer](#112-requeststaticvpoffer)와 같습니다.

### Input Parameters

| Parameter | Type | Description | **M/O** | **Note** |
|-----------|------|-------------|---------|----------|
| policyId | String | 검증 정책 식별자 | M | |
| device | String | 응대장치 식별자 | M | |
| service | String | 서비스 식별자 | M | |
| locked | boolean | Offer 잠김 여부 | M | `true`: passcode 필요 |

### Output Parameters

| Type | Description | **M/O** | **Note** |
|------|-------------|---------|----------|
| VpOfferPayload | VP Offer 페이로드 | M | offerId와 validUntil은 null. [Link](#31-vpofferpayload) |

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

`VpProfileProtocol`은 VP 흐름의 VerifyProfile을 생성합니다. [getProfileService](#1112-getprofileservice)로 얻으며, `VerifierService.requestVerifyProfile`이 이 인터페이스에 위임합니다. 기본 구현체는 `VpProfileProtocolImpl`입니다([1.8](#18-구현-클래스)).

<br>

### 1.4.1 requestVerifyProfile

### Class Name
`VpProfileProtocol`

### Function Name
`requestVerifyProfile`

### Function Introduction
`주어진 정책으로 Proof가 없는 VerifyProfile을 생성합니다. 프로파일의 process에는 NonceGenerator로 생성한 verifierNonce와 전달받은 reqE2e가 들어갑니다.`

- 동작과 예외는 [1.1.3 requestVerifyProfile](#113-requestverifyprofile)과 같습니다. 생성된 `verifierNonce`는 VP 검증을 위해 저장하고, 프로파일은 서명한 뒤 전달해야 합니다.

### Input Parameters

| Parameter | Type | Description | **M/O** | **Note** |
|-----------|------|-------------|---------|----------|
| policyId | String | 검증 정책 식별자 | M | |
| profileId | String | 고유 프로파일 식별자 | M | UUID 권장 |
| reqE2e | ReqE2e | Verifier의 E2E 키 교환 정보 | M | 예: `EcdhSessionProvider.createSession(txId)` 결과. [Link](#35-reqe2e) |

### Output Parameters

| Type | Description | **M/O** | **Note** |
|------|-------------|---------|----------|
| VerifyProfile | Proof가 없는 검증 프로파일 | M | 애플리케이션에서 Proof 추가 필요 |

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

`VpVerificationProtocol`은 VP를 복호화하고 검증합니다. [getVerificationService](#1113-getverificationservice)로 얻습니다. `verifyPresentation`은 전체 파이프라인을 수행하며, `validateAuthType`, `validateNonce`, `validateVcStatuses`는 그중 한 단계로서 파싱된 VP(`org.omnione.did.data.model.vp.VerifiablePresentation`)에 대해 따로 호출할 수 있습니다. 기본 구현체는 `VpVerificationProtocolImpl`입니다([1.8](#18-구현-클래스)).

<br>

### 1.5.1 verifyPresentation

### Class Name
`VpVerificationProtocol`

### Function Name
`verifyPresentation`

### Function Introduction
`VP 검증 전체 파이프라인(E2E 복호화 → 인증 타입 확인 → Nonce 확인 → VpManager 검증 → VC 상태 확인)을 수행하고 검증된 VP JSON을 반환합니다.`

- 동작과 예외는 [1.1.4 verifyPresentation](#114-verifypresentation)과 같습니다.

### Input Parameters

| Parameter | Type | Description | **M/O** | **Note** |
|-----------|------|-------------|---------|----------|
| request | VpVerificationRequest | VP 검증 요청 | M | [Link](#32-vpverificationrequest) |

### Output Parameters

| Type | Description | **M/O** | **Note** |
|------|-------------|---------|----------|
| String | 검증된 VP JSON 문자열 | M | |

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
`txId의 E2E 세션으로 E2E 암호화된 VP를 복호화하고, 검증하지 않은 VP JSON을 반환합니다.`

- 동작과 예외는 [1.1.5 decryptVp](#115-decryptvp)와 같습니다.

### Input Parameters

| Parameter | Type | Description | **M/O** | **Note** |
|-----------|------|-------------|---------|----------|
| txId | String | E2E 세션 조회용 트랜잭션 ID | M | |
| encHolderPublicKey | String | Holder의 E2E 공개키 | M | Multibase 인코딩 (암호화되지 않음) |
| encVp | String | E2E 암호화된 VP | M | Multibase 인코딩 |
| iv | String | 초기화 벡터 | M | Multibase 인코딩 |

### Output Parameters

| Type | Description | **M/O** | **Note** |
|------|-------------|---------|----------|
| String | 복호화된 VP JSON 문자열 | M | |

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
`VP의 인증 타입을 requiredAuthType과 비교합니다. verifyPresentation의 인증 타입 확인 단계입니다.`

- `requiredAuthType`이 `null` 또는 `0x00000000`이면 확인하지 않습니다.
- `0x00000006`(PIN 또는 BIO)이면 `vp.proof.verificationMethod`에서 `#` 뒤 값이 `pin` 또는 `bio`(대소문자 무시)여야 합니다. `proof` 또는 `verificationMethod`가 없으면 `VerifierSdkException`(`SDK_INVALID_PROOF`), `#` 뒤 값이 없거나 다른 값이면 `SDK_INVALID_AUTH_TYPE`이 발생합니다.
- 그 밖의 값은 SDK가 확인하지 않고 반환합니다. 다른 인증 타입은 애플리케이션이 직접 확인해야 합니다.

### Input Parameters

| Parameter | Type | Description | **M/O** | **Note** |
|-----------|------|-------------|---------|----------|
| vp | VerifiablePresentation | 파싱된 VP | M | `org.omnione.did.data.model.vp.VerifiablePresentation` |
| requiredAuthType | Integer | 요구 인증 타입 비트마스크 | O | `null`, `0x00000000`, `0x00000006` 또는 그 밖의 값 |

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
`VP의 verifierNonce가 expectedNonce와 같은지 확인합니다. verifyPresentation의 Nonce 확인 단계입니다.`

- VP의 `verifierNonce`가 `null`이거나 비어 있거나 `expectedNonce`와 다르면(문자열 완전 일치 비교) `VerifierSdkException`(`SDK_INVALID_NONCE`)이 발생합니다.
- `expectedNonce`는 `null` 여부를 확인하지 않으며, `null`이면 `NullPointerException`이 발생합니다.

### Input Parameters

| Parameter | Type | Description | **M/O** | **Note** |
|-----------|------|-------------|---------|----------|
| vp | VerifiablePresentation | 파싱된 VP | M | `org.omnione.did.data.model.vp.VerifiablePresentation` |
| expectedNonce | String | 기대하는 Verifier Nonce | M | VerifyProfile에서 저장한 `verifierNonce` |

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
`StorageProvider.getVcMeta로 VP에 포함된 각 VC의 상태를 확인합니다. verifyPresentation의 마지막 단계입니다.`

- VC가 없는 VP는 `VerifierSdkException`(`SDK_INVALID_VP`)이 발생합니다.
- `id`가 없는 VC는 건너뜁니다. 나머지 VC마다 `getVcMeta(vcId)`를 호출하며, `null`을 반환하면 해당 VC는 건너뜁니다.
- 상태는 메타데이터의 `status`, `vcStatus`, `credentialStatus.status` 순으로 읽고 대소문자를 무시하고 비교합니다. `REVOKED`, `INACTIVE`, `EXPIRED`는 `SDK_INVALID_VC`가 발생하며, `ACTIVE`, 그 밖의 값, 상태가 없는 경우는 통과합니다.
- `StorageProvider`가 던진 예외와 메타데이터 JSON 파싱 오류는 그대로 전달됩니다. 이 단계에서는 서명과 유효기간을 확인하지 않습니다.

### Input Parameters

| Parameter | Type | Description | **M/O** | **Note** |
|-----------|------|-------------|---------|----------|
| vp | VerifiablePresentation | 파싱된 VP | M | `org.omnione.did.data.model.vp.VerifiablePresentation` |

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

`VerificationConfirmProtocol`은 검증 결과를 생성하고 VP에서 Holder DID, VC, 클레임을 추출합니다. [getConfirmService](#1114-getconfirmservice)로 얻습니다. 추출 메서드는 VP를 Gson `com.google.gson.JsonObject`로 받으며 검증하지 않으므로, `verifyPresentation`으로 검증한 VP에 대해서만 호출해야 합니다. 기본 구현체는 `VerificationConfirmProtocolImpl`입니다([1.8](#18-구현-클래스)).

<br>

### 1.6.1 confirmVerification (VP)

### Class Name
`VerificationConfirmProtocol`

### Function Name
`confirmVerification`

### Function Introduction
`검증된 VP에서 Holder DID, 제출된 VC, 클레임을 추출하여 검증 결과를 반환합니다. SDK는 결과를 저장하지 않습니다.`

- 동작과 예외는 [1.1.6 confirmVerification (VP)](#116-confirmverification-vp)와 같습니다.

### Input Parameters

| Parameter | Type | Description | **M/O** | **Note** |
|-----------|------|-------------|---------|----------|
| txId | String | 트랜잭션 ID | M | |
| vpJson | String | 검증된 VP JSON 문자열 | M | |
| verified | boolean | 검증 성공 여부 | M | |

### Output Parameters

| Type | Description | **M/O** | **Note** |
|------|-------------|---------|----------|
| VerificationConfirmResult | 검증 확인 결과 | M | [Link](#33-verificationconfirmresult) |

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
`Offer 타입에 따라 검증된 VP 또는 ZKP Proof의 검증 결과를 반환합니다. SDK는 결과를 저장하지 않습니다.`

- 동작과 예외는 [1.1.7 confirmVerification (VP or Proof)](#117-confirmverification-vp-or-proof)와 같습니다.

### Input Parameters

| Parameter | Type | Description | **M/O** | **Note** |
|-----------|------|-------------|---------|----------|
| txId | String | 트랜잭션 ID | M | |
| vpOrProofJson | String | 검증된 VP 또는 Proof JSON 문자열 | M | |
| verified | boolean | 검증 성공 여부 | M | |
| offerType | OfferType | Offer 타입 | M | `VerifyProofOffer`: ZKP Proof, 그 외 값: VP |

### Output Parameters

| Type | Description | **M/O** | **Note** |
|------|-------------|---------|----------|
| VerificationConfirmResult | 검증 확인 결과 | M | [Link](#33-verificationconfirmresult) |

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
`VP의 Holder DID(holder 속성)를 반환합니다.`

- VP에 `holder`가 없으면 `VerifierSdkException`(`SDK_INVALID_VP`)이 발생합니다. 값은 문자열로 읽습니다.

### Input Parameters

| Parameter | Type | Description | **M/O** | **Note** |
|-----------|------|-------------|---------|----------|
| vpObject | JsonObject | VP JSON 객체 | M | `com.google.gson.JsonObject` |

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
`VP의 verifiableCredential 배열에 있는 VC의 ID를 배열 순서대로 반환합니다.`

- `id`가 없는 VC는 건너뜁니다. VP에 `verifiableCredential`이 없으면 빈 목록을 반환합니다.

### Input Parameters

| Parameter | Type | Description | **M/O** | **Note** |
|-----------|------|-------------|---------|----------|
| vpObject | JsonObject | VP JSON 객체 | M | `com.google.gson.JsonObject` |

### Output Parameters

| Type | Description | **M/O** | **Note** |
|------|-------------|---------|----------|
| List\<String\> | VC ID 목록 | M | VC가 없으면 빈 목록 |

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
`VP의 VC를 VC ID → VC 객체 맵으로 반환합니다.`

- 각 VC는 Gson으로 JSON에서 `Map`으로 변환됩니다(JSON 숫자는 `Double`). `id`가 없는 VC는 건너뜁니다. VP에 `verifiableCredential`이 없으면 빈 맵을 반환합니다.

### Input Parameters

| Parameter | Type | Description | **M/O** | **Note** |
|-----------|------|-------------|---------|----------|
| vpObject | JsonObject | VP JSON 객체 | M | `com.google.gson.JsonObject` |

### Output Parameters

| Type | Description | **M/O** | **Note** |
|------|-------------|---------|----------|
| Map\<String, Object\> | VC ID → VC 객체 | M | VC가 없으면 빈 맵 |

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
`VP에 포함된 모든 VC의 클레임을 클레임 코드 → 클레임 값 맵으로 추출합니다.`

- 각 VC의 `credentialSubject.claims`를 읽어 `code`와 `value`가 모두 있는 항목을 추가합니다. 값은 문자열로 읽으며, JSON 원시값이 아니면 Gson에서 예외가 발생합니다.
- 같은 코드가 여러 VC에 있으면 뒤쪽 VC의 값이 남습니다. VP에 `verifiableCredential`이 없으면 빈 맵을 반환합니다.

### Input Parameters

| Parameter | Type | Description | **M/O** | **Note** |
|-----------|------|-------------|---------|----------|
| vpObject | JsonObject | VP JSON 객체 | M | `com.google.gson.JsonObject` |

### Output Parameters

| Type | Description | **M/O** | **Note** |
|------|-------------|---------|----------|
| Map\<String, Object\> | 클레임 코드 → 값 | M | 값은 `String` |

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
`extractClaims와 같은 방식으로 클레임을 추출한 뒤 requiredClaims에 있는 코드만 반환합니다.`

- VP에 없는 코드는 오류 없이 제외됩니다. `requiredClaims`는 `null`이면 안 됩니다(`NullPointerException`).

### Input Parameters

| Parameter | Type | Description | **M/O** | **Note** |
|-----------|------|-------------|---------|----------|
| vpObject | JsonObject | VP JSON 객체 | M | `com.google.gson.JsonObject` |
| requiredClaims | List\<String\> | 반환할 클레임 코드 목록 | M | |

### Output Parameters

| Type | Description | **M/O** | **Note** |
|------|-------------|---------|----------|
| Map\<String, Object\> | 클레임 코드 → 값 | M | `requiredClaims`에 있는 코드만 포함 |

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

`ZkpProofVerificationProtocol`은 ZKP ProofRequestProfile을 생성하고 ZKP Proof를 복호화·검증합니다. [getZkpProofVerificationProtocol](#1115-getzkpproofverificationprotocol)로 얻습니다. 기본 구현체는 `ZkpProofVerificationProtocolImpl`입니다([1.8](#18-구현-클래스)).

<br>

### 1.7.1 requestZkpProofRequestProfile

### Class Name
`ZkpProofVerificationProtocol`

### Function Name
`requestZkpProofRequestProfile`

### Function Introduction
`ProofRequest Nonce와 E2E 키쌍을 생성하여 Proof가 없는 ProofRequestProfile을 만들고, EcdhSessionProvider.saveSession으로 E2E 세션을 저장합니다.`

- 동작과 예외는 [1.1.8 requestZkpProofRequestProfile](#118-requestzkpproofrequestprofile)과 같습니다. 모든 오류는 `VerifierSdkException`(`SDK_UNKNOWN_ERROR`)으로 발생합니다.

### Input Parameters

| Parameter | Type | Description | **M/O** | **Note** |
|-----------|------|-------------|---------|----------|
| request | ProofRequestProfileRequest | 프로파일 요청 메타데이터 | M | `txId`, `profileId`, `reqE2e`(`curve`, `cipher`, `padding`) 사용. [Link](#310-proofrequestprofilerequest) |
| policy | ZkpPolicy | ZKP 정책 설정 | M | `title`, `description`, `language`, `name`, `version`, `requestedAttributes`, `requestedPredicates` 사용. [Link](#311-zkppolicy) |

### Output Parameters

| Type | Description | **M/O** | **Note** |
|------|-------------|---------|----------|
| ProofRequestProfile | ZKP 증명 요청 프로파일 | M | Proof 미포함. [Link](#38-proofrequestprofile) |

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
`제출된 ZKP Proof를 복호화하고, Nonce를 저장된 proofRequest와 비교한 뒤 ZKP 라이브러리로 Proof를 검증합니다.`

- 동작과 예외는 [1.1.9 verifyZkpProof](#119-verifyzkpproof)와 같습니다. 유효하지 않은 Proof는 `verified=false` 결과를 반환하고, 요청 오류는 `VerifierSdkException`으로 발생합니다.

### Input Parameters

| Parameter | Type | Description | **M/O** | **Note** |
|-----------|------|-------------|---------|----------|
| request | ZkpVerificationRequest | ZKP 검증 요청 | M | [Link](#36-zkpverificationrequest) |

### Output Parameters

| Type | Description | **M/O** | **Note** |
|------|-------------|---------|----------|
| ZkpVerificationResult | ZKP 검증 결과 | M | [Link](#34-zkpverificationresult) |

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
`txId의 E2E 세션으로 E2E 암호화된 ZKP Proof를 복호화하며, 검증하지 않습니다.`

- 동작과 예외는 [1.1.10 decryptZkpProof](#1110-decryptzkpproof)와 같습니다.

### Input Parameters

| Parameter | Type | Description | **M/O** | **Note** |
|-----------|------|-------------|---------|----------|
| txId | String | E2E 세션 조회용 트랜잭션 ID | M | |
| encProof | String | 암호화된 Proof | M | Multibase 인코딩 |
| iv | String | 초기화 벡터 | M | Multibase 인코딩 |
| accE2e | ZkpVerificationRequest.AccE2e | Holder의 E2E 수락 정보 | M | 복호화에 `accE2e.publicKey` 사용. [Link](#36-zkpverificationrequest) |

### Output Parameters

| Type | Description | **M/O** | **Note** |
|------|-------------|---------|----------|
| Proof | 복호화된 Proof 객체 | M | `org.omnione.did.zkp.datamodel.proof.Proof` |

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

## 1.8 구현 클래스

SDK의 기본 구현체는 `org.omnione.did.verifier.v1.core` 패키지에 있습니다. `VerifierSdkBuilder.build()`가 생성하므로 애플리케이션이 직접 생성할 일은 보통 없습니다.

- `VerifierServiceImpl`은 `VerifierService`를 구현합니다. [1.1 VerifierService](#11-verifierservice) 참조.
- `VpOfferProtocolImpl`은 `VpOfferProtocol`을 구현합니다. [1.3 VpOfferProtocol](#13-vpofferprotocol) 참조.
- `VpProfileProtocolImpl`은 `VpProfileProtocol`을 구현합니다. [1.4 VpProfileProtocol](#14-vpprofileprotocol) 참조.
- `VpVerificationProtocolImpl`은 `VpVerificationProtocol`을 구현합니다. [1.5 VpVerificationProtocol](#15-vpverificationprotocol) 참조.
- `VerificationConfirmProtocolImpl`은 `VerificationConfirmProtocol`을 구현합니다. [1.6 VerificationConfirmProtocol](#16-verificationconfirmprotocol) 참조.
- `ZkpProofVerificationProtocolImpl`은 `ZkpProofVerificationProtocol`을 구현합니다. [1.7 ZkpProofVerificationProtocol](#17-zkpproofverificationprotocol) 참조.

인터페이스 밖의 공개 멤버는 아래 생성자뿐이며, 어느 생성자도 인자의 `null` 여부를 확인하지 않습니다.

| Class | Constructor | **Note** |
|-------|-------------|----------|
| VerifierServiceImpl | `VerifierServiceImpl(VerificationConfigProvider configProvider, VerifierInfoProvider verifierInfoProvider, EcdhSessionProvider sessionProvider, StorageProvider storageService, TransactionProvider transactionManager, NonceGenerator nonceGenerator, CryptoHelper cryptoHelper)` | `build()`에서 사용. 기본 프로토콜 구현체 5개를 생성 |
| VerifierServiceImpl | `VerifierServiceImpl(VpOfferProtocol offerService, VpProfileProtocol profileService, VpVerificationProtocol verificationService, VerificationConfirmProtocol confirmService, ZkpProofVerificationProtocol zkpProofVerificationService)` | 전달받은 프로토콜 구현체를 사용. 일부 프로토콜을 직접 구현한 것으로 교체할 때 사용 |
| VpOfferProtocolImpl | `VpOfferProtocolImpl(VerificationConfigProvider configProvider, TransactionProvider transactionManager)` |  |
| VpProfileProtocolImpl | `VpProfileProtocolImpl(VerificationConfigProvider configProvider, VerifierInfoProvider verifierInfoProvider, NonceGenerator nonceGenerator)` |  |
| VpVerificationProtocolImpl | `VpVerificationProtocolImpl(EcdhSessionProvider sessionProvider, StorageProvider storageService, CryptoHelper cryptoHelper)` | `cryptoHelper`는 사용하지 않음 |
| VerificationConfirmProtocolImpl | `VerificationConfirmProtocolImpl()` |  |
| ZkpProofVerificationProtocolImpl | `ZkpProofVerificationProtocolImpl(EcdhSessionProvider sessionProvider, StorageProvider storageService, VerifierInfoProvider verifierInfoProvider, CryptoHelper cryptoHelper)` |  |

### Usage
```java
// customVerification: 애플리케이션이 직접 구현한 VpVerificationProtocol
VerifierService verifierService = new VerifierServiceImpl(
    new VpOfferProtocolImpl(configProvider, transactionProvider),
    new VpProfileProtocolImpl(configProvider, verifierInfoProvider, nonceGenerator),
    customVerification,
    new VerificationConfirmProtocolImpl(),
    new ZkpProofVerificationProtocolImpl(sessionProvider, storageProvider, verifierInfoProvider, cryptoHelper)
);
```

<br>

# 2. 인터페이스

## 2.1 VerificationConfigProvider

### Declaration

```java
public interface VerificationConfigProvider {
    VerificationPolicy getPolicy(String policyId);
    boolean existsPolicy(String policyId);
}
```

### Description
`정책 ID로 검증 정책을 제공합니다. 데이터베이스, 설정 파일, 원격 서비스 등에서 정책을 로드하도록 구현합니다.`

### Methods

| Method | Description | **SDK Call** | **On Failure** |
|--------|-------------|--------------|----------------|
| `VerificationPolicy getPolicy(String policyId)` | `policyId`의 정책을 [VerificationPolicy](#37-verificationpolicy)로 반환합니다. SDK는 `policyName`, `description`, `language`, `mode`, `endpoints`, `validityDuration`, `filter`, `process`를 읽습니다 | `requestVpOffer`, `requestStaticVpOffer`, `requestVerifyProfile` 호출마다 한 번, 애플리케이션이 전달한 `policyId`로 호출 | `null`을 반환하거나 예외를 던집니다. `null`이면 SDK가 `SDK_POLICY_NOT_FOUND`를 발생시키고, 예외는 호출자에게 그대로 전달됩니다 |
| `boolean existsPolicy(String policyId)` | 정책 존재 여부를 반환합니다 | SDK에서 호출하지 않음 |  |

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
`Verifier 정보를 제공합니다: Verifier의 ProviderDetail(VerifyProfile 및 ProofRequestProfile의 verifier로 사용), DID Document(JSON 문자열), DID. 서명 키는 이 인터페이스에서 다루지 않습니다.`

### Methods

| Method | Description | **SDK Call** | **On Failure** |
|--------|-------------|--------------|----------------|
| `ProviderDetail getVerifierInfo()` | Verifier의 `ProviderDetail`(`org.omnione.did.data.model.provider.ProviderDetail`: `did`, `certVcRef`, `name`, `description`, `logo`, `ref`)을 반환합니다. SDK는 이를 그대로 `profile.verifier`에 설정합니다 | `requestVerifyProfile`, `requestZkpProofRequestProfile` 호출마다 한 번 | `null`은 확인하지 않습니다. `requestVerifyProfile`에서는 예외가 그대로 전달되고, `requestZkpProofRequestProfile`에서는 `SDK_UNKNOWN_ERROR`로 변환됩니다 |
| `String getVerifierDidDocument()` | Verifier의 DID Document를 JSON 문자열로 반환합니다 | SDK에서 호출하지 않음 |  |
| `String getVerifierDid()` | Verifier의 DID를 반환합니다 | SDK에서 호출하지 않음 |  |

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
`트랜잭션별 E2E 암호화 세션(임시 ECDH 키쌍)을 생성·관리하고 E2E 암호화된 데이터를 복호화합니다. decrypt는 Holder 공개키로 ECDH를 수행하고 세션키를 유도하여 데이터를 복호화하며, decryptVp, verifyPresentation, decryptZkpProof, verifyZkpProof에서 사용됩니다. SDK는 세션을 저장하지 않으며 기본 구현을 제공하지 않으므로, 인메모리/Redis/DB 등으로 구현해야 합니다.`

### Methods

| Method | Description | **SDK Call** | **On Failure** |
|--------|-------------|--------------|----------------|
| `ReqE2e createSession(String txId)` | `txId`용 임시 E2E 키쌍과 Nonce를 생성해 개인키, Nonce, curve, cipher, padding을 `txId`로 보관하고 [ReqE2e](#35-reqe2e)로 반환합니다(`publicKey`, `nonce`는 Multibase 인코딩) | SDK에서 호출하지 않음. 애플리케이션이 `requestVerifyProfile` 전에 호출해 결과를 `reqE2e`로 전달 | 애플리케이션이 정함 |
| `void saveSession(String txId, KeyPairInfo keyPair, ReqE2e reqE2e)` | SDK가 생성한 키쌍과 E2E 설정을 `txId`로 저장합니다. `decrypt`를 위해 최소한 `keyPair.privateKey`와 `reqE2e`의 `nonce`, `curve`, `cipher`, `padding`을 보관해야 합니다 | `requestZkpProofRequestProfile` 마지막에 한 번, `request.txId`, `CryptoHelper.generateEcKeyPair`로 만든 키쌍, 반환할 프로파일의 `reqE2e`로 호출 | 예외를 던지면 SDK가 `SDK_UNKNOWN_ERROR`로 변환합니다 |
| `ReqE2e getSession(String txId)` | `txId`에 저장된 E2E 설정을 반환합니다 | SDK에서 호출하지 않음 |  |
| `void removeSession(String txId)` | `txId`의 세션을 삭제합니다 | SDK에서 호출하지 않음. 트랜잭션이 끝나면 애플리케이션이 삭제 |  |
| `boolean existsSession(String txId)` | `txId`의 세션 존재 여부를 반환합니다 | `decryptVp`, `verifyPresentation`, `decryptZkpProof`, `verifyZkpProof`의 첫 단계 | `false`이면 SDK가 `SDK_E2E_SESSION_NOT_FOUND`를 발생시킵니다. 예외는 그대로 전달되므로 세션이 없을 때는 `false`를 반환합니다 |
| `String decrypt(String txId, String encHolderPublicKey, String encVp, String iv)` | `txId`의 세션으로 데이터를 복호화합니다: Verifier 개인키와 `encHolderPublicKey`로 ECDH, 공유 비밀키와 세션 Nonce로 세션키 유도, `iv`와 세션의 cipher·padding으로 대칭키 복호화. 평문을 UTF-8 문자열로 반환합니다. 입력은 모두 Multibase 인코딩이며 `encHolderPublicKey`는 암호화되지 않은 Holder 공개키입니다 | `existsSession`이 `true`를 반환한 뒤. VP: 요청의 `encHolderPublicKey`, `encVp`, `iv`. ZKP: `accE2e.publicKey`, `encProof`, `iv` | 예외를 던지면 SDK가 모두 `SDK_DECRYPTION_FAILED`로 변환합니다. 반환값의 `null` 여부는 확인하지 않습니다 |

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
`프로토콜 검증에 필요한 데이터를 제공합니다: Holder/Issuer의 DID Document(JSON 문자열), VC 메타데이터(JSON 문자열), ZKP Credential Schema/Definition. 블록체인, 데이터베이스 등 저장소에 맞게 구현합니다. Policy와 Filter는 이 인터페이스가 아니라 VerificationConfigProvider에서 제공합니다.`

- `findDidDocument`가 `null`을 반환하면 VP 검증은 `SDK_DID_DOCUMENT_NOT_FOUND`로 실패합니다.
- VC 상태는 `getVcMeta` 결과의 `status`(또는 `vcStatus`, `credentialStatus.status`) 필드에서 읽으며, `REVOKED`, `INACTIVE`, `EXPIRED`는 `SDK_INVALID_VC`로 거부됩니다. `getVcMeta`가 `null`을 반환하면 해당 VC의 상태 확인은 생략됩니다.

### Methods

| Method | Description | **SDK Call** | **On Failure** |
|--------|-------------|--------------|----------------|
| `String findDidDocument(String did)` | `did`의 DID Document를 JSON 문자열로 반환합니다(`DidDocument.fromJson`으로 파싱) | `verifyPresentation`: VP의 Holder DID에 한 번, 각 VC의 Issuer DID에 한 번씩 | `null`이면 SDK가 `SDK_DID_DOCUMENT_NOT_FOUND`를 발생시킵니다. 예외는 그대로 전달됩니다 |
| `String getVcMeta(String vcId)` | `vcId`의 VC 메타데이터를 JSON 객체 문자열로 반환합니다. 상태 필드만 읽습니다 | `verifyPresentation`, `validateVcStatuses`: `id`가 있는 VC마다 한 번 | `null`이면 해당 VC의 상태 확인을 생략합니다. 예외와 JSON 파싱 오류는 그대로 전달됩니다 |
| `boolean existsDidDocument(String did)` | DID Document 존재 여부를 반환합니다 | SDK에서 호출하지 않음 |  |
| `CredentialSchema getZKPCredential(String schemaId)` | `schemaId`의 ZKP Credential Schema를 반환합니다 | `verifyZkpProof`: Proof의 identifier마다 한 번, `identifiers[].schemaId`로 호출 | `null`은 확인하지 않고 ZKP 라이브러리에 전달합니다. `VerifierSdkException`은 그대로 전달되고, 그 밖의 예외는 `SDK_SIGNATURE_VERIFICATION_FAILED`로 변환됩니다 |
| `CredentialDefinition getZKPCredentialDefinition(String credDefId)` | `credDefId`의 ZKP Credential Definition을 반환합니다 | `verifyZkpProof`: Proof의 identifier마다 한 번, `identifiers[].credDefId`로 호출 | `getZKPCredential`과 같음 |

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
`트랜잭션 ID를 생성하고 조회합니다. createTransactionId는 새 트랜잭션 ID(UUID 권장)를 반환하며, SDK는 이를 Dynamic VP Offer의 offerId로 사용합니다. getTransactionId는 트랜잭션 ID에 해당하는 DB 기본키(Long)를 반환합니다. SDK는 트랜잭션을 저장하지 않으며 애플리케이션이 관리합니다.`

### Methods

| Method | Description | **SDK Call** | **On Failure** |
|--------|-------------|--------------|----------------|
| `String createTransactionId()` | 고유한 새 트랜잭션 ID를 반환합니다(UUID 권장) | `requestVpOffer` 호출마다 한 번, 반환값은 `offerId`가 됨. `requestStaticVpOffer`에서는 호출하지 않음 | `null` 여부는 확인하지 않습니다. 예외는 그대로 전달됩니다 |
| `Long getTransactionId(String txId)` | `txId`에 해당하는 애플리케이션 내부 키(예: DB 기본키)를 반환합니다 | SDK에서 호출하지 않음 |  |

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
`지정한 바이트 길이의 암호학적으로 안전한 Base64 인코딩 Nonce를 생성합니다. generateNonce()(16바이트)로 생성된 verifierNonce는 VerifyProfile에 포함되어 VP 제출 시 검증됩니다.`

### Methods

| Method | Description | **SDK Call** | **On Failure** |
|--------|-------------|--------------|----------------|
| `String generateNonce(int length)` | `length` 바이트의 암호학적으로 안전한 난수를 문자열로 반환합니다(Base64 권장) | `generateNonce()`를 통해 `length` = 16으로 호출 | 예외는 그대로 전달됩니다 |
| `default String generateNonce()` | `generateNonce(16)`을 호출합니다. 재정의할 수 있습니다 | `requestVerifyProfile` 호출마다 한 번, 반환값은 `profile.process.verifierNonce`가 됨 | 확인하지 않습니다. VP의 `verifierNonce`와 비교되므로 빈 값이면 안 됩니다 |

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
`암호화 유틸리티를 제공합니다: 서명 검증, SHA-256 해시, Base64/Multibase 인코딩, E2E 키쌍 및 Nonce 생성, ECDH 공유 비밀키 생성, 세션키 유도(KDF), 대칭키 복호화. SDK 내부에서는 ProofRequestProfile 생성 시 generateEcKeyPair와 generateNonce를 사용하며, 나머지 메서드는 애플리케이션(예: EcdhSessionProvider 구현체)에서 사용할 수 있습니다. did-crypto-sdk-server 등 암호화 라이브러리를 사용하여 구현합니다.`

### Methods

| Method | Description | **SDK Call** | **On Failure** |
|--------|-------------|--------------|----------------|
| `boolean verifySignature(String publicKey, String signature, byte[] data)` | `publicKey`(Base64 또는 Multibase)로 `data`에 대한 `signature`를 검증하고 결과를 반환합니다 | SDK에서 호출하지 않음. VP·VC 서명은 did-core-sdk-server의 `VpManager`가 검증 |  |
| `String sha256(byte[] data)` | `data`의 SHA-256 해시를 Base64로 반환합니다 | SDK에서 호출하지 않음 |  |
| `byte[] decodeMultibase(String multibase)` | Multibase 문자열을 디코딩합니다 | SDK에서 호출하지 않음 |  |
| `String encodeBase64(byte[] data)` | `data`를 Base64로 인코딩합니다 | SDK에서 호출하지 않음 |  |
| `byte[] decodeBase64(String base64)` | Base64 문자열을 디코딩합니다 | SDK에서 호출하지 않음 |  |
| `KeyPairInfo generateKeyPair(String curve)` | `curve`(`Secp256r1`, `Secp256k1`)의 EC 키쌍을 생성합니다. 두 키 모두 Multibase 인코딩 | SDK에서 호출하지 않음 |  |
| `String generateNonce(int length)` | `length` 바이트의 난수 Nonce를 Multibase 인코딩으로 생성합니다 | `requestZkpProofRequestProfile`에서 `length` = 16으로 호출. 반환값은 프로파일의 `reqE2e.nonce`가 되고 `saveSession`에 전달됨 | 예외는 `SDK_UNKNOWN_ERROR`로 변환됩니다 |
| `String encodeMultibase(byte[] data)` | `data`를 Multibase로 인코딩합니다 | SDK에서 호출하지 않음 |  |
| `byte[] generateSharedSecret(byte[] holderPublicKey, byte[] verifierPrivateKey, String curve)` | ECDH 공유 비밀키를 계산합니다 | SDK에서 호출하지 않음 |  |
| `byte[] deriveSessionKey(byte[] sharedSecret, byte[] nonce, String cipherType)` | 공유 비밀키와 Nonce로 `cipherType`(예: `AES-256-CBC`)용 대칭 세션키를 유도합니다 | SDK에서 호출하지 않음 |  |
| `byte[] decrypt(byte[] encData, byte[] sessionKey, byte[] iv, String cipherType, String paddingType)` | 세션키로 `encData`를 복호화합니다(`paddingType` 예: `PKCS5`) | SDK에서 호출하지 않음 |  |
| `KeyPairInfo generateEcKeyPair(String curve)` | E2E 암호화용 EC 키쌍을 생성합니다. 두 키 모두 Multibase 인코딩. `publicKey`는 `reqE2e.publicKey`로 Holder에 전달되고, 키쌍은 `saveSession`에 전달됩니다 | `requestZkpProofRequestProfile`에서 `request.reqE2e.curve`로 호출 | 예외는 `SDK_UNKNOWN_ERROR`로 변환됩니다. `null`을 반환해도 `SDK_UNKNOWN_ERROR`로 끝납니다 |

<br>

# 3. 데이터 클래스

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
| offerId | String | Offer 식별자 | O | Static QR인 경우 null |
| type | OfferType | Offer 타입 | M | `VerifyOffer` \| `VerifyProofOffer`. SDK의 Offer 메서드는 `VerifyOffer`로 설정 |
| mode | PresentMode | 제출 모드 | M | `DIRECT` \| `INDIRECT` \| `PROXY` (표시명 `Direct`, `Indirect`, `Proxy`) |
| device | String | 응대장치 식별자 | M | |
| service | String | 서비스 식별자 | M | |
| endpoints | List\<String\> | Verifier API 엔드포인트 목록 | O | 정책에서 가져옴. Direct 모드에서 필수 |
| validUntil | String | Offer 만료 시각 | O | ISO-8601. Static QR인 경우 null |
| locked | Boolean | Offer 잠김 여부 | O | 기본값: false. `true`: passcode 필요 |

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
| txId | String | 트랜잭션 식별자 | M | E2E 세션 조회에 사용 |
| vp | String | 평문 VP JSON | O | `verifyPresentation`에서 사용하지 않음 |
| encrypted | Boolean | E2E 암호화 여부 | O | 기본값: false. `verifyPresentation`에서 사용하지 않음 (VP는 항상 복호화됨) |
| encVp | String | 암호화된 VP | M | Multibase 인코딩 |
| iv | String | 초기화 벡터 | M | Multibase 인코딩 |
| encHolderPublicKey | String | Holder의 E2E 공개키 | M | Multibase 인코딩 (암호화되지 않음) |
| verifierNonce | String | VerifyProfile에 포함된 Nonce | M | VP의 `verifierNonce`와 같아야 함 (불일치 시 `SDK_INVALID_NONCE`) |
| serverToken | String | 인증 토큰 | O | SDK에서 사용하지 않음 |
| requiredAuthType | Integer | 요구 인증 타입 비트마스크 | O | `null` 또는 0x00000000: 제한 없음. SDK는 0x00000006(PIN 또는 BIO)만 확인하며, 그 외 값은 SDK에서 확인하지 않음 |
| filter | Filter | 정책에서 가져온 클레임 필터 | O | 설정된 경우 VpManager 검증에 적용 |

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
| txId | String | 트랜잭션 식별자 | M | |
| verified | Boolean | 검증 성공 여부 | M | |
| holderDid | String | Holder DID | O | `VerifyProofOffer`인 경우 설정되지 않음 |
| submittedVcs | Map\<String, Object\> | 제출된 VC 목록 (VC ID → VC 객체) | O | `VerifyProofOffer`인 경우 설정되지 않음 |
| extractedClaims | Map\<String, Object\> | 추출된 클레임 (코드 → 값) | O | `VerifyProofOffer`인 경우: `{"zkpVerificationResult": "Successful"}` |
| verifiedAt | Instant | 검증 일시 | M | |
| errorMessage | String | 실패 사유 | O | `verified=false` 시 존재 |
| errorCode | String | 에러 코드 | O | SDK에서 설정하지 않음 |

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
| txId | String | 트랜잭션 식별자 | M | |
| verified | boolean | ZKP Proof 유효 여부 | M | ZKP 라이브러리가 Proof를 유효하지 않다고 판단하면 `false` |
| errorMessage | String | 실패 사유 | O | `verified=false` 시 존재 |
| revealedAttributes | Map\<String, String\> | 공개된 속성 값 | O | `verified=true` 시 설정. 현재 구현은 빈 Map을 반환 |
| verifiedPredicates | List\<VerifiedPredicate\> | 검증된 Predicate 목록 | O | 현재 구현에서는 설정되지 않음 |
| holderDid | String | Holder DID | O | 현재 구현에서는 설정되지 않음 |
| proofJson | String | 검증된 Proof JSON | O | `verified=true` 시 존재 |
| verifiedAt | long | 검증 시각 | M | Unix timestamp (밀리초) |

**VerifiedPredicate**

| Parameter | Type | Description | **M/O** | **Note** |
|-----------|------|-------------|---------|----------|
| type | String | Predicate 타입 | M | `>=`, `<=`, `>`, `<` |
| name | String | 속성 이름 | M | |
| value | String | 비교 값 | M | |
| verified | boolean | Predicate 검증 성공 여부 | M | |

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
| nonce | String | E2E 세션키 유도용 Nonce | M | 16바이트 |
| curve | String | 키 곡선 | M | `Secp256r1` 또는 `Secp256k1` |
| publicKey | String | Verifier의 임시 E2E 공개키 | M | Multibase 인코딩 |
| cipher | String | 암호화 알고리즘 | M | 예: `AES-256-CBC` |
| padding | String | 패딩 방식 | M | 예: `PKCS5` |

`ProofRequestProfileRequest`에서 사용할 때는 `curve`, `cipher`, `padding`만 읽으며, `publicKey`와 `nonce`는 SDK가 생성합니다.

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
| txId | String | 트랜잭션 식별자 | M | E2E 세션 조회에 사용 |
| encProof | String | E2E 암호화된 ZKP Proof | M | Multibase 인코딩 |
| iv | String | 초기화 벡터 | M | Multibase 인코딩. 복호화에 사용 |
| accE2e | AccE2e | Holder의 E2E 수락 정보 | M | |
| nonce | String | Holder가 제출한 Proof Nonce | M | 10진수 숫자 문자열. `proofRequest`의 Nonce와 같아야 함 |
| proofRequest | Object | Verifier가 저장한 ProofRequest | M | `org.omnione.did.zkp.datamodel.proofrequest.ProofRequest`. Holder가 제출한 값이 아니라 ProofRequestProfile에서 저장한 값이어야 함 |
| proofVerifyParams | List\<ProofVerifyParam\> | Proof 검증 파라미터 | O | `verifyZkpProof`에서 사용하지 않음. Schema와 Definition은 `StorageProvider`로 조회 |

**AccE2e**

| Parameter | Type | Description | **M/O** | **Note** |
|-----------|------|-------------|---------|----------|
| publicKey | String | Holder의 E2E 공개키 | M | Multibase 인코딩. 복호화에 사용 |
| iv | String | 초기화 벡터 | O | SDK에서 사용하지 않음 |
| proof | Object | AccE2e에 대한 Holder 서명 | O | SDK에서 검증하지 않음 |

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
| policyId | String | 정책 식별자 | O | SDK에서 사용하지 않음 |
| policyName | String | 정책 이름 | M | VerifyProfile의 `title` |
| description | String | 정책 설명 | O | VerifyProfile의 `description`. `null`이면 빈 문자열 |
| logo | Object | 로고 이미지 | O | SDK에서 사용하지 않음 |
| language | String | 언어 코드 | M | 예: `ko`, `en`. VerifyProfile의 `language` |
| mode | String | 제출 모드 | M | `Direct`, `Indirect`, `Proxy`([PresentMode](#315-presentmode) 표시명). 그 밖의 값이면 Offer 메서드에서 `IllegalArgumentException` 발생 |
| endpoints | List\<String\> | Verifier API 엔드포인트 목록 | O | VP Offer의 `endpoints` |
| validityDuration | int | Offer 유효기간(초) | M | Dynamic VP Offer의 `validUntil` = 현재 시각 + 이 값. 기본값: 0 |
| filter | Filter | 클레임 필터 | M | VerifyProfile의 `profile.filter`. SDK에서 확인하지 않음 |
| process | VerifyProcess | 기본 process | O | `endpoints`, `authType`을 VerifyProfile의 `profile.process`에 복사. `reqE2e`, `verifierNonce`는 SDK가 설정 |
| verifier | ProviderDetail | Verifier 정보 | O | SDK에서 사용하지 않음. 대신 `VerifierInfoProvider.getVerifierInfo()` 사용 |

`VerificationConfigProvider.getPolicy`가 이 클래스를 반환합니다. VP를 검증할 때는 같은 정책의 `authType`과 `filter`를 `VpVerificationRequest.requiredAuthType`과 `filter`로 전달합니다.

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
| id | String | 프로파일 식별자 | M | 요청의 `profileId` |
| type | String | 프로파일 타입 | M | 항상 `VerifyProfile` |
| title | String | 제목 | M | ZkpPolicy의 `title` |
| description | String | 설명 | O | ZkpPolicy의 `description` |
| language | String | 언어 | O | ZkpPolicy의 `language` |
| profile | ZkpInnerProfile | 내부 프로파일 | M | [Link](#39-zkpinnerprofile) |
| proof | Object | Verifier 서명 | O | SDK 결과에서는 `null`. 애플리케이션이 서명하여 `setProof`로 설정 |

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
| verifier | Object | Verifier 정보 | M | `VerifierInfoProvider.getVerifierInfo()`의 `ProviderDetail` |
| proofRequest | ProofRequest | ZKP 증명 요청 | M | `org.omnione.did.zkp.datamodel.proofrequest.ProofRequest`. `ZkpVerificationRequest.proofRequest`를 위해 보관 |
| reqE2e | ReqE2e | Verifier의 E2E 키 교환 정보 | M | `publicKey`, `nonce`는 SDK가 생성. [Link](#35-reqe2e) |

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
| txId | String | 트랜잭션 식별자 | M | `saveSession`으로 저장되는 E2E 세션의 키. `ZkpVerificationRequest.txId`에 같은 값 사용 |
| policyId | String | ZKP 정책 식별자 | O | SDK에서 사용하지 않음 |
| profileId | String | 프로파일 식별자 | M | ProofRequestProfile의 `id`. UUID 권장 |
| reqE2e | ReqE2e | E2E 설정 | M | `curve`, `cipher`, `padding`만 읽음. [Link](#35-reqe2e) |
| verifierDid | String | Verifier DID | O | SDK에서 사용하지 않음 |

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
| policyId | String | 정책 식별자 | O | SDK에서 사용하지 않음 |
| title | String | 프로파일 제목 | M | ProofRequestProfile의 `title` |
| description | String | 프로파일 설명 | O | |
| language | String | 프로파일 언어 | O | 예: `ko`, `en` |
| name | String | ProofRequest 이름 | M | `proofRequest.name` |
| version | String | ProofRequest 버전 | M | `proofRequest.version` |
| requestedAttributes | String | 요청 속성(JSON) | O | referent → `AttributeInfo`(`name`, `restrictions`) JSON 객체. `proofRequest.requestedAttributes`로 파싱. `null`이거나 비어 있으면 무시 |
| requestedPredicates | String | 요청 Predicate(JSON) | O | referent → `PredicateInfo`(`name`, `pType`(`GE`, `LE`, `GT`, `LT`, `EQ`), `pValue`, `restrictions`) JSON 객체. `proofRequest.requestedPredicates`로 파싱. `null`이거나 비어 있으면 무시 |
| curve | String | E2E 곡선 | O | SDK에서 사용하지 않음(`ProofRequestProfileRequest.reqE2e` 사용) |
| cipher | String | E2E 암호화 알고리즘 | O | SDK에서 사용하지 않음 |
| padding | String | E2E 패딩 | O | SDK에서 사용하지 않음 |

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
| publicKey | String | 공개키 | M | Multibase 인코딩. `reqE2e.publicKey`로 Holder에 전달 |
| privateKey | String | 개인키 | M | Multibase 인코딩. `EcdhSessionProvider.decrypt`를 위해 Verifier만 보관 |

`CryptoHelper.generateKeyPair`와 `generateEcKeyPair`가 반환하며, `EcdhSessionProvider.saveSession`에 전달됩니다.

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

`org.omnione.did.verifier.v1.model.data.ProofVerifyParam`은 `ZkpVerificationRequest.proofVerifyParams`의 원소 타입입니다. `verifyZkpProof`는 이 값을 사용하지 않고 `StorageProvider`로 Schema와 Definition을 조회합니다.

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
| VerifyOffer | VP 검증 Offer | `requestVpOffer`, `requestStaticVpOffer`가 설정 |
| VerifyProofOffer | ZKP Proof 검증 Offer | `confirmVerification`에서 JSON을 파싱하지 않음 |

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
| DIRECT | Direct | Holder 앱이 Verifier에 VP를 직접 제출 |
| INDIRECT | Indirect | 응대장치를 거쳐 Verifier에 VP 전달 |
| PROXY | Proxy | Proxy 서버를 거쳐 VP 전달 |

`fromDisplayName`은 표시명(대소문자 구분)에 해당하는 상수를 반환하며, 그 밖의 값이면 `IllegalArgumentException`이 발생합니다. `toString()`은 표시명을 반환하며 Jackson 직렬화에 사용됩니다(`@JsonValue`). Gson은 상수 이름(예: `DIRECT`)으로 직렬화합니다.

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
| errorCode | VerifierSdkErrorCode | 에러 코드 | M | [Link](#317-verifiersdkerrorcode) |

SDK의 모든 오류는 이 unchecked 예외로 발생합니다. 첫 번째 생성자는 에러 코드의 기본 메시지를, 두 번째 생성자는 전달한 메시지를 사용합니다. Provider 구현체도 이 예외를 던질 수 있으며, 각 Provider 메서드의 예외를 SDK가 어떻게 처리하는지는 [2. 인터페이스](#2-인터페이스)에 설명되어 있습니다.

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
| code | String | 에러 코드 | M | `SSDKVRF`와 숫자 6자리 |
| message | String | 기본 에러 메시지 | M | |
| httpStatus | int | 권장 HTTP 상태 코드 | M | 예: 400, 404, 500 |

`getMessageByCode`는 코드 문자열에 해당하는 메시지를 반환하며, 일치하는 상수가 없으면 `Unknown error code: <code>`를 반환합니다. 전체 코드 목록은 [Verifier SDK Error Codes](../errorCode/VerifierSDKError.md)를 참조합니다.

