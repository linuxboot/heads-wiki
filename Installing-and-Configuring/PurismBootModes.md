---
layout: default
title: Purism Boot Modes
permalink: /PurismBootModes/
parent: Installing and configuring
nav_order: 80
---

<!-- markdownlint-disable MD033 -->
<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>
<!-- markdownlint-enable MD033 -->

Purism Boot Modes
===

[Purism](https://puri.sm) has developed and upstreamed two distinct boot modes for Heads that provide different levels of security depending on user needs. These modes are available in PureBoot (Purism's distribution of Heads) and have been contributed back to the upstream Heads project.

## Overview

The two modes address different user requirements:

- **Basic Mode**: Simplified setup with minimal security features for users who want the benefits of Heads without complex security requirements
- **Restricted Mode**: Advanced security features for users who need maximum protection and are willing to manage additional security procedures

## Basic Mode

PureBoot Basic mode provides a streamlined Heads experience with reduced security complexity. This mode is designed for users who want some of the benefits of Heads firmware but prefer not to manage USB security dongles or complex attestation procedures.

**Key characteristics:**
- No USB security dongle required
- Simplified boot process
- Reduced security attestation requirements
- Easier initial setup and daily use

For detailed information about Basic mode, including setup instructions and technical details, refer to Purism's official documentation:

- [Purism Blog: Introducing PureBoot Basic](https://puri.sm/posts/introducing-pureboot-basic-2/)
- [Purism Documentation: PureBoot Basic Overview](https://docs.puri.sm/Software/PureBoot/Overview/Basic.html)

## Restricted Mode

PureBoot Restricted mode implements advanced security features that provide stronger protection against various attack vectors. This mode is recommended for users with high security requirements who are comfortable with additional operational complexity.

**Key characteristics:**
- Enhanced security through USB security dongle integration
- Advanced attestation procedures
- Stronger protection against tampering
- More complex daily operations requiring security dongle interaction

For detailed information about Restricted mode, including setup procedures and security implications, refer to Purism's official documentation:

- [Purism Blog: Introducing PureBoot Restricted Boot](https://puri.sm/posts/introducing-pureboot-restricted-boot/)
- [Purism Documentation: PureBoot Restricted Overview](https://docs.puri.sm/Software/PureBoot/Overview/Restricted.html)

## Choosing Between Modes

The choice between Basic and Restricted modes depends on your specific security requirements and operational preferences:

**Choose Basic Mode if:**
- You want simplified daily operations
- You prefer not to manage USB security dongles
- Your threat model accepts reduced security attestation
- You need easier setup and maintenance procedures

**Choose Restricted Mode if:**
- You require maximum security protection
- You can manage USB security dongle procedures
- Your threat model demands strong attestation
- You're comfortable with additional operational complexity

## Integration with Heads

Both modes have been contributed to the upstream Heads project and represent important developments in making secure boot accessible to different user communities. The implementation demonstrates how the Heads framework can accommodate varying security requirements while maintaining its core principles.

For general Heads configuration and setup information, see:
- [Prerequisites](/Prerequisites)
- [Configuring Keys](/Configuring-Keys)
- [Boot Options](/BootOptions)

## Community Discussion

The development and integration of these modes has been discussed extensively in the Heads community. Key discussion points include:

- Balancing security with usability
- Maintaining compatibility with existing Heads installations  
- Standardizing mode selection and configuration procedures
- Documentation and user guidance best practices

For ongoing discussions about boot modes and security features, you can join the community conversations linked in the original feature requests and development discussions.