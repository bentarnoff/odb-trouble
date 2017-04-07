# ODB TROUBLESHOOTING GUIDE

## Common problems
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
#### Missing BOSH director UUID
If using the BOSH v1 CLI you need to re-add the director_uuid to the manifest
```
bosh status --uuid
```
Edit the manifest and add the `director_uuid: ` from the previous command at the top of the manifest. For more, see https://bosh.io/docs/manifest-v2.html#deployment 

#### Large BOSH queue

OpsManager currently deploys two BOSH workers. On-demand service brokers will add tasks to the BOSH queue. Until the workers reach the queued items the application developer will see `create in progess` in the CF CLI. 

The number of BOSH workers will become configurable in future version of OpsManager.

### Configuration

#### Service instances in failing state

You may have configured a VM / Disk type in tile plan page in OpsManager that is too low for the service instance to start. Please see tile specific guideance on resource requirements.

### Authentication

#### UAA Changes

If you have rotated any UAA users credentials then you may see authenticiatin issues in the service broker logs. 

To resolve this you should redeloy the service tile in OpsManager, which will provide the broker with the latest configuration.

!!! note
	You must ensure that any changes to UAA credentials are reflected in the OpsManager `credentials` tab of the Elastic Runtime Tile.

### Networking

Common issues include:

1. Network latency issues when connecting to the service instance to create or delete a binding. Solution is to try again or improve network performance
1. Network firewall rules are blocking connections from the service broker to the service instance. In OpsManager go into the tile and see the two networks that are configured in the networks tab. Ensure that these networks allow access to each other.
1. Network firewall rules are blocking connections from the service network to the BOSH director network. Service instances must have access to the director so that the BOSH agents can report in.
1. Cloud Foundry application security groups may not be configured to allow access to the service network. This is required to allow applications on the runtime to connect to services.
1. There could also be a problem accessing BOSH’s UAA or the BOSH director. Follow network troubleshooting and check BOSH director is online

#### Validating the service brokers connectivty to service instances

To validate you can `bosh ssh` onto the service broker by first downloading the broker manifest and targeting the deployment, and try to reach the service instance. If no BOSH task-id is in the error message, then look in the broker log using the broker-request-id from the task.

#### Validating the application on the runtime has access to the service instance.

You can use `cf ssh` to gain access to the application container and then attempt to connect to the service instance using the binding detailed in the VCAP_SERVICES environment variable.

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
Please also provide your [service broker logs](techniques/#accessing-broker-logs-and-vms) and your [service instance logs](techniques/#accessing-service-instance-logs-and-vms) to help expedite troubleshooting.

## Knowledge Base (Community)

Imagine a world where we autopopulate the first 10 hits from http://discuss.pivotal.io . For now, you'll just have to click on the link.