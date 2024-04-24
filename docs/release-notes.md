# Release Notes EasyVista

## General

This release of EasyVista is for opCon System 21.0 or greater. The connector only supports connections to the OpCon-API to retrieve job information and insert or update Incident information. 

## Release 21.0.4

### New Features

**CONNUTIL-634**    
                    Implemented a new rule **doNotCreateTicketIfNoTagExistsAndTagRoutingEnabled** to not submit a ticket if tag routing is enabled and no tags exists for the job. The default value for the rule is false.

                    To implement this rule, the existing EasyVista template must be updated including the new doNotCreateTicketIfNoTagExistsAndTagRoutingEnabled rule in the rules section of the template.

                    ```
                    "rules" : {
                        "includeJobLogAttachment" : true,
                        "includeTagRouting" : false,
                        "includeCorrelationId" : true,
                        "useTitleDefinitionForDescriptionDefinition" : false,
                        "doNotCreateTicketIfNoTagExistsAndTagRoutingEnabled" : true
                    },
                    ```

### Fixes

**CONNUTIL-633**    
                    Fixed the order of the files as they appear in the EasyVista ticket to match the same order extracted from OpCon.

## Release 21.0.3

### New Features

**CONNUTIL-629**    
                    Correctly a problem where Unix job logs were not correctly uploaded to the EasyVista Incident.

### Fixes


## Release 21.0.2

### New Features

**CONNUTIL-575**    
                    Altered WS Client to ignore unknown API fields

### Fixes





