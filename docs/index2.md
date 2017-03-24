# ON-DEMAND SERVICES: TROUBLESHOOTING GUIDE 

## Creating a service instance

If you are unable to create a new service instance via Apps Manager or the CF CLI, perform the following steps to diagnose and resolve the issue:

The first step is to retrieve the last error message from the cloud controller by running `$ cf service {YOURSERVICEINSTANCE}`

```java
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
Message: {ERROR MESSAGE OUTPUT TO COPY}
Started: 2017-03-13T10:16:55Z
Updated: 2017-03-13T10:17:58Z

```

### BOSH issues

```
Instance provisioning failed: There was a problem completing your request. 
Please contact your operations team providing the following information: 
service: redis-acceptance, 
service-instance-guid: ae9e232c-0bd5-4684-af27-1b08b0c70089, 
broker-request-id: 63da3a35-24aa-4183-aec6-db8294506bac, 
task-id: 442, 
operation: create
```

Errors like the above require that you trace the error back to the originating component. In this case, we'll use the provided `task-id` to retrieve the task summary from BOSH so we can dig deeper. If you are unfamiliar with how to connect to BOSH, please read the (Advanced Troubleshooting with the BOSH CLI,https://docs.pivotal.io/pivotalcf/1-9/customizing/trouble-advanced.html#cli) article. 

Once you've connected to the BOSH director using the BOSH CLI, run the following command:

	$ bosh task {YOUR_TASK_ID}
	$ [INSERT BOSH TASK OUTPUT HERE]


### Manifest issues

Sometimes, errors can result from misconfiguration of the Service Broker in Ops Manager. You can retrieve the manifest being used by BOSH by performing the following steps:

First, retrieve the guid of your service instance with the following command:

	$ cf service {YOUR_SERVICE} --guid

Second, pre-pend the text "service_instance-" to the guid you just obtained to create the ID that BOSH uses for your service instance (e.g. if your guid is 1Hy6H), this becomes service_instance-1Hy6H).

Third, download your bosh manifest for the service 

	$ bosh download manifest {BOSH_ID_OF_SI} myservice.yml

Inspect the manifest for any discrepancies from desired values. You may want to change these values in Ops Manager and redeploy the tile.

<script src="//cdnjs.cloudflare.com/ajax/libs/highlight.js/9.10.0/highlight.min.js"></script>

