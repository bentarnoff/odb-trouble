# ODB TROUBLESHOOTING GUIDE

## Techniques
### Using BOSH and CF to access logs and SSH

1. Follow the steps described here to [target your BOSH CLI](https://docs.pivotal.io/pivotalcf/1-9/customizing/trouble-advanced.html#prepare) and [log-in to your CF CLI](https://docs.cloudfoundry.org/cf-cli/getting-started.html)

#### Accessing broker logs and VM(s)

1. Identify the ODB deployment using 
```
$ bosh deployments
```
1. Download the ODB manifest using 
```
$ bosh download manifest <odb-deployment-name> odb.yml
```
1. Select the ODB deployment using 
```
$ bosh deployment odb.yml
```
1. View VMs in the deployment using 
```
$ bosh instances
```
1. Download broker logs by using the 
```
$ bosh logs <instance-id>
```
1. To ssh onto the VM use 
```
$ bosh ssh <instance-id>
```

#### Accessing service instance logs and VM(s)

1. To target an individual service instance deployment, retrieve the 
the guid of your service instance with the CF CLI command 
```
$ cf service {YOUR_SERVICE} --guid
```

1. Pre-pend the text `service_instance-` to the guid you just obtained to create the ID that BOSH uses for your service instance (e.g. if your guid is `1Hy6H`, this becomes `service_instance-1Hy6H`).

1. Download your bosh manifest for the service using 
```
$ bosh download manifest service_instance-guid myservice.yml
```
1. Select the deployment using 
```
$ bosh deployment myservice.yml
```
1. View VMs in the deployment using 
```
$ bosh instances
```
1. Download instance logs by using the 
```
$ bosh logs <instance-id>
```
1. To ssh onto the VM use 
```
$ bosh ssh <instance-id>
```

### Parsing a CF error message

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

### Smoke Tests, Checks, and Errands
### Viewing resource saturation and scaling resources

Once a deployment has been selected, you can use the `bosh vms --vitals` or the `bosh instances --vitals` commands to view current resource utilization. You can also view process level information by using `bosh instances --ps`.

### Identifying the owner of a service instance
If you have spotted a failing deployment, you can identify which org/space owns this service instance, as well as retrieve a list of apps bound to it by following the steps below: 

1. `bosh vms $deployment` shows a vm in failing state
1. Take the deployment name and strip the `service-instance_` leaving you with a `guid`
1. Login to CF using a CF Admin user
1. Run 
```
$ cf curl /v2/service_instances/:guid | grep "space_url"
"space_url": "/v2/spaces/$space_guid"
```
2. Take the space url and run 
```
$ cf curl /v2/spaces/$space_guid | grep -E "name|organization_url"
"name": "myspace",
"organization_url": "/v2/organizations/$org_guid"
```

3. Take the organization_url and run 
```
cf curl /v2/organizations/$org_guid | grep “name”
"name": "myorg"
```

4. Combine the output of the names to give you the org and space of the service instance. 
From here run:
```
cf target -o $org -s $space
``` 
5. `cf services` shows you all the services and bound apps
6. To find who is the space manger run:
```
cf curl /v2/spaces/$space_guid/managers
```
7. Use this information to contact the manager if needed 

### Monitoring quota saturation and number of service instances
Quota saturation and total number of service instances are available via ODB metrics emitted to loggregator. The metric names are shown below:

| **Metric Name**                                                           | **Description**                                           |
|---------------------------------------------------------------------------|-----------------------------------------------------------|
| `on-demand-broker/{service-name-marketplace}/quota_remaining`             | global quota remaining for all instances across all plans |
| `on-demand-broker/{service-name-marketplace}/{plan_name}/quota_remaining` | quota remaining for a particular plan                     |
| `on-demand-broker/{service-name-marketplace}/total_instances`             | total instances created across all plans                  |
| `on-demand-broker/{service-name-marketplace}/{plan_name}/total_instances` | total instances created for a given plan                  |

!!! note
	Quota metrics are not emitted if no quota has been set.

## Common Errors
### Installation and Upgrades
1. Certificate issues - ODB requires valid certificates. Ensure that your certificates are valid and generate new ones if necessary via Ops Manager 
2. Deploy fails - Deploys can fail for a variety of reasons. View the logs via Ops Manager to determine why the deploy is failing.
3. Networking problems - These can occur when:
CF cannot reach the service broker
CF cannot reach the service instances
The service network cannot access the BOSH director	
4. Register broker errand fails
5. The Tile is being reinstalled but was not correctly uninstalled 
6. The Smoke test errand failed
7. Resource sizing issues - These occur when the resource sizes selected for a given plan are less than the service requires to function.
8. Service specific issues

### BOSH problems
### Configuration
### Authentication
### Networking

Common issues include:

1. Network latency issues when connecting to the service instance to create or delete a binding. Solution is to try again or improve network performance
1. Network firewall rules are blocking connections from the service broker to the service instance. In OpsManager go into the tile and see the two networks that are configured in the networks tab. Ensure that these networks allow access to each other.
1. There could also be a problem accessing BOSH’s UAA or the BOSH director. Follow network troubleshooting and check BOSH director is online

To validate you can `bosh ssh` onto the service broker by first downloading the broker manifest and targeting the deployment, and try to reach the service instance. If no BOSH task-id is in the error message, then look in the broker log using the broker-request-id from the task.


### Quotas

####Plan Quota issues
If you see the following error:

```
Message: Service broker error: The quota for this service plan has been exceeded. 
Please contact your Operator for help.
```

1. Check your current plan quota - TODO - write operator facing docs for ODB metrics 
1. Increase the plan quota
1. Log into ops manager
1. Reconfigure the quota on the plan page
1. Deploy the tile
1. Find who is using the plan quota and take the appropriate action - TODO CF workflow

####Global Quota issues
If you see this error:

```
Message: Service broker error: The quota for this service has been exceeded. 
Please contact your Operator for help.
```

1. Check your current global quota - TODO OpsManager workflow
1. Increase the global quota 
1. Log into ops manager
1. Reconfigure the quota on the on-demand settings page
1. Deploy the tile
1. Find out who is using the quota and take the appropriate action - TODO CF workflow

### Failing jobs and unhealthy instances
To determine whether there is an issue with the service deployment

```
$ bosh vms --vitals service-instance_$guid
```

For additional information 
```
$ bosh instances --ps --vitals
```

If the VM is failing you will need to follow the service specific information, as any unadvised corrective actions (such as running bosh restart on a VM) may cause issues in the service instance.

### Missing logs and metrics


If no logs are being emitted by the on-demand broker, check that your syslog forwarding address is correct in Ops Manager.

To verify if metrics are being emitted to the firehose:

1. Install the [`cf nozzle` plugin](https://github.com/cloudfoundry/firehose-plugin)
1. Run: 
```
$ cf nozzle -f ValueMetric | grep --line-buffered “on-demand-broker/<service offering name>” 
```

If no metrics appear within 5 minutes:
Verify that the broker network has access to the loggregator system, on all required ports. [Contact Pivotal support](#filing-a-support-ticket) if you are unable to resolve the issue. 

### Orphaned Instances

If you have orphaned instances (instances that are recognized by CF but not by BOSH or vice versa), you can remove the deployment from BOSH and CF directly:

1. To find orphaned instances, run the `orphan-deployments` bosh errand
```
$ bosh run errand orphan-deployments
```
1. Run 
```
$ cf service --guid
``` 
1. Record this guid
1. Run
```
$ bosh delete deployment service-instance_$guid
```
1. If this fails, you can ignore any errors while deleting by running 
```
$ bosh delete deployment service-instance_$guid --force
``` 

	!!! warning
		This may leave IaaS resources in an unusable state

1. Next purge the service instance in CF
```
$ cf purge-service-instance myservice
```

## Filing a support ticket

You can file a support ticket [here](https://support.pivotal.io/). Please be sure to provide the [error message](#parsing-a-cf-error-message) from 
```
$ cf service <your-service>
``` 
Please also provide your [service broker logs](#accessing-broker-logs-and-vms) and your [service instance logs](#accessing-service-instance-logs-and-vms) to help expedite troubleshooting.

## Knowledge Base (Community)

Imagine a world where we autopopulate the first 10 hits from http://discuss.pivotal.io . For now, you'll just have to click on the link.