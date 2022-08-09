---
slug: '/'
sidebar_label: 'EasyVista Connector'
---

# EasyVista Connector

The EasyVista Connector can be used to submit incident tickets automatically to EasyVista from OpCon when a task has an error condition.

![EasyVista Component Overview](/img/easyvista-component-overview.png)

The OpCon EasyVista implementation includes components that detect when a task errors, create the EasyVista Incident record (includes adding task job log) and inserts the incident number into the task in the Daily tables.

- Notification Manager 		An OpCon feature that initiates the EasyVista Connector when a task encounters an error condition.
- Windows Agent		        An OpCon Windows Agent that is used to execute the EasyVista connector.
- EasyVista Connector		An OpCon connector that communicates with EasyVista through the EasyVista Rest-API to create an incident.
- Config				    Defines connection to the OpCon System.
- Templates			        Contains the EasyVista definitions to be used for that connection.

## Process
The ticket creation process consists of the following steps.

![Ticket Creation Process](/img/ticket-creation-process.png)

1.	The EasyVista Connector is executed by the Notification Manager when a failed task is detected. The Notification Manager uses the following standard OpCon properties to pass information to the EasyVista Connector.

```
$MACHINE NAME		The name of the Agent on which the task was executing.
$JOB TERMINATION	The termination code of the task.
$SCHEDULE DATE-EVIS	A special version of the Schedule Date format created to support EasyVista Connector.
$SCHEDULE ID		The schedule ID of the workflow in the OpCon System.
$SCHEDULE INST		The schedule instance of the workflow in the Daily OpCon table.
$SCHEDULE NAME		The name of the workflow.
$JOB NAME		    The name of the task.
```
2.	Before creating a new incident ticket, the EasyVista Connector checks to see if an incident ticket has already been created for the task. The task information is extracted from the OpCon Daily Job table.
3.	If an incident ticket exists, a new ticket is created. The previous ticket number is attached to the task documentation and the incident ticket description. When creating an incident ticket, the workflow name, the task name, the agent the task was executing on and the termination code are included in the incident description. 
4.	The returned incident number is written into the OpCon task in the task Incident Ticket ID field. If there was an existing ticket and a new ticket was created, the previous ticket number is written into the task documentation field.
5.	The EasyVista Connector calls the OpCon Rest-API to retrieve the task’s job log and attaches this to the created or existing EasyVista Incident ticket. The job log of the failed task can be accessed by double-clicking on the file definition in the Attachments sections of the created incident ticket.
