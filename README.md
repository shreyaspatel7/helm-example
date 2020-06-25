# helm-example
## Introduction
This is an example of helm deployment of the simple flask api [app](https://github.com/shreyaspatel7/app-test) we created previously. As a part of CICD we will build and push the image from the previous repo. Then either via downstream job or through spinnaker we can perform the helm release throught out this helm chart. The helm chart includes manifests for deployment, and clusterIP service. We have also included livenesprob on the TCP port that is running the flask api as well as readinessprob for /ready endpoint of the application.


We have also added a test pod manifests to test the release via service DNS to make wget call to check if the /will api end point is responding or not.

## Implementation detail

This helm chart will deploy the flask api [app](https://github.com/shreyaspatel7/app-test) with any given configuration mentioned in the following section. It will as also create clusterIP k8s service mapping to appropriate pod.


### Configurations
As a part of nonroot user policy, we have configured the docker image to run as a specific user. The pod and deployment security context will be configured to throw the following yaml values. 
`We can further improve this chart by adding pod security policies and network security policies`.

```
podSecurityContext:
 runAsGroup: 1001
 runAsUser: 1001

....

securityContext:
 capabilities:
   drop:
   - ALL
 readOnlyRootFilesystem: true
 runAsNonRoot: true
 runAsUser: 1001
 allowPrivilegeEscalation: false
```

The pod port configuration is being done using the following yaml values in the values.yaml file:


```
netowrking:
 container:
   port: 5000
   name: app
   protocol: TCP
```
*If you don't define these configurations then the default is set to the following*:
```
ports:
- name: http
   containerPort: 80
   protocol: TCP
```

###
The monitoring configurations such as liveness and readiness probs are being done using the following:

```
monitorConfig:
 enabled: true
 livenessProbe:
   tcpSocket:
     port: 5000
   initialDelaySeconds: 5
   failureThreshold: 3
   timeoutSeconds: 5
   successThreshold: 1
   periodSeconds: 2

 readinessProbe:
   httpGet:
     path: /ready
     port: 5000
   timeoutSeconds: 5
   periodSeconds: 5
   initialDelaySeconds: 5
   failureThreshold: 3
   successThreshold: 1
 ```

You can disable the monitoring by simply setting `enabled: false`


## How do I perform releases ?

Simply run the following helm command after cloning this repo :)

```
helm install app-release . -n app --create-namespace

```

You can debug the templates via following command

```
helm install app-release . --dry-run --debug
```

You can test the release via following command

```
helm test app-release -n app
```

