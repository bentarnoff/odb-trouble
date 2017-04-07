# Errors
## My install failed
1. [Certificate issues]() - (link TK) ODB requires valid certificates. Ensure that your certificates are valid and generate new ones if necessary via Ops Manager 
2. Deploy fails - Deploys can fail for a variety of reasons. View the logs via Ops Manager to determine why the deploy is failing.
3. [Networking problems](common_problems/#networking) - These can occur when:
-CF cannot reach the service broker
-CF cannot reach the service instances
-The service network cannot access the BOSH director	
4. [Register broker errand](techniques/#register-broker) fails
6. The [Smoke test errand](techniques/#smoke-test) failed
7. Resource sizing issues - These occur when the resource sizes selected for a given plan are less than the service requires to function. Check your resource configuration in Ops Manager and ensure that the configuration matches that recommended by the service.
8. Service specific issues

## Reinstalling a tile in the same environment where it was previously uninstalled
1. Ensure that the previous tile was correctly uninstalled
1. `cf login` as an admin user
1. `cf m` does not list the service
1. `bosh login` as an admin user
1. `bosh deployments` does not show a deployment for the service. For example no `p-concourse-$guid` deployment exists
1. Before reinstalling a tile all previous service instances and service brokers must be deleted from the BOSH director
1. If the service broker still exists then run the [delete-all-service-instances errand]() and then [degregister-broker errand]()
1. Next delete the service broker BOSH deployment
1. Finally install the tile again

## My upgrade failed
## My app devs report that they cannot create or delete service instances
 
```
Instance provisioning failed: There was a problem completing your request. Please contact your operations team providing the following information: service: redis-acceptance, service-instance-guid: ae9e232c-0bd5-4684-af27-1b08b0c70089, broker-request-id: 63da3a35-24aa-4183-aec6-db8294506bac, task-id: 442, operation: create
``` 
Check the BOSH task information:
Follow the advanced BOSH troubleshooting guide to log in to BOSH and target the service instance using the instructions on parsing an error message.

`bosh task $task-id` from the error above

1. Configuration errors: If the BOSH error shows a problem with the deployment manifest

Get the service-instance-manifest:
`bosh download manifest service-instance_$guid myservice.yml`

[Access the broker logs]()  and use the `broker-request-id` from the error message above to search the log for more information.

2. [Authentication errors]()

3. [Network errors]()

4. [Quota errors]() 

## My app devs report that they cannot bind to or unbind from service instances
### Instance does not exist
If the error message says “Server error, status code: 502, error code: 10001, message: Service broker error: instance does not exist”

1. Check that the service instance exists in BOSH and CF
```
cf service myservice --guid
```
2. Record the guid response and run the bosh vms command
```bosh vms service-instance_$guid```
If the bosh deployment is not found then it has been deleted from BOSH. Contact Pivotal support for further assistance on recovery steps.

### Other errors
```
Server error, status code: 502, error code: 10001, message: Service broker error: There was a problem completing your request. Please contact your operations team providing the following information: service: example-service, service-instance-guid: 8d69de6c-88c6-4283-b8bc-1c46103714e2, broker-request-id: 15f4f87e-200a-4b1a-b76c-1c4b6597c2e1, operation: bind
```

To find out the exact issue with creating the binding [access the service broker logs]()

Use the broker-request-id from the error message above to search the log for more information. 

Contact Pivotal support for further assistance if you are unable to resolve the problem.

2. [Authentication errors]()

3. [Network errors]()


## My App devs report that they cannot connect to a created service

Ask the user to send application logs that show the connection error. If the error is originating from the service then follow service specific instructions. If the issue appears to be network related then:

Check that application security groups are configured correctly
https://docs.cloudfoundry.org/adminguide/app-sec-groups.html
Access should be configured for the service network the tile is deployed to
Ensure that the network the PCF Elastic Runtime tile is deployed to has network access to the service network.
In OpsManager go into the service tile and see the service network that is configured in the networks tab.
In OpsManager go into the ERT tile and see the network it is assigned to.
Make sure that these networks can access each other.

## App dev reports a problem with creating or using a service key
### Create errors
### Use errors

## Filing a support ticket

You can file a support ticket [here](https://support.pivotal.io/). Please be sure to provide the [error message](#parsing-a-cf-error-message) from 
```
$ cf service <your-service>
``` 
Please also provide your [service broker logs](techniques/#accessing-broker-logs-and-vms), your [service instance logs](techniques/#accessing-service-instance-logs-and-vms) and [BOSH task output](techniques/#parsing-a-cf-error-message), if a `task-id` is provided as part of the `cf service <your-service>` output, to help expedite troubleshooting.

## Knowledge Base (Community)

Imagine a world where we autopopulate the first 10 hits from http://discuss.pivotal.io . For now, you'll just have to click on the link.