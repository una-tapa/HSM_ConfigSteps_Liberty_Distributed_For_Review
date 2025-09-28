# Hardware Cryptography for Liberty - Technote Draft

## Overview

This repository contains a draft technote for enabling hardware cryptography with Liberty for Linux. The document is provided for review by our customer and IBM support teams.

## Purpose

This technote aims to provide comprehensive guidance for configuring hardware cryptographic devices with Liberty for Linux environments. It extends IBM's existing documentation on hardware cryptography for Liberty by addressing distributed environments.

## Feedback

Your feedback is valuable to improve this technote. Please provide comments through one of the following channels:
- Submit a Pull Request with suggested changes
- Provide feedback via the associated support case

## Related Documentation

This technote is modeled after existing IBM documentation for z/OS environments:

- [Enabling hardware cryptography for Liberty for z/OS using Java 8](https://www.ibm.com/support/pages/enabling-hardware-cryptography-liberty-zos-using-java-8)
- [Enabling hardware cryptography for Liberty for z/OS using Java 11, Java 17, or Java 21](https://www.ibm.com/support/pages/node/6840291)

## Content

The primary document in this repository is `enabling_hardware_crypto_liberty_linux.md`, which provides step-by-step instructions for configuring Liberty for Linux with hardware security modules (HSMs) using the PKCS#11 interface. The implementation details are based on a successful customer deployment with a Thales Luna HSM.

## Status

This is a draft document intended for review purposes only. The content has not been officially published and may be subject to changes based on feedback.

## Acknowledgements

Special thanks to our customer for providing the implementation details that made this technote possible.