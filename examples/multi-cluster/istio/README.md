## Multi-region YugabyteDB on Kubernetes with Istio
This directory contains values files used for each of the Kubernetes
cluster. You can find more details in [this blog
post](https://blog.yugabyte.com/multi-region-yugabytedb-deployments-on-kubernetes-with-istio/).

Let's take a look at each configuration section.

### Istio compatibility
```yaml
istioCompatibility:
  enabled: true

multicluster:
  createServicePerPod: true
  createCommonTserverService: true
```

The `createServicePerPod` option creates a separate clusterIP service
for each YB-Master and YB-TServer pod. This is required due to an
issue with Istio, where it is not possible to reach directly to a
single pod when we have headless service + StatefulSet.

`createCommonTserverService` creates a service which has same name
across all the Kubernetes clusters regardless of the Helm release
name. This is helpful to have same service FQDN across all the
clusters.


### Master addresses
```yaml
isMultiAz: true

masterAddresses: "{east-yugabyte-yb-master-0.ybdb.svc.cluster.local,east-yugabyte-yb-master-0.east-yugabyte-yb-masters.ybdb.svc.cluster.local},{west-yugabyte-yb-master-0.ybdb.svc.cluster.local},{central-yugabyte-yb-master-0.ybdb.svc.cluster.local}"
```

With `isMultiAz` option, all the YB-Master and YB-TServer pods
communicate with the master addresses specified in
`masterAddresses`. The format here is:

```
{<broadcasted_address (pod service)>,<internal_address (headless service FQDN)>}, {…}, {…}
```

### Other options
```yaml
replicas:
  totalMasters: 3
  master: 1
  tserver: 2

gflags:
  master:
    placement_region: us-east1
    placement_cloud: gcp
  tserver:
    placement_region: us-east1
    placement_cloud: gcp

oldNamingStyle: false
```

With the `replicas` options, we create 1 YB-Master and 2 YB-TServer
per Kubernetes server.

With the `gflags` option, we set the correct region and cloud provider
name, it can be anything like `aws`, `azure` etc.

The `oldNamingStyle: false` option makes sure that the release name is
used as part of Kubernetes object names.
