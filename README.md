# NGINX manifest for AWeber

This repository is an assessment turn in for Ryan at AWeber

The properties of this assessment are as follows.\
Write a Kubernetes manifest to deploy nginx. The configuration must meet the following requirements:

1. Creates a Deployment object for nginx named “hello-world”
2. The Deployment should assign the label “app: hello-world” to the nginx pods, which selectors in all Service and Deployment objects should match on
3. Creates a Service object for nginx that’s exposed on port 80 within the cluster and port 30080 outside the cluster
4. The nginx containers should listen on port 8080
5. The nginx containers should run latest stable nginx alpine Docker image  
6. The nginx.conf should be defined as a ConfigMap that’s mounted as a volume into the container
7. When HTTP requests are made to nginx, it should respond with “Hello, world!”
