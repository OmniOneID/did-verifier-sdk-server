# Changelog

## [Unreleased] (v1.0.0)

### 🚀 New Features
- Server Verifier SDK
    - VP Offer generation (Dynamic/Static QR)
    - Verify Profile generation with E2E key exchange
    - VP verification (E2E decryption, signature verification, VC status check)
    - Verification confirmation and claim extraction
    - ZKP Proof Request Profile generation and ZKP proof verification
    - Provider interfaces for application servers
- Uses did-common-sdk-server 2.0.0 and did-zkp-sdk-server 2.0.1

### 🐛 Bug Fixes
- ZKP proof verification uses the nonce stored in the proof request and rejects invalid proofs
