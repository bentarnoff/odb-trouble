# ODB TROUBLESHOOTING GUIDE

## Using BOSH and CF to access logs and SSH

1. Follow the steps described here to [target your BOSH CLI](https://docs.pivotal.io/pivotalcf/1-9/customizing/trouble-advanced.html#prepare) and [log-in to your CF CLI](https://docs.cloudfoundry.org/cf-cli/getting-started.html)

### Accessing broker logs and VM(s)

1. Identify the ODB deployment using `bosh deployments`
1. Download the ODB manifest using `bosh download manifest <odb-deployment-name> odb.yml`
1. Select the ODB deployment using `bosh deployment odb.yml`
1. View VMs in the deployment using `bosh instances`
1. Download broker logs by using the `bosh logs <instance-id>` command

### Accessing service instance logs and VM(s)

1. To target an individual service instance deployment, retrieve the 
the guid of your service instance with the CF CLI command `$ cf service {YOUR_SERVICE} --guid`

1. Pre-pend the text "service_instance-" to the guid you just obtained to create the ID that BOSH uses for your service instance (e.g. if your guid is `1Hy6H`), this becomes `service_instance-1Hy6H`).

1. Download your bosh manifest for the service using `$ bosh download manifest service_instance-guid myservice.yml`
1. Select the deployment using `bosh deployment myservice.yml`
1. View VMs in the deployment using `bosh instances`
1. Download broker logs by using the `bosh logs <instance-id>` command
1. To ssh onto the VM use `bosh ssh <instance-id>`

## Viewing resource saturation and scaling resources

Once a deployment has been selected, you can use the `bosh vms --vitals` or the `bosh instances --vitals` commands to view current resource utilization. You can also view process level information by using `bosh instances --ps`.

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

The `task-id` field maps to the BOSH task id. For further information on a failed BOSH task, use the `bosh task <task-id>` command in the BOSH CLI.

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