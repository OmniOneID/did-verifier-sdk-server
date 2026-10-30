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

| 버전 | 날짜 | 변경 내용 |
|------|------|---------|
| v1.0.0 | 2026-10-20 | 최초 작성 |


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

`VerifierService` 인터페이스는 SDK Facade입니다. `VerifierSdkBuilder`를 통해 인스턴스를 획득합니다. 7개 Provider는 모두 필수이며, 하나라도 누락되면 `build()`에서 `VerifierSdkException`(`SDK_CONFIGURATION_ERROR`)이 발생합니다.

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

<br>

### 1.1.3 requestVerifyProfile

### Class Name
`VerifierService`

### Function Name
`requestVerifyProfile`

### Function Introduction
`주어진 정책에 대한 검증 요구사항을 기술하는 VerifyProfile을 생성합니다. SDK는 NonceGenerator로 verifierNonce를 생성하여 reqE2e와 함께 프로파일의 process에 넣습니다. 호출 애플리케이션은 반환된 프로파일에 서명을 추가한 뒤 Holder에게 전달해야 합니다.`

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

<br>

### 1.1.6 confirmVerification (VP)

### Class Name
`VerifierService`

### Function Name
`confirmVerification`

### Function Introduction
`검증된 VP에서 Holder DID, 제출된 VC, 클레임을 추출하여 검증 결과를 반환합니다(SDK는 결과를 저장하지 않음). verified가 false이면 VP를 파싱하지 않고 실패 결과를 반환합니다. 표준 VP 검증 흐름에서 사용하는 오버로드입니다.`

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

<br>

### 1.1.8 requestZkpProofRequestProfile

### Class Name
`VerifierService`

### Function Name
`requestZkpProofRequestProfile`

### Function Introduction
`ZKP 검증 요청을 정의하는 ProofRequestProfile을 생성합니다. SDK는 ProofRequest의 Nonce와 E2E 키쌍을 생성하고, EcdhSessionProvider.saveSession으로 E2E 세션을 저장합니다. 반환된 프로파일에는 Proof가 포함되지 않으므로, 호출 애플리케이션이 서명하여 Proof를 설정한 뒤 Holder에게 전달해야 합니다.`

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

<br>

# 3. 데이터 클래스

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
