# Kivera Helm Charts

This repository contains Helms charts for the Kivera Proxy.

## Prerequisites

- Clone this repository to your local.
- Download a set of proxy credentials from the [Kivera Proxies page](https://app.kivera.io/proxies).
- [Helm](https://helm.sh/docs/intro/install/) installed.
- [Kubectl](https://kubernetes.io/docs/tasks/tools/) installed.
- Set the kubectl context to target your desired cluster.

## Deploy

Run the following commands from the root of of this repository.

### Basic Deployment

```
helm install kivera-proxy ./kivera --set-file credentials=path/to/credentials.json
```

### Deploy Behind a Load Balancer

The helm chart will deploy the service with type ClusterIP by default. In order to deploy behind a load balancer, set the following options.

```
helm install kivera-proxy ./kivera \
    --set service.type=LoadBalancer \
    --set-file credentials=path/to/credentials.json
```

### Deploy With Self Signed Certificate

1. Generate a private key and self signed certificate.

```
openssl ecparam -name prime256v1 -genkey -out ca.pem
openssl req -new -x509 -key ca.pem -out ca-cert.pem -subj "/C=AU/ST=NSW/L=Sydney/O=Kivera/OU=IT/CN=kivera.io" -days 3652
```

2. Deploy Kivera using the helm chart.

```
helm install kivera-proxy ./kivera \
    --set-file credentials=path/to/credentials.json \
    --set-file pubCert=ca-cert.pem \
    --set-file pvtKey=ca.pem
```

## Verify Deployment

Run the following commands to verify the deployment.

```
kubectl get deployment kivera-proxy
kubectl get svc kivera-proxy
kubectl get pod -l app=kivera
```

If deployed with service type *LoadBlancer*.

```
export ENDPOINT=$(kubectl get svc kivera-proxy -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
export PROXY_ENDPOINT=$ENDPOINT:8080
export MGMT_ENDPOINT=$ENDPOINT:8090
```

If deployed with service type *ClusterIP*.

```
export POD_NAME=$(kubectl get pod -l app=kivera -o jsonpath="{.items[0].metadata.name}")
kubectl port-forward $POD_NAME 8080:8080 &
kubectl port-forward $POD_NAME 8090:8090 &
export PROXY_ENDPOINT=localhost:8080
export MGMT_ENDPOINT=localhost:8090
```

Verify that the proxy is running by invoking the proxy version endpoint.

```
curl $MGMT_ENDPOINT/version
```
