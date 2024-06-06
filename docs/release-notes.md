# Release Notes EasyVista

## General

This release of EasyVista is for opCon System 21.0 or greater. The connector only supports connections to the OpCon-API to retrieve job information and insert or update Incident information. 

## Release 21.0.4

### Migration Considerations

When upgrading to the latest release and tag routing is enabled, the DEFAULT tag value must be defined, otherwise the connector will terminate with the following message :
**Connector terminated due to configuration error - tag routing enabled, but no default routing tag defined **.

### New Features

**CONNUTIL-639**    
                    Implemented new tag type called **EXIT** to exit connector and not create an incident ticket when a matching job tag is encountered. Only works when tag routing enabled.  

                    To implement this, the existing EasyVista template must be updated to include an EXIT tag in the tags section of the template and the indicatorValue consist of a tag name thst will indicate no ticket should be created. In the example below, if the OpCon job contains a tag of NOTICKET then a ticket will not be created.

                    ```
                      "tags" : [ {
                            "indicator" : "TAG_START",
                            "indicatorValue" : "ABC",
                            "attributes" : [ {
                                "attribute" : "attribute-name",
                                "value" : "attribute value"
                            }]
                        },{
                            "indicator" : "TAG_END",
                            "indicatorValue" : "XYZ",
                            "attributes" : [ {
                                "attribute" : "attribute-name",
                                "value" : "attribute value"
                            }]
                        },{
                            "indicator" : "EXIT",
                            "indicatorValue" : "NOTICKET",
                            "attributes" : [ {
                                "attribute" : "",
                                "value" : ""
                            }]
                        },{
                            "indicator" : "DEFAULT",
                            "indicatorValue" : "DEFAULT",
                            "attributes" : [ {
                                "attribute" : "attribute-name",
                                "value" : "attribute value"
                            }]
                        }]

                    ```
                    In the above example, a incident ticket will not be created if the OpCon job has a job tag of NOTICKET.
                    The connector will be exit with the following message :
                    **Ticket creation terminated for the job EASYVISTA-TEST.JOB001 as EXIT tag defined**. 

### Fixes

**CONNUTIL-633**    
                    Fixed the order of the files as they appear in the EasyVista ticket to match the same order extracted from OpCon.

**CONNUTIL-640**    
                    Add check for DEFAULT routing tag if tag routing enabled. If DEFAULT tag not defined, terminate the connector displaying configuration error message.

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





