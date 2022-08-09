# Installation

The EasyVista Connector installation consists of multiple steps that are required to complete the installation successfully. 

The connector requires a SMA OpCon Windows Agent to provide the connection between Notification Manager in the OpCon System and the EasyVista Connector software. 
It requires the OpCon Rest-API to extract the unique id of the task in the daily tables and the retrieval of the task's job log.
It uses the OpCon connection to update the returned incident number in the task in the Daily tables.

## Supported Software Levels
The following software levels are required to implement the Asysco AMT Connector.

- OpCon Release 21.0 or higher.
- Embedded Java OpenJDK 11 (part of installation).
- OpCon Rest API Configured to use TLS.
- OpCon Windows Agent to provide link to EasyVista Connector.
- OpCon Notification Manager.
- An EasyVista implementation that supports the Rest API.

## Installation
The installation process consists of the following steps:

- OpCon Windows Agent Installation.
- EasyVista Connector Installation.
- EasyVista Connector Configuration.
- OpCon Notification Manager Definition
 
### OpCon Windows Agent Installation
Copy the supplied install file SMAEasyVistaConnector-win.zip and extract it into the installation directory.

After the installation is complete, the root installation directory contains the connector executable (EasyVista.exe), the encryption software executable (Encrypt.exe), the Connector.config file and four directories, java, joblogs, templates and log. The java directory contains the java software required to execute the connector (OpenJDK 11), the joblogs directory is used to temporarily contain job logs extracted from the OpCon system, the log directory contains the connector log files and the templates directory contains the EasyVista template files.

### EasyVista Connector Installation
The EasyVista connector can be installed on the OpCon Windows Server.
Copy the downloaded install file SMAEasyVistaConnector-win.zip and extract it into a temp directory (c:\temp). Extract the information including sub-directories into the required directory.

#### Create $SCHEDULE DATE-EVIS Global Property
Create the special **$SCHEDULE DATE-EVIS** global property that contains the schedule date in the yyyy-MM-dd format from the standard $SCHEDULE DATE property. Set the value to **yyyy-MM-dd**.

### EasyVista Connector Configuration
The configuration of the EasyVista Connector requires setting the OpCon connection information for the OpCon system associated with the connector.

All user and password values placed in the configuration and template files must be encrypted using the Encrypt.exe utility provided with the connector. 

#### Encrypt Utility
The Encrypt utility uses standard 64 bit encryption.

Supports a -v argument and displays the encrypted value

On Windows, example on how to encrypt the value "abcdefg":

```
Encrypt.exe -v abcdefg

```

#### Connector.config configuration
Configure the Connector.config file in the installation directory setting the required information.
The Connector.config contains the following values

Property Name | Value
--------- | -----------
**[GENERAL]**                       | header
**JOBLOGDIR**                       | The name of the directory where the retrieved log files are stored. After successful attachment to the EasyVista Incident, the log file is deleted. The name is a sub directory of the installation directory (default joblogs).
**TEMPLATESDIR**                    | The name of the directory where the template definitions are stored. The name is a sub directory of the installation directory (default templates).
**DEBUG**                           | The Connector supports a debug mode which can be enabled by setting the value to ON. The connector should be run with DEBUG disabled (OFF) and enabled (ON) when requested to capture an error condition. Value either ON or OFF (default OFF).
**[PROXY SERVER]**                  | header - Used to define the full URL of a proxy server if required
**USES_PROXY**                      | Indicates if this connector should use a Proxy Server connection. Values are True or False (default False).
**ADDRESS**                         | The address of the Proxy Server.
**PORT**                            | The port of the Proxy Server.
**[OPCON API]**                     | header - Used to define the connection to the OpCon API
**ADDRESS**                         | The server address of the OpCon API.
**PORT**                            | The port number used by the OpCon API server.
**USES_TLS**                        | Must be set to True.
**TOKEN**                           | An application token used for Authentication when communicating with the OpCon-API. 
**[TICKET DEFINITIONS]**            | header - Used to define information to be used when creating a ticket.
**DESCRIPTION**                     | The text that is used when creating an incident (description field). Default value  OpCon Task Failure ( date {0} schedule {1} job {2} server {3} error code {4} ). The text is customizable. When a value is required in the text use the {n} syntax  where n is the value of the parameter defined in the table below. 0 is the schedule date when the error occurred (resolves to [[$SCHEDULE DATE-EVIS]] value), 1 is the workflow name (resolves to [[JI.SCHED]] value), 2 is the task name (resolves to [[JI.JOB]] value), 3 is the server name where the job executed (resolves to [[JI.MACH]] value) and 4 is the termination code (resolves to [[JI.ECODE]] value). 
**TITLE**                           | The text that is used when creating an incident (description field). Default value  OpCon Task Failure (schedule {1} job {2}) The text is customizable. When a value is required in the text use the {n} syntax  where n is the value of the parameter defined in the table below. 0 is the schedule date when the error occurred (resolves to [[$SCHEDULE DATE-EVIS]] value), 1 is the workflow name (resolves to [[JI.SCHED]] value), 2 is the task name (resolves to [[JI.JOB]] value), 3 server name where the job executed (resolves to [[JI.MACH]] value) and 4 the termination code (resolves to [[JI.ECODE]] value).

Example configuration file. 

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
The OPCON API section provides the information about connecting to the OpCon System using the OpCon Rest-API so the job log can be retrieved from the OpCon task. The SERVER, PORT USES_TLS and TOKEN statements provide the definitions that will allow the EasyVista connector to connect to the OpCon Rest API. The TOKEN statement contains an application token (see OpCon Rest-API documentation on how to generate an application token).

#### Templates
Templates provide information about the EasyVista connection and the definitions that will be submitted in the JSON payload as part of the request. A default template can be found in the templates directory of the connector installation.

The template includes the address of the EasyVista instance, the credentials, the rules associated with the connector, attributes to be included and tag routing definitions.

A template includes the following definitions:

Attribute Name Name | Value
--------- | -----------
**descriptionDefinition**	                   | Defines the description to use instead of the default description defined in the Connector.config. See section on Customized Description and Title Definitions.
**titleDefinition**                            | Defines the title to use instead of the default description defined in the Connector.config. See section on Customized Description and Title Definitions.
**server**                                     | header - Defines the address, the company value and the template to display the incident associated with the EasyVista Instance
**address**                                    | The address associated with the EasyVista instance (i.e. production, test)
**usesTls**                                    | If the EasyVista instance requires TLS. Value either true or false (default true).
**company**                                    | A string representing the company which is included in the URL when communicating with EasyVista. 
**viewIncidentUrlTemplate**                    | The URL definition that can be used display the incident ticket in EasyVista. This url is invoked when the Incident Ticket ID of the job is selected in Solution Manager. See Generate AutoConnection Link section for information on generating this value.
**rules**                                      | header - Defines what functions are supported by the connector.
**includeJobLogAttachment**                    | Indicates if the job log should be attached to the incident ticket. Value either true or false (default true).
**includeTagRouting**                          | Indicates if User defined tags should be used for incident routing purposes. Value either true or false (default false). 
**includeCorrelationId**                       | Indicates if the correlation information should be included in the information submitted to EasyVista. Value either true or false (default false).
**useTitleDefinitionForDescriptionDefinition** | Indicates if the title definition should be used for the value of the description attribute. Value either true or false (default false).
**credentials**                                | header	
**user**                                       | The user which has the required privileges to connect to the EasyVista System to submit requests. The name must be encrypted using the Encrypt.exe utility.
**password**                                   | The password of the user which has the required privileges  to connect to the EasyVista System to submit requests. The password must be encrypted using the Encrypt.exe utility .
**ticketDefinitions**	                       | header - Defines attributes that will added to the JSON payload. 
**indicator**                                  | A name that identifies this attribute to the connector.
**attribute**	                               | Defines the attribute name that will be added to the JSON payload along with the value.
**value**	                                   | The value that will be included with the attribute. Note that a special value DescriptionDefinition can be used which will insert the generated OpCon fault message as the value for the attribute.  
**ticketAdditionalFields**                     | header - Defines additional attributes that will added to the JSON payload. See EasyVista REST-API documentation for additional fields that can be included in the JSON payload.
**indicator** 	                               | A name that identifies this attribute to the connector. 
**attribute**	                               | Defines the attribute name that will be added to the JSON payload along with the value.
**value**	                                   | The value that will be included with the attribute. Note that a special value DescriptionDefinition can be used which will insert the generated OpCon fault message as the value for the attribute.  
**tags**                                       | header - Defines information if OpCon User defined Tags are to be used to include attributes in the ServiceNow submission. This is enabled if the rule **includeTagRouting** is set to True.
**indicator**                                  | Defines what part of the tag should be used to identify the request. Supports TAG_END,  TAG_START or DEFAULT. The DEFAULT value is used if there is no TAG_END or TAG_START match and TAG Routing is enabled.
**indicatorValue**	                           | The value that is matched to the OpCon User defined tag (either the end or the start).
**attribute**	                               | Defines the attribute name that will be added to the JSON payload along with the value.
**value**                                      | The value associated with the attribute.

The server section of the template defines the address information associated with the EasyVista Instance. This allows a single connector the ability to submit requests to multiple EasyVista instances by simply creating multiple templates. The server section is required.

The rules section of the template defines which rules should be used for the request. Values are either true or false. The rules section is required.

The credentials section of the template defines the EasyVista user and password to be used for the connection. These values must be encrypted using the Encrypt.exe program. The credentials section is required.

The ticketDefinitions section defines required attributes that will be added to the payload. This includes either catalogueGui or catalogueCode, origin and description. The description attribute has a special value ‘DefinitionDescription’ which inserts the generated OpCon failure error message as the attribute value.

The ticketAdditionalFields section defines optional attributes that will be added to the payload. 

The tags section defines incident routing information if OpCon Job User defined tags are used to route tickets. This functionality is enabled when the includeTagRouting rule is set to true. Routing can be done by defining strings that either start (TAG_START) or end (TAG_END) a tag. If no values are found and the **includeTagRouting** rule is set to true,  then the DEFAULT value will be used. It is possible to add multiple attributes for a routing condition.

```
{
  "descriptionDefnition" : "date @EV_Date job (@EV_Job) of schedule (@EV_Schedule) running on server (@EV_Agent) failed with error 
     code @EV_Errorcode",
  "titleDefinition" : "OpCon Task Failure (schedule @EV_Schedule job @EV_Job )",
  "server" : {
    "address" : "test-fr-vp-01.easyvista-training.com",
    "usesTls" : true,
	"company" : "50009",
	"viewIncidentUrlTemplate" : "https://test-fr-vp-01.easyvista-
          training.com/autoconnect_mail.php?field1=5C0F051E590F056F10&field2=&field4=%7BAF1AE6AD-FF4B-41B0-93B3- 
          99BEF6052B12%7D&field5=ViewDialog&field6={0}&field7=RFC_NUMBER"
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
  }]
}

```
The above sample template uses environment variables for the description and title definitions. When using this implementation, a $JOB:ADD event must be used and the environment variables defined within the job being added by the event.

```
{
  "descriptionDefnition" : "",
  "titleDefinition" : "",
  "server" : {
    "address" : "test-fr-vp-01.easyvista-training.com",
    "usesTls" : true,
	"company" : "50009",
	"viewIncidentUrlTemplate" : "https://test-fr-vp-01.easyvista-
          training.com/autoconnect_mail.php?field1=5C0F051E590F056F10&field2=&field4=%7BAF1AE6AD-FF4B-41B0-93B3- 
          99BEF6052B12%7D&field5=ViewDialog&field6={0}&field7=RFC_NUMBER"
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
  }]
}

```
The above sample template uses values in the config file for the description and title definitions. 

### Customized Description and Title Definitions
The description and title attribute values can be customized by using environment variables. If using this option, the EasyVista task must be started in Notification Manager by using a $JOB:ADD event instead of Run Command as Run Command does not support Environment Variables.

The OpCon properties are mapped to environment variables and these environment variables are then used in the message. When defining these environment variables the names must start with the value @EV_ indicating that the environment variable is an EasyVista environment variable. During connector startup, any environment variables starting with @EV_ are loaded into the mapping table. These values associated with the variables are then inserted into the values of the description and title attributes.

```
  "descriptionDefnition" : "date @EV_Date job (@EV_Job) of schedule (@EV_Schedule) running on server (@EV_Agent) failed with error 
     code @EV_Errorcode",
  "titleDefinition" : "OpCon Task Failure (schedule @EV_Schedule job @EV_Job )",
```
Looking at the above extract from the template, the variables in the attribute value of the description attribute will change as follows:

```  
date @EV_Date job (@EV_Job) of schedule (@EV_Schedule) running on server (@EV_Agent) failed with error code @EV_Errorcode
OpCon Task Failure (schedule @EV_Schedule job @EV_Job )

date 2021-10-26 job (EvFailure_DEF) of schedule (EasyVista Tests) running on server (LocalWin) failed with error code 0000000001
OpCon Task Failure (schedule EasyVista Tests job EvFailure_DEF )
```
### Generate Auto Connection Link
The viewIncidentUrlTemplate attribute of the server section in the template, is used to create a direct link to the created Incident ticket. The link can be generated for the environment by using the following procedure within EasyVista.

- Select Administration > Access Management > Auto Connection Link in the EasyVista menu.
- Enter the following parameters
  - Form Name = DialogF0_Incidents
  - Field Name = RFC_NUMBER
  - Value = Incident_Number
- Click Generate Link
- After the link has been generated, copy and then paste the generated link into the template changing the Incident_Number to {0}. 
  (field5=ViewDialog&field6=Incident_Number&field7=RFC_NUMBER)
  (field5=ViewDialog&field6={0}&field7=RFC_NUMBER)

### OpCon Notification Manager Definition
Notification Manager is used to execute the EasyVista Connector when a task completes with a failure condition. Using this approach allows the tasks to be added to the rule instead of defining a failure event on every task. If using environment variables to customize the description and title fields, a $JOB:Add event must be used instead of Run Command as the Run Command option does not support environment variables.

#### Configuring $JOB:ADD event
Using Notification Manager, select the Jobs tab and create a new Group called EasyVista.
Once the Group has been created, select the EasyVista Group, perform a ‘right-click’ and select Add Job Trigger. In the Add Job Trigger selection, select Job Failed.

In the Notification Definitions section, select Send OpCon/xps Events. In the Event tab, select Add event and select the $JOB:ADD template.

Insert the following definition to perform a $JOB:ADD of the Windows task that will execute the EasyVista Connector.

```
[[$SCHEDULE DATE]],AdHoc,EASY-VISTA,DAILY,ID=[[$JOBID]];MACH=[[$MACHINE NAME]];SCHED=[[$SCHEDULE NAME]];JOB=[[$JOB NAME]];ECODE=[[$JOB TERMINATION]];SID=[[$SCHEDULE ID]];SINST=[[$SCHEDULE INST]],Y
```
#### Configuring Run Command
Using Notification Manager, select the Jobs tab and create a new Group called EasyVista.
Once the Group has been created, select the EasyVista Group, perform a ‘right-click’ and select Add Job Trigger. In the Add Job Trigger selection, select Job Failed.

In the Run Command tab, enter the following:

**Command**			       C:\Connectors\EasyVista\EasyVista.exe -a [[$MACHINE NAME]] -s [[$SCHEDULE NAME]] -jn [[$JOB NAME]] -e [[$JOB TERMINATION]] -sd [[$SCHEDULE DATE-EVIS]] -si [[$SCHEDULE ID]] -sn [[$SCHEDULE INST]] -t bas_easyvista.json

                           Where 
                           C:\Connectors\EasyVista\EasyVista.exe 	is the location of the connector.
                           -a [[$MACHINE NAME]]		                resolves to the agent name
                           -s [[$SCHEDULE NAME]] 		            resolves to the schedule name
                           -jn [[$JOB NAME]]			            resolves to the job name
                           -e [[JOB TERMINATION]]		            resolves to the job termination code
	                       -sd [[$SCHEDULE DATE-EVIS]]	            resolves to the date (format YYYY-MM-DD)
	                       -si [[$SCHEDULE ID]]		                resolves to the schedule ID
	                       -sn [[$SCHEDULE INST]]		            resolves to the schedule instance
	                       -t bas_easyvista.json		            which template in the templates folder to use

**Working Directory**		C:\Connectors\EasyVista

**Batch User**			    Use Service Account 	                The batch User under which the task will be run.

#### Definition of EASY-VISTA task in OpCon AdHoc workflow
If the $JOB:ADD option was selected when configuring Notification Manager, add the EASY-VISTA task to the AdHoc schedule.
Create a Windows Job, associated with the agent that the EasyVista Connector was installed on. 

- Set the Command Line, inserting the full path name of the EasyVista Connector executable and the required arguments. 
  C:\Connectors\EasyVista\EasyVista.exe -a [[JI.MACH]] -s [[JI.SCHED]] -jn [[JI.JOB]] -e [[JI.ECODE]] -sd [[$SCHEDULE DATE-EVIS]] -si [[JI.SID]] -sn [[JI.SINST]] -t template
- Set the -t option (template) to the template value defined for this EasyVista instance. The template must be placed in the templates directory of the EasyVista installation.
- Set the working directory to the full path of the EasyVista Connector installation directory
- Create a frequency called ALLDAYS allowing the task to be scheduled 7 days a week.
- Select Disable Build.
- Select Allow Multi-Instance.
- Add the required environment Variables matching the definitions in the descriptionDefinition and titleDefinition fields of the template.
