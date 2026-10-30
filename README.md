Server Verifier SDK
==

Welcome to the Server Verifier SDK Repository. <br>
This repository contains the source code, documentation, and related resources for the Server Verifier SDK, which provides the VP (Verifiable Presentation) and ZKP (Zero-Knowledge Proof) verification protocols used by the OpenDID Verifier server.

## Folder Structure
Overview of the major folders and documents in the project directory:

```
did-verifier-sdk-server
├── CHANGELOG.md
├── CLA.md
├── CODE_OF_CONDUCT.md
├── CONTRIBUTING.md
├── LICENSE
├── dependencies-license.md
├── MAINTAINERS.md
├── README.md
├── README_ko.md
├── RELEASE-PROCESS.md
├── SECURITY.md
├── docs
│   └── api
│       └── VERIFIER_SDK-SERVER_API.md
│       └── VERIFIER_SDK-SERVER_API_ko.md
│   └── errorCode
│       └── VerifierSDKError.md
└── source
    └── did-verifier-sdk-server
        ├── gradle
        ├── libs
        └── src
        └── build.gradle
        └── README.md
```

<br/>

Below is a description of each folder and file in the directory:

| Name                    | Description                                     |
| ----------------------- | ----------------------------------------------- |
| CHANGELOG.md            | Version-specific changes in the project         |
| CODE_OF_CONDUCT.md      | Code of conduct for contributors                |
| CONTRIBUTING.md         | Contribution guidelines and procedures          |
| LICENSE                 | License                                         |
| dependencies-license.md | Licenses for the project’s dependency libraries |
| MAINTAINERS.md          | Guidelines for project maintainers              |
| RELEASE-PROCESS.md      | Procedures for releasing new versions           |
| SECURITY.md             | Security policies and vulnerability reporting   |
| docs                    | Documentation                                   |
| ┖ api                   | API guide documentation                         |
| ┖ errorCode             | Error codes and troubleshooting guides          |
| source                  | SDK source code project                         |
| ┖ did-verifier-sdk-server | Verifier SDK source code and build files      |
| &nbsp;&nbsp;&nbsp;┖ gradle                | Gradle build configurations and scripts         |
| &nbsp;&nbsp;&nbsp;┖ libs                  | OpenDID SDK libraries used by this SDK          |
| &nbsp;&nbsp;&nbsp;┖ src                   | Main source code directory                      |
| &nbsp;&nbsp;&nbsp;┖ build.gradle          | Gradle build configuration file                 |
| &nbsp;&nbsp;&nbsp;┖ README.md             | Build and usage instructions                    |

<br/>


## Libraries

Libraries used in this project are organized into two main categories:

1. **OpenDID Libraries**: These libraries are developed by the OpenDID project and are available in the [libs folder](source/did-verifier-sdk-server/libs). They include:

   - `did-core-sdk-server-2.0.0.jar`
   - `did-crypto-sdk-server-2.0.0.jar`
   - `did-datamodel-sdk-server-2.0.0.jar`
   - `did-zkp-sdk-server-2.0.1.jar`
   - `did-common-sdk-server-2.0.0.jar`

2. **Third-Party Libraries**: These libraries are open-source dependencies managed via the [build.gradle](source/did-verifier-sdk-server/build.gradle) file. For a detailed list of third-party libraries and their licenses, please refer to the [dependencies-license.md](dependencies-license.md) file.

## Installation
For build and usage instructions, please refer to the [Verifier SDK README](source/did-verifier-sdk-server/README.md).

## API Reference

| Document                | Description                                                   | Link                                                           |
| ----------------------- | ------------------------------------------------------------- | -------------------------------------------------------------- |
| Verifier SDK Server API | `VerifierService` APIs, provider interfaces and data models   | [English](./docs/api/VERIFIER_SDK-SERVER_API.md) / [Korean](./docs/api/VERIFIER_SDK-SERVER_API_ko.md) |
| Error Codes             | `VerifierSdkErrorCode` list                                   | [Verifier SDK Error Codes](./docs/errorCode/VerifierSDKError.md) |

## Change Log

The Change Log provides a detailed record of version-specific changes and updates. You can find it here:
- [Change Log](./CHANGELOG.md)

## OpenDID Demonstration Videos <br>
To watch our demonstration videos of the OpenDID system in action, please visit our [Demo Repository](https://github.com/OmniOneID/did-demo-server). <br>

These videos showcase key features including user registration, VC issuance, and VP submission processes.

## Contributing

Please read [CONTRIBUTING.md](CONTRIBUTING.md) and [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md) for details on our code of conduct, and the process for submitting pull requests to us.

## License
[Apache 2.0](LICENSE)
