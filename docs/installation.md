---
title: EasyVista Connector installation
description: "Install, configure, and connect the EasyVista Connector to OpCon and EasyVista so that incident tickets are created automatically when jobs fail."
sidebar_label: 'Installation'
tags:
  - Procedural
  - System Administrator
  - Installation
---

# EasyVista Connector installation

## What is it?

The EasyVista Connector installation sets up the software that allows OpCon to automatically create EasyVista incident tickets when jobs fail. The installation process connects the connector to the OpCon REST API and to one or more EasyVista instances through template files.

Complete this installation when you need to:

- Set up the EasyVista Connector for the first time on an OpCon Windows Agent
- Configure the connector to communicate with the OpCon REST API and one or more EasyVista instances
- Define Notification Manager rules that trigger the connector when jobs fail

## Supported software levels

The following software levels are required to implement the EasyVista Connector:

- OpCon Release 21.0 or higher
- Embedded Java OpenJDK 11 (included in the installation)
- OpCon REST API configured to use TLS
- OpCon Windows Agent to provide the link to the EasyVista Connector
- OpCon Notification Manager
- An EasyVista implementation that supports the REST API

## Installation

The installation process consists of the following steps:

- OpCon Windows Agent installation
- EasyVista Connector installation
- EasyVista Connector configuration
- OpCon Notification Manager definition

### OpCon Windows Agent installation

Copy the supplied install file `SMAEasyVistaConnector-win.zip` and extract it into the installation directory.

After the installation is complete, the root installation directory contains the connector executable (`EasyVista.exe`), the encryption utility (`Encrypt.exe`), the `Connector.config` file, and four directories: `java`, `joblogs`, `templates`, and `log`. The `java` directory contains the Java software required to run the connector (OpenJDK 11). The `joblogs` directory temporarily stores job logs extracted from the OpCon system. The `log` directory contains the connector log files. The `templates` directory contains the EasyVista template files.

### EasyVista Connector installation

The EasyVista Connector can be installed on the OpCon Windows server.

To install the EasyVista Connector, complete the following steps:

1. Copy the downloaded install file `SMAEasyVistaConnector-win.zip` and extract it into a temporary directory (for example, `c:\temp`).
2. Extract the contents, including subdirectories, into the required installation directory.

#### Create `$SCHEDULE DATE-EVIS` global property

Create the special **`$SCHEDULE DATE-EVIS`** global property that stores the schedule date in `yyyy-MM-dd` format from the standard `$SCHEDULE DATE` property. Set the value to `yyyy-MM-dd`.

### EasyVista Connector configuration

The configuration of the EasyVista Connector requires setting the OpCon connection information for the OpCon system associated with the connector.

All user and password values placed in the configuration and template files must be encrypted using the `Encrypt.exe` utility provided with the connector.

#### Encrypt utility

The `Encrypt.exe` utility uses standard 64-bit encryption.

The utility supports a `-v` argument and displays the encrypted value.

To encrypt a value on Windows, run the following command:

```
Encrypt.exe -v abcdefg
```

#### Connector.config configuration

Configure the `Connector.config` file in the installation directory with the required values.

| Property | Value |
|---|---|
| **[GENERAL]** | Section header |
| **JOBLOGDIR** | The name of the directory where retrieved log files are stored. After successful attachment to the EasyVista incident, the log file is deleted. This is a subdirectory of the installation directory (default: `joblogs`) |
| **TEMPLATESDIR** | The name of the directory where template definitions are stored. This is a subdirectory of the installation directory (default: `templates`) |
| **DEBUG** | Enables debug mode when set to `ON`. Run with `OFF` in production; set to `ON` when troubleshooting to capture additional log detail. Values: `ON` or `OFF` (default: `OFF`) |
| **[PROXY SERVER]** | Section header — defines a proxy server connection if required |
| **USES_PROXY** | Indicates whether the connector should use a proxy server. Values: `True` or `False` (default: `False`) |
| **ADDRESS** | The address of the proxy server |
| **PORT** | The port of the proxy server |
| **[OPCON API]** | Section header — defines the connection to the OpCon REST API |
| **ADDRESS** | The server address of the OpCon REST API |
| **PORT** | The port number used by the OpCon REST API server |
| **USES_TLS** | Must be set to `True` |
| **TOKEN** | An application token used for authentication when communicating with the OpCon REST API. This value must be encrypted using `Encrypt.exe` |
| **[TICKET DEFINITIONS]** | Section header — defines information used when creating a ticket |
| **DESCRIPTION** | The text used as the incident description. Default value: `OpCon Task Failure ( date {0} schedule {1} job {2} server {3} error code {4} )`. The text is customizable using `{n}` placeholder syntax, where: `{0}` = schedule date (`$SCHEDULE DATE-EVIS`), `{1}` = schedule name (`[[JI.SCHED]]`), `{2}` = job name (`[[JI.JOB]]`), `{3}` = agent name (`[[JI.MACH]]`), `{4}` = termination code (`[[JI.ECODE]]`) |
| **TITLE** | The text used as the incident title. Default value: `OpCon Task Failure (schedule {1} job {2})`. Uses the same `{n}` placeholder syntax as DESCRIPTION |

Example `Connector.config` file:

```
[GENERAL]
JOBLOGDIR=joblogs
TEMPLATESDIR=templates
DEBUG=OFF

[PROXY SERVER]
USES_PROXY=False
SERVER=
PORT=

[OPCON API]
SERVER=BVHTEST02
PORT=9010
USES_TLS=True
TOKEN=fc0520dc-fc93-4d3a-bf2f-7d0584c69df2

[TICKET DEFINITIONS]
DESCRIPTION=OpCon Task Failure ( date {0} schedule {1} job {2} server {3} error code {4} )
TITLE=OpCon Task Failure ( schedule {0} job {1} )
```

The `[OPCON API]` section provides the information needed to connect to the OpCon system using the OpCon REST API so the job log can be retrieved. The `TOKEN` value contains an application token for authentication (see the OpCon REST API documentation for instructions on generating an application token).

#### Templates

Templates provide information about the EasyVista connection and the definitions that will be submitted in the JSON payload as part of the request. A default template is provided in the `templates` directory of the connector installation.

The template includes the address of the EasyVista instance, the credentials, the rules associated with the connector, attributes to be included, and tag routing definitions.

A template includes the following definitions:

| Attribute | Value |
|---|---|
| **descriptionDefinition** | Defines the description to use instead of the default description defined in `Connector.config`. See [Customized description and title definitions](#customized-description-and-title-definitions) |
| **titleDefinition** | Defines the title to use instead of the default title defined in `Connector.config`. See [Customized description and title definitions](#customized-description-and-title-definitions) |
| **server** | Section — defines the address, company value, and incident URL template for the EasyVista instance |
| **address** | The address of the EasyVista instance (production or test) |
| **usesTls** | Whether the EasyVista instance requires TLS. Values: `true` or `false` (default: `true`) |
| **company** | A string representing the company, included in the URL when communicating with EasyVista |
| **viewIncidentUrlTemplate** | The URL definition used to display an incident ticket in EasyVista. This URL opens when the **Incident Ticket ID** of a job is selected in Solution Manager. See [Generate auto connection link](#generate-auto-connection-link) |
| **rules** | Section — defines which functions are supported by the connector |
| **includeJobLogAttachment** | Whether the job log should be attached to the incident ticket. Values: `true` or `false` (default: `true`) |
| **includeTagRouting** | Whether OpCon job tags should be used for incident routing. Values: `true` or `false` (default: `false`) |
| **includeCorrelationId** | Whether correlation information should be included in the submission to EasyVista. Values: `true` or `false` (default: `false`) |
| **useTitleDefinitionForDescriptionDefinition** | Whether the title definition should be used as the value for the description attribute. Values: `true` or `false` (default: `false`) |
| **credentials** | Section — defines the EasyVista login credentials |
| **user** | The EasyVista user with sufficient privileges to submit requests. Must be encrypted using `Encrypt.exe` |
| **password** | The password for the EasyVista user. Must be encrypted using `Encrypt.exe` |
| **ticketDefinitions** | Section — defines required attributes added to the JSON payload |
| **indicator** | A name that identifies this attribute to the connector |
| **attribute** | The attribute name added to the JSON payload |
| **value** | The value included with the attribute. The special value `DescriptionDefinition` inserts the generated OpCon failure message as the attribute value |
| **ticketAdditionalFields** | Section — defines optional attributes added to the JSON payload |
| **indicator** | A name that identifies this attribute to the connector |
| **attribute** | The attribute name added to the JSON payload |
| **value** | The value included with the attribute |
| **tags** | Section — defines tag routing information when `includeTagRouting` is set to `true` |
| **indicator** | Defines how the tag should be matched. Supports `TAG_END`, `TAG_START`, `EXIT`, or `DEFAULT` |
| **indicatorValue** | The value matched against the OpCon job tag (start or end of the tag) |
| **attribute** | The attribute name added to the JSON payload |
| **value** | The value associated with the attribute |

The `server` section defines the address information for the EasyVista instance. A single connector can submit requests to multiple EasyVista instances by creating separate template files. The `server` section is required.

The `rules` section defines which rules apply to the request. Values are either `true` or `false`. The `rules` section is required.

The `credentials` section defines the EasyVista user and password. These values must be encrypted using `Encrypt.exe`. The `credentials` section is required.

The `ticketDefinitions` section defines required attributes added to the payload. This includes `catalogueGui` or `catalogueCode`, `origin`, and `description`. The `description` attribute accepts the special value `DefinitionDescription`, which inserts the generated OpCon failure message as the attribute value.

The `ticketAdditionalFields` section defines optional attributes added to the payload.

The `tags` section defines incident routing when OpCon job tags are used to route tickets. This is enabled when `includeTagRouting` is set to `true`.

Tag matching is performed in the following order: `EXIT` first, then `TAG_START` and `TAG_END`, then `DEFAULT`.

| Indicator | Description |
|---|---|
| **EXIT** | Matches against the complete job tag. If matched, no ticket is created |
| **TAG_START** | Checks whether any job tag begins with the `indicatorValue` |
| **TAG_END** | Checks whether any job tag ends with the `indicatorValue` |
| **DEFAULT** | Used when no `TAG_END` or `TAG_START` match is found and tag routing is enabled |

Example template using environment variables for description and title:

```json
{
  "descriptionDefnition" : "date @EV_Date job (@EV_Job) of schedule (@EV_Schedule) running on server (@EV_Agent) failed with error code @EV_Errorcode",
  "titleDefinition" : "OpCon Task Failure (schedule @EV_Schedule job @EV_Job )",
  "server" : {
    "address" : "test-fr-vp-01.easyvista-training.com",
    "usesTls" : true,
    "company" : "50009",
    "viewIncidentUrlTemplate" : "https://test-fr-vp-01.easyvista-training.com/autoconnect_mail.php?field1=5C0F051E590F056F10&field2=&field4=%7BAF1AE6AD-FF4B-41B0-93B3-99BEF6052B12%7D&field5=ViewDialog&field6={0}&field7=RFC_NUMBER"
  },
  "rules" : {
    "includeJobLogAttachment" : true,
    "includeTagRouting" : false,
    "includeCorrelationId" : true,
    "useTitleDefinitionForDescriptionDefinition" : false
  },
  "credentials" : {
    "user" : "633231686447566a61413d3d",
    "password" : "633231686447566a614449774d6a453d"
  },
  "ticketDefinitions" : [{
      "indicator" : "catalogCode",
      "attribute" : "Catalog_Code",
      "value" : "46"
    },{
      "indicator" : "origin",
      "attribute" : "Origin",
      "value" : "WEB Service"
    },{
      "indicator" : "description",
      "attribute" : "Description",
      "value" : "DefinitionDescription"
  }],
  "ticketAdditionalFields" : [{
      "indicator" : "urgencyId",
      "attribute" : "Urgency_ID",
      "value" : "1"
  }],
  "tags" : [{
      "indicator" : "TAG_END",
      "indicatorValue" : "",
      "attributes" : [{
          "attribute" : "",
          "value" : ""
      },{
          "attribute" : "",
          "value" : ""
      }]
  },{
      "indicator" : "TAG_START",
      "indicatorValue" : "",
      "attributes" : [{
          "attribute" : "",
          "value" : ""
      },{
          "attribute" : "",
          "value" : ""
      }]
  },{
      "indicator" : "DEFAULT",
      "indicatorValue" : "DEFAULT",
      "attributes" : [{
          "attribute" : "",
          "value" : ""
      },{
          "attribute" : "",
          "value" : ""
      }]
  },{
      "indicator" : "EXIT",
      "indicatorValue" : "NOTICKET",
      "attributes" : [ {
          "attribute" : "",
          "value" : ""
      }]
  }]
}
```

When using environment variables, a `$JOB:ADD` event must be used in Notification Manager and the environment variables must be defined within the job being added by the event.

Example template using `Connector.config` values for description and title:

```json
{
  "descriptionDefnition" : "",
  "titleDefinition" : "",
  "server" : {
    "address" : "test-fr-vp-01.easyvista-training.com",
    "usesTls" : true,
    "company" : "50009",
    "viewIncidentUrlTemplate" : "https://test-fr-vp-01.easyvista-training.com/autoconnect_mail.php?field1=5C0F051E590F056F10&field2=&field4=%7BAF1AE6AD-FF4B-41B0-93B3-99BEF6052B12%7D&field5=ViewDialog&field6={0}&field7=RFC_NUMBER"
  },
  "rules" : {
    "includeJobLogAttachment" : true,
    "includeTagRouting" : false,
    "includeCorrelationId" : true,
    "useTitleDefinitionForDescriptionDefinition" : false
  },
  "credentials" : {
    "user" : "633231686447566a61413d3d",
    "password" : "633231686447566a614449774d6a453d"
  },
  "ticketDefinitions" : [{
      "indicator" : "catalogCode",
      "attribute" : "Catalog_Code",
      "value" : "46"
    },{
      "indicator" : "origin",
      "attribute" : "Origin",
      "value" : "WEB Service"
    },{
      "indicator" : "description",
      "attribute" : "Description",
      "value" : "DefinitionDescription"
  }],
  "ticketAdditionalFields" : [{
      "indicator" : "urgencyId",
      "attribute" : "Urgency_ID",
      "value" : "1"
  }],
  "tags" : [{
      "indicator" : "TAG_END",
      "indicatorValue" : "",
      "attributes" : [{
          "attribute" : "",
          "value" : ""
      },{
          "attribute" : "",
          "value" : ""
      }]
  },{
      "indicator" : "TAG_START",
      "indicatorValue" : "",
      "attributes" : [{
          "attribute" : "",
          "value" : ""
      },{
          "attribute" : "",
          "value" : ""
      }]
  },{
      "indicator" : "DEFAULT",
      "indicatorValue" : "DEFAULT",
      "attributes" : [{
          "attribute" : "",
          "value" : ""
      },{
          "attribute" : "",
          "value" : ""
      }]
  },{
      "indicator" : "EXIT",
      "indicatorValue" : "NOTICKET",
      "attributes" : [ {
          "attribute" : "",
          "value" : ""
      }]
  }]
}
```

### Customized description and title definitions

The description and title attribute values can be customized using environment variables. If using this option, the EasyVista job must be started in Notification Manager using a `$JOB:ADD` event instead of **Run Command**, because **Run Command** does not support environment variables.

OpCon properties are mapped to environment variables. Environment variable names must start with `@EV_` to indicate they are EasyVista environment variables. During connector startup, all environment variables starting with `@EV_` are loaded into the mapping table and their values are inserted into the description and title attributes.

Example template attribute values using environment variables:

```
"descriptionDefnition" : "date @EV_Date job (@EV_Job) of schedule (@EV_Schedule) running on server (@EV_Agent) failed with error code @EV_Errorcode"
"titleDefinition" : "OpCon Task Failure (schedule @EV_Schedule job @EV_Job )"
```

At runtime, the variables are resolved to their values:

```
date 2021-10-26 job (EvFailure_DEF) of schedule (EasyVista Tests) running on server (LocalWin) failed with error code 0000000001
OpCon Task Failure (schedule EasyVista Tests job EvFailure_DEF )
```

### Generate auto connection link

The `viewIncidentUrlTemplate` attribute in the `server` section of the template creates a direct link to the created incident ticket. Generate this URL in EasyVista using the following procedure.

To generate the auto connection link, complete the following steps:

1. In EasyVista, go to **Administration** > **Access Management** > **Auto Connection Link**.
2. Enter the following parameters:
   - **Form Name**: `DialogF0_Incidents`
   - **Field Name**: `RFC_NUMBER`
   - **Value**: `Incident_Number`
3. Select the **Generate Link** button.
4. Copy the generated link and paste it into the `viewIncidentUrlTemplate` field in the template file.
5. In the pasted URL, replace the `Incident_Number` portion with `{0}`:
   - Before: `field5=ViewDialog&field6=Incident_Number&field7=RFC_NUMBER`
   - After: `field5=ViewDialog&field6={0}&field7=RFC_NUMBER`

### OpCon Notification Manager definition

Notification Manager triggers the EasyVista Connector when a job completes with a failure condition. Using Notification Manager allows multiple jobs to be covered by a single rule instead of defining a failure event on each individual job.

If using environment variables to customize the description and title fields, use a `$JOB:ADD` event instead of **Run Command**, because **Run Command** does not support environment variables.

#### Configuring a `$JOB:ADD` event

To configure a `$JOB:ADD` event in Notification Manager, complete the following steps:

1. In Notification Manager, select the **Jobs** tab.
2. Select the **Add Group** button and create a new group called `EasyVista`.
3. Right-click the **EasyVista** group and select **Add Job Trigger**.
4. In the **Add Job Trigger** dialog, select **Job Failed**.
5. In the **Notification Definitions** section, select **Send OpCon/xps Events**.
6. Select the **Event** tab, select the **Add event** button, and select the **`$JOB:ADD`** template.
7. Insert the following definition:

```
[[$SCHEDULE DATE]],AdHoc,EASY-VISTA,DAILY,ID=[[$JOBID]];MACH=[[$MACHINE NAME]];SCHED=[[$SCHEDULE NAME]];JOB=[[$JOB NAME]];ECODE=[[$JOB TERMINATION]];SID=[[$SCHEDULE ID]];SINST=[[$SCHEDULE INST]],Y
```

#### Configuring Run Command

To configure a Run Command in Notification Manager, complete the following steps:

1. In Notification Manager, select the **Jobs** tab.
2. Select the **Add Group** button and create a new group called `EasyVista`.
3. Right-click the **EasyVista** group and select **Add Job Trigger**.
4. In the **Add Job Trigger** dialog, select **Job Failed**.
5. Select the **Run Command** tab.
6. In the **Command** field, enter the following, replacing paths as appropriate for your environment:

```
C:\Connectors\EasyVista\EasyVista.exe -a [[$MACHINE NAME]] -s [[$SCHEDULE NAME]] -jn [[$JOB NAME]] -e [[$JOB TERMINATION]] -sd [[$SCHEDULE DATE-EVIS]] -si [[$SCHEDULE ID]] -sn [[$SCHEDULE INST]] -t bas_easyvista.json
```

| Argument | Resolves to |
|---|---|
| `-a [[$MACHINE NAME]]` | The agent name |
| `-s [[$SCHEDULE NAME]]` | The schedule name |
| `-jn [[$JOB NAME]]` | The job name |
| `-e [[$JOB TERMINATION]]` | The job termination code |
| `-sd [[$SCHEDULE DATE-EVIS]]` | The date in `YYYY-MM-DD` format |
| `-si [[$SCHEDULE ID]]` | The schedule ID |
| `-sn [[$SCHEDULE INST]]` | The schedule instance |
| `-t bas_easyvista.json` | The template file in the `templates` directory to use |

7. In the **Working Directory** field, enter `C:\Connectors\EasyVista`.
8. In the **Batch User** field, select **Use Service Account**.

#### Definition of the EASY-VISTA job in the OpCon AdHoc schedule

If the `$JOB:ADD` option was selected when configuring Notification Manager, add the `EASY-VISTA` job to the **AdHoc** schedule.

To create the AdHoc job, complete the following steps:

1. Create a Windows job associated with the agent where the EasyVista Connector is installed.
2. In the **Command Line** field, enter the full path to the EasyVista Connector executable and the required arguments:

```
C:\Connectors\EasyVista\EasyVista.exe -a [[JI.MACH]] -s [[JI.SCHED]] -jn [[JI.JOB]] -e [[JI.ECODE]] -sd [[$SCHEDULE DATE-EVIS]] -si [[JI.SID]] -sn [[JI.SINST]] -t template
```

3. Set the `-t` argument (template) to the template file name defined for this EasyVista instance. The template file must be placed in the `templates` directory of the EasyVista installation.
4. In the **Working Directory** field, enter the full path to the EasyVista Connector installation directory.
5. Create a frequency named `ALLDAYS` that allows the job to be scheduled 7 days a week.
6. Select the **Disable Build** option.
7. Select the **Allow Multi-Instance** option.
8. Add the required environment variables matching the `descriptionDefinition` and `titleDefinition` fields in the template.

## Security considerations

All user and password values in `Connector.config` and template files must be encrypted using the `Encrypt.exe` utility. The `TOKEN` value in the `[OPCON API]` section must also be encrypted. The OpCon REST API must be configured to use TLS (`USES_TLS=True`).

## FAQs

**What is the `Encrypt.exe` utility and when do I use it?**
`Encrypt.exe` is a utility included with the connector that applies 64-bit encryption to sensitive values. Use it to encrypt user credentials and the OpCon API token before placing them in `Connector.config` or a template file. Run `Encrypt.exe -v <value>` to display the encrypted output, then copy it into the configuration.

**Can the connector connect to multiple EasyVista instances?**
Yes. Create a separate template file for each EasyVista instance. When configuring Notification Manager or the AdHoc job, specify which template to use with the `-t` argument. Each template file can point to a different EasyVista address, company, and credentials.

**What is the `$SCHEDULE DATE-EVIS` property and why is it required?**
`$SCHEDULE DATE-EVIS` is a global OpCon property that stores the schedule date in `yyyy-MM-dd` format. The EasyVista Connector requires this date format. Create this property in OpCon and set its value to `yyyy-MM-dd` before configuring Notification Manager.

**What is the difference between `$JOB:ADD` and Run Command in Notification Manager?**
**Run Command** runs the connector directly with command-line arguments. **`$JOB:ADD`** adds a Windows job to the AdHoc schedule that then runs the connector. Use `$JOB:ADD` when you need to pass environment variables to customize the description and title definitions, since **Run Command** does not support environment variables.

**What does the `EXIT` tag do?**
When tag routing is enabled (`includeTagRouting: true`), the `EXIT` tag causes the connector to exit without creating an incident ticket if an OpCon job contains a matching tag value. This is useful for suppressing tickets for specific jobs that should not create EasyVista incidents when they fail.

**Where can I find the OpCon application token?**
See the OpCon REST API documentation for instructions on generating an application token. Once generated, encrypt the token value using `Encrypt.exe` before adding it to `Connector.config`.

## Glossary

**`Connector.config`** — The main configuration file for the EasyVista Connector. It defines the OpCon API connection, proxy settings, debug mode, and default incident description and title templates.

**Template** — A JSON configuration file in the `templates` directory that defines the EasyVista instance connection, credentials, rules, and ticket attributes for one EasyVista environment.

**`Encrypt.exe`** — The encryption utility included with the EasyVista Connector. All credential values and the OpCon API token must be encrypted using this utility before being placed in configuration files.

**`$SCHEDULE DATE-EVIS`** — A global OpCon property storing the schedule date in `yyyy-MM-dd` format. Required by the EasyVista Connector for date formatting in incident descriptions.

**Tag routing** — A feature controlled by the `includeTagRouting` rule in the template. When enabled, the connector uses OpCon job tags to route incident tickets to specific EasyVista queues or to suppress ticket creation.

**`EXIT` tag** — A special tag routing indicator that prevents the connector from creating an incident ticket when a matching OpCon job tag is found.

**`viewIncidentUrlTemplate`** — A URL pattern in the template's `server` section that generates a direct link to an EasyVista incident ticket. The connector replaces `{0}` in the URL with the incident number at runtime.

**AdHoc schedule** — An OpCon schedule used to run on-demand jobs. The EASY-VISTA job is added to the AdHoc schedule when using the `$JOB:ADD` Notification Manager configuration.

**Related topics:**

- [Overview](./overview.md)
- [Operation](./operation.md)
