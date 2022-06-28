## Production Service Mesh Control Plane (SMCP) Best Practices and Recommendations

For each SMCP capability, we state our assumptions first and then our recommendation based on RH best practices.

## Distributed Tracing - Jaeger  

- Assumption: flexible and scalable production-ready Service Mesh monitoring capability platform

### Recommendation:
- Use "Production" deployment strategy type in Jaeger CRD:
    Intended for production environments, where long term storage of trace data is important.  
    The Agent can be injected as a sidecar on the instrumented application. 
    The Query and Collector services are configured with a supported storage type, which is currently Elasticsearch. 
    Multiple instances of each of these components can be provisioned as required for performance and resilience purposes.
- Elasticsearch propeties:
    - storageClass: use same as provisioned by the Elasticsearch operator
    - redundancyPolicy: defines how Elasticsearch shards are replicated across data nodes in the cluster
        e.g. SingleRedundancy(one replica shard)
    - nodeCount: number of Elasticsearch nodes. For high availability use at least 3 nodes. 
        Do not use 2 nodes as “split brain” problem can happen.
- Collector: the component responsible for receiving the spans that were captured by the tracer and 
    writing them to persistent Elasticsearch storage when using the production strategy
    The Collectors are stateless and thus many instances of Jaeger Collector can be run in parallel. 
    Collectors require almost no configuration, except for the location of the Elasticsearch cluster.
- Configuration Steps:
    Create the Jaeger CRD first, then configure SMCP to use that resource. 
    The Jaeger resource must be in the same namespace as the SMCP resource. 
    This approach gives users complete flexibility for configuring Jaeger.


The following Jaeger CRD yaml snippet illustrates the above points.

![](jaeger.png)


## Control Plane Security - mTLS

- Assumption: secure communication within Control Plane components (e.g. between Istiod pod and Jaeger pod)

## Data Plane Security - mTLS

- Assumption: workloads within the mesh Do Not call out to services outside the mesh; secure communication within the mesh between namepaces/projects (e.g. between bookinfo and sleep)

### Recommendation:
- Set "mtls: true" under Security: controlPlane
    By default, Red Hat OpenShift Service Mesh generates a self-signed root certificate and key and uses them to sign the control plane component workload certificates

- Set "mtls: true" under Security: dataPlane
    By default, Red Hat OpenShift Service Mesh generates a self-signed root certificate and key and uses them to sign the data plane component workload certificates
    
    
## OSSM Fine Grained Security Protocol Requirements

- Assumption: need more detailed feedback from Telus regarding environment specific requirements for encrypted traffic in the service mesh, you can control the cryptographic functions, etc.

## Automatic Route Creation

### This settings instructs the Control Plane to auto create OpenShift Routes based on Service Mesh Gateway CRD's

- spec.gateways.openshiftRoute:enabled: true
- Naming convention: "http://<namespace>-<gateway name>-<hash>-<control plane namespace>.<wildcard domain host name>"
- e.g. http://bookinfo-bookinfo-gateway-525eca1d5089dbdc-istio-system.apps.cluster-47mmf.47mmf.sandbox1759.opentlc.com
- The Control Plane will manage the creation/deletion of the route when Gateway CRD changed/deleted/created
- If the Gateway CRD defines a secure HTTPS gateway resource, the control plane will create the secure TLS edge-termited route accordingly.
- Note: manually created Routes will not be affected by this setting

## Sampling Setting for Enovy Proxy

    The sampling rate determines how often the Envoy proxy generates a trace. You can use the sampling rate option to control what percentage of requests get reported to your tracing system. 
    You can configure this setting based upon your traffic in the mesh and the amount of tracing data you want to collect. You configure sampling as a scaled integer representing 0.01% increments. 
    For example, setting the value to 10 means to sample 0.1% of traces, setting the value to 500 samples 5% of traces, and a setting of 10000 samples 100% of traces.

![](sampling.png)
    
## Multitenant deployment model

  - A namespace can only be included in a single mesh, and multiple meshes can be installed in a single OpenShift cluster.
  - Control plane is used to configure communication between services in the mesh. 
  - OSSM supports “soft multitenancy”, where there is one control plane and one mesh per tenant, and there can be multiple independent control planes within the cluster. 
  - Multitenant deployments specify the projects that can access the Service Mesh and isolate the Service Mesh from other control plane instances.
  - The cluster administrator gets control and visibility across all the Istio control planes, while the tenant administrator only gets control over their specific Service Mesh, Kiali, and Jaeger instances.
  - You can grant a team permission to deploy its workloads only to a given namespace or set of namespaces. If granted the "mesh-user" role by the service mesh administrator, users can create a ServiceMeshMember resource to add namespaces to the ServiceMeshMemberRoll.

    
## Resources: Requests and Limits

-  Requests are more important than limits. Limits tell the linux kernel when to consider your process a candidate for freeing up memory. Requests help the kubernetes scheduler figure out where it can run your pod. Not setting them, or setting them artificially low, can have bad effects.
-  We do not recommend manually setting CPU and Memory resource requests and limits.  Let the Operator manage it.
-  Unless you have high confidence of your production pod memory/cpu consumption, do not manually set the resources.
    
## References

Service Mesh Jaeger:
https://docs.openshift.com/container-platform/4.9/service_mesh/v2x/ossm-reference-jaeger.html
