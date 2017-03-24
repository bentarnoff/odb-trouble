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

## Smoke Tests, Checks, and Errands
## Viewing resource saturation and scaling resources

Once a deployment has been selected, you can use the `bosh vms --vitals` or the `bosh instances --vitals` commands to view current resource utilization. You can also view process level information by using `bosh instances --ps`.

## Monitoring quota saturation and number of service instances

## Common Errors
### Installation and Upgrades
Certificate issues - ODB requires valid certificates 
Deploy fails
Network issues
Network broker is being deployed to is cannot reach the BOSH director
Network broker is being deployed to can’t reach CF
Register broker fails
Tile is being reinstalled and was not correctly uninstalled [existing issue for all tiles]
Smoke test failed
Network issues
CF cannot reach the service broker
CF cannot reach the service instances
service network cannot access the BOSH director
Resource sizing issues
The resource sizes selected for a given plan are less than the service requires to function.
Service specific issues

### BOSH problems
### Configuration
### Authentication
### Networking

Common issues include:

1. Network latency issues when connecting to the service instance to create or delete a binding. Solution is to try again or improve network performance
1. Network firewall rules are blocking connections from the service broker to the service instance. In OpsManager go into the tile and see the two networks that are configured in the networks tab. Ensure that these networks allow access to each other.

To validate you can `bosh ssh` onto the service broker by first downloading the broker manifest and targeting the deployment, and try to reach the service instance. If no BOSH task-id is in the error message (same error as above, but blank after task-id: ) then look in the broker log using the broker-request-id from the task.

There could be a problem accessing BOSH’s UAA or the BOSH director
Follow network troubleshooting and check BOSH director is online

### Quotas

Plan Quota issues
If the error is “Service broker error: The quota for this service plan has been exceeded. Please contact your Operator for help.” then

Checking your current plan quota - TODO - write operator facing docs for ODB metrics 
Increase the plan quota
Log into ops manager
Reconfigure the quota on the plan page
Deploy the tile
Finding who is using the plan quota - TODO CF workflow

Global Quota issues
If the error is “Service broker error: The quota for this service has been exceeded. Please contact your Operator for help.” then
Checking your current global quota - TODO OpsManager workflow
Increase the global quota 
Log into ops manager
Reconfigure the quota on the on-demand settings page
Deploy the tile
Finding who is using the quota - TODO CF workflow

### Failing jobs and unhealthy instances
An issue with the service deployment
Run `bosh vms --vitals service-instance_$guid`
For additional information run `bosh instances --ps --vitals`
If the VM is failing you will need to follow the service specific information, as any uninstructed remediatory actions (such as running bosh restart on a VM) may cause issues in the service instance.

### Missing logs and metrics


If no logs are being emitted by the on-demand broker, check that your syslog forwarding address is correct in Ops Manager.

To verify if metrics are being emitted to the firehose:
1. Install the [`cf nozzle` plugin](https://github.com/cloudfoundry/firehose-plugin)
2. Run `cf nozzle -f ValueMetric | grep --line-buffered “on-demand-broker/<service offering name>” `

If no metrics appear within 5 minutes:
Verify that the broker network has access to the loggregator system, on all required ports. [Contact Pivotal support](#filing-a-support-ticket) if you are unable to resolve the issue. 

### Orphaned Instances

you can remove the deployment from BOSH and CF directly
`cf service --guid` 
Record this guid
`bosh delete deployment service-instance_$guid`
If this fails:
`bosh delete deployment service-instance_$guid --force`
This will ignore any errors while deleting
WARNING - this may leave IaaS resources
Next purge the service instance in CF
`cf purge-service-instance myservice`

## Filing a support ticket

You can file a support ticket [here](https://support.pivotal.io/). Please be sure to provide the [error message](#parsing-a-cf-error-message) from `cf service <your-service>`, in addition to your [service broker logs](#accessing-broker-logs-and-vms) and your [service instance logs](#accessing-service-instance-logs-and-vms) to help expedite troubleshooting.

## Knowledge Base (Community)

Imagine a world where we autopopulate the first 10 hits from http://discuss.pivotal.io . For now, you'll just have to click on the link.