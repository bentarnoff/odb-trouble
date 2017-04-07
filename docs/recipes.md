# ODB TROUBLESHOOTING GUIDE

## Recipes
### My install failed
1. [Certificate issues]() - (link TK) ODB requires valid certificates. Ensure that your certificates are valid and generate new ones if necessary via Ops Manager 
2. Deploy fails - Deploys can fail for a variety of reasons. View the logs via Ops Manager to determine why the deploy is failing.
3. [Networking problems](common_problems/#networking) - These can occur when:
CF cannot reach the service broker
CF cannot reach the service instances
The service network cannot access the BOSH director	
4. [Register broker errand](techniques/#register-broker) fails
6. The Smoke test errand failed
7. Resource sizing issues - These occur when the resource sizes selected for a given plan are less than the service requires to function.
8. Service specific issues

### Reinstalling a tile in the same environment where it was previously uninstalled
1. Ensure that the previous tile was correctly uninstalled
1. `cf login` as an admin user
1. `cf m` does not list the service
1. `bosh login` as an admin user
1. `bosh deployments` does not show a deployment for the service. For example no `p-concourse-$guid` deployment exists
1. Before reinstalling a tile all previous service instances and service brokers must be deleted from the BOSH director
1. If the service broker still exists then run the [delete-all-service-instances errand]() and then [degregister-broker errand]()
1. Next delete the service broker BOSH deployment
1. Finally install the tile again

### My upgrade failed
### My app devs report that they cannot create service instances
### My app devs report that they cannot bind to service instances
### My app devs report that they cannot delete service instances
### My app devs report that they cannot unbind from service instances
### App dev reports that they cannot connect to a created service
### App dev reports a problem with creating or using a service key

## Filing a support ticket

You can file a support ticket [here](https://support.pivotal.io/). Please be sure to provide the [error message](#parsing-a-cf-error-message) from 
```
$ cf service <your-service>
``` 
Please also provide your [service broker logs](techniques/#accessing-broker-logs-and-vms), your [service instance logs](techniques/#accessing-service-instance-logs-and-vms) and [BOSH task output](techniques/#parsing-a-cf-error-message), if a `task-id` is provided as part of the `cf service <your-service>` output, to help expedite troubleshooting.

## Knowledge Base (Community)

Imagine a world where we autopopulate the first 10 hits from http://discuss.pivotal.io . For now, you'll just have to click on the link.