# ODB TROUBLESHOOTING GUIDE

## Accessing logs and identifying problems
## Connecting to a problematic machine
## Viewing resource usage
## Parsing a CF error message

Failed operations (create, update, bind, unbind, delete) will result in an error message that can be retrieved using the CF cli

```bash hl_lines="14 15 16 17 18 19 20"
$ cf service myservice

Service instance: myservice
Service: super-db
Bound apps:
Tags:
Plan: dedicated-vm
Description: Dedicated Instance
Documentation url:
Dashboard: 

Last Operation
Status: create failed
Message: Instance provisioning failed: There was a problem completing your request. 
		 Please contact your operations team providing the following information: 
		 service: redis-acceptance, 
		 service-instance-guid: ae9e232c-0bd5-4684-af27-1b08b0c70089,
		 broker-request-id: 63da3a35-24aa-4183-aec6-db8294506bac, 
		 task-id: 442, 
		 operation: create
Started: 2017-03-13T10:16:55Z
Updated: 2017-03-13T10:17:58Z
```
The information under the "Message" field can be used to debug further. Please provide this information to Pivotal Support when filing a ticket.

The `task-id` field maps to the BOSH task id. For further information on a failed BOSH task, use the `bosh task <task-id>` command in the [BOSH CLI](https://docs.pivotal.io/pivotalcf/1-9/customizing/trouble-advanced.html#cli).

The `broker-request-guid` maps to the portion of the On-Demand Broker log containing the failed step. Access the broker log via your syslog aggregator, or access bosh logs for the broker by typing `bosh logs broker 0`. If you have more than one broker instance, you will need to repeat this process for each instance.

## Common Errors
### BOSH problems
### Service Broker configuration
### Authentication
### Networking
### Quotas
## Cleaning up stale references manually
## Filing a support ticket
## Monitoring recommendations
## Knowledge Base (Community)