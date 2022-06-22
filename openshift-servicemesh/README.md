## Production Service Mesh Control Plane (SMCP) Best Practices and Recommendations

For each non-default SMCP configuration settings, we state our assumptions first and then our recommendation based on RH best practices.

## Jaeger is used for monitoring and troubleshooting microservices-based distributed tracing.  

# Assumption: flexible and scalable production-ready Service Mesh monitoring capability platform

# Recommendation:
- Use "Production" deployment strategy type in Jaeger CRD
- Configuration Steps:
    Create the Jaeger CRD first, then configure SMCP to use that resource. 
    The Jaeger resource must be in the same namespace as the SMCP resource. 
    This approach gives users complete flexibility for configuring Jaeger.
    
    
    
    
## References

Service Mesh Jaeger:
https://docs.openshift.com/container-platform/4.9/service_mesh/v2x/ossm-reference-jaeger.html
