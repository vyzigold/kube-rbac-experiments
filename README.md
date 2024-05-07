This repository includes CRs I used to get authentication for metrics in openstack.

What works:
- ceilometer-internal endpoint can be secured with either token-based auth or mTLS with the kube-rbac-proxy.
- prometheus endpoint can be secured with either token-based auth or mTLS with the kube-rbac-proxy.
- Node Exporter on the compute nodes can't use kube-rbac-proxy, because kube-rbac-proxy requires access to the kubernetes api, which we cannot guarantee. But Node Exporter supports mTLS natively.

What doesn't work:
- Securing of alertmanager. We'd require additional support in COO, so that prometheus can authenticate to alertmanager when accessing it. (I'm trying to work on this)
- How do users access prometheus. Users would need to either have a certificate for mTLS or a token. This isn't easy to do in a browser without some kind of login page, which we don't have.
- I don't see an option for the console UI dashboard plugin to specify how it should authenticate with our secured prometheus. (Still need to test this)
- Authentication of the observabilityclient when it accesses prometheus, but this should be easy enough to add. Getting either mTLS or token based auth should be fairly simple.
- For mTLS to work, there must exist the user (User, not a ServiceAccount) of the same name is is specified in the commonName of the client certificate being used. This user needs to have the RBAC permissions to access the requested resources. This doesn't seem correct to me. It should work with ServiceAccounts. There is this: https://rhobs-handbook.netlify.app/products/openshiftmonitoring/collecting\_metrics.md/#kube-rbac-proxy-sidecar which includes a config, which looks like it fixes this issue.

How to make this work:
- Deploy openstack control and dataplane as you normally would.
- Install COO
- Enable Autoscaling, Heat and MetricStorage.
- Delete the telemetry-operator to stop it from overriding your changes
- Each folder has multiple files. Use `oc apply -f <filename>` for all files except for `patch.yaml`
- Execute `oc apply --server-side -f patch.yaml`

NOTE: There might be some additional configuration needed. I haven't done any "third pass" over the instructions yet. I might have forgotten something, so in case something doesn't work have a look at the statuses of the created resources, logs or ping me, I might remember what's going on.
