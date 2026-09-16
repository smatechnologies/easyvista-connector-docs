---
sidebar_label: 'Release notes'
title: EasyVista Connector release notes
description: "Version history and change details for the EasyVista Connector, including new features, improvements, and bug fixes."
tags:
  - Reference
  - System Administrator
  - Automation Engineer
---

# EasyVista Connector release notes

This connector requires OpCon Release 21.0 or higher.

:::note
Release numbers on this page are not in date order. 18.3.1 was released after 19.1.1, from a separate development line.
:::

## 21

### 21.1.5

2025 November

#### What's new

**CON-370**: Implemented encryption of the `TOKEN` value in `Connector.config` using the `Encrypt.exe` utility.

#### Why this matters

Token values stored in `Connector.config` must now be encrypted using `Encrypt.exe` before use. This protects OpCon API credentials at rest and aligns the connector with the encryption standard applied to other credential values in the configuration file.

#### Upgrade notes

When upgrading to 21.1.5, encrypt the `TOKEN` value in the `[OPCON API]` section of `Connector.config` using `Encrypt.exe` before restarting the connector.

---

### 21.0.4

2024 June

#### What's new

**CONNUTIL-639**: Implemented a new tag type called `EXIT` that causes the connector to exit without creating an incident ticket when a matching OpCon job tag is encountered. This option is only available when tag routing is enabled.

To use the `EXIT` tag, add an `EXIT` entry to the `tags` section of the EasyVista template. Set `indicatorValue` to the tag name that signals no ticket should be created. When the connector finds a matching job tag, it exits with the following message:

```
Ticket creation terminated for the job <schedule>.<job> as EXIT tag defined
```

**CONNUTIL-640**: Added a validation check for the `DEFAULT` routing tag when tag routing is enabled. If the `DEFAULT` tag is not defined in the template, the connector terminates with the following message:

```
Connector terminated due to configuration error - tag routing enabled, but no default routing tag defined
```

#### Bug fixes

**CONNUTIL-633**: Fixed the order in which files appear in the EasyVista ticket to match the order extracted from OpCon.

#### Upgrade notes

When upgrading to 21.0.4 with tag routing enabled, verify that a `DEFAULT` tag is defined in each template. If the `DEFAULT` tag is missing, the connector will not start.

---

### 21.0.3

2024 March

#### What's new

**CONNUTIL-629**: Fixed an issue where Unix job logs were not correctly uploaded to the EasyVista incident.

---

### 21.0.2

2022 July

#### What's new

**CONNUTIL-575**: Updated the web service client to ignore unknown API fields returned by the EasyVista REST API.

---

### 21.0.1

2022 April

#### What's new

:eight_spoked_asterisk: First release of the EasyVista Connector for OpCon 21.

---

## 20

### 20.0.1

2021 October

#### What's new

No change detail is available for this release. It was built from the same development line as 18.3.1 and carries the same tag routing enhancement, allowing more than one attribute to be set for a routing entry.

---

## 19

### 19.1.1

2020 August

#### What's new

No change detail is available for this release.

---

## 18

### 18.3.1

2021 October

#### What's new

:eight_spoked_asterisk: Introduced template files, allowing one connector to submit requests to more than one EasyVista instance.

:eight_spoked_asterisk: Added proxy server support.

:eight_spoked_asterisk: Added tag routing, using OpCon job tags to route incident tickets.

:eight_spoked_asterisk: Extended tag routing so that more than one attribute can be set for a routing entry.

