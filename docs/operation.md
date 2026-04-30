---
title: EasyVista Connector operation
description: "Monitor and manage the EasyVista Connector in production, including how to view incident ticket status directly from OpCon Solution Manager."
sidebar_label: 'Operation'
tags:
  - Procedural
  - Operations Staff
  - System Administrator
---

# EasyVista Connector operation

## What is it?

Once the EasyVista Connector is installed and configured, day-to-day operation requires minimal intervention. The connector runs automatically in response to job failures triggered by Notification Manager. The primary operational task is viewing the status of EasyVista incident tickets directly from OpCon using Solution Manager.

Use this page when you need to:

- View an EasyVista incident ticket linked to a failed OpCon job
- Access the EasyVista incident details without leaving Solution Manager

## Viewing incident status

You can display the status of an EasyVista incident directly from OpCon using Solution Manager.

To view an incident ticket, complete the following steps:

1. In Solution Manager, locate the failed job in the Operations view.
2. Right-click the job and select the job from the context menu to open the **Job Selection** dialog.
3. In the **Job Selection** dialog, select the **Incident Ticket ID** value. The EasyVista login page or incident details open in a browser.
4. If prompted, enter your EasyVista username and password. The EasyVista incident ticket is displayed.

## Security considerations

Access to EasyVista incident tickets requires valid EasyVista credentials. The EasyVista user account used by the connector must have sufficient privileges to create and update incidents in EasyVista. User credentials stored in the template file must be encrypted using the `Encrypt.exe` utility provided with the connector.

## FAQs

**What do I do if the Incident Ticket ID field is empty for a failed job?**
If the **Incident Ticket ID** field is empty, the connector may not have run for that job failure. Check the connector log files in the `log` directory of the connector installation for error messages. Verify that the Notification Manager rule is configured correctly and that the agent is available.

**Can I view an incident ticket without leaving OpCon?**
Selecting the **Incident Ticket ID** value in the Job Selection dialog opens the EasyVista incident in a browser using the `viewIncidentUrlTemplate` defined in the connector template. This requires that the URL template is correctly configured in the template file.

**Where are the connector log files located?**
The connector writes log files to the `log` subdirectory of the connector installation directory. Enable debug mode by setting `DEBUG=ON` in the `[GENERAL]` section of `Connector.config` to capture additional detail for troubleshooting.

**What happens if the EasyVista system is unavailable when a job fails?**
If the EasyVista REST API is unreachable, the connector will fail to create the incident ticket and log an error. The OpCon job failure notification will still fire, but no incident ticket will be created. You will need to create the ticket manually or rerun the connector once EasyVista is available.

## Glossary

**Incident Ticket ID** — The EasyVista incident number written back into the OpCon job record after a ticket is created. Selecting this value in Solution Manager opens the corresponding EasyVista incident.

**Job Selection dialog** — The dialog in Solution Manager that appears when you right-click a job in the Operations view. It provides access to job details including the **Incident Ticket ID** field.

**`viewIncidentUrlTemplate`** — The URL pattern defined in the connector template that generates a direct link to an EasyVista incident ticket. The connector substitutes `{0}` with the incident number at runtime.

**Debug mode** — A connector setting that enables verbose logging. Set `DEBUG=ON` in `Connector.config` to capture detailed information for troubleshooting.

**Related topics:**

- [Overview](./overview.md)
- [Installation](./installation.md)
