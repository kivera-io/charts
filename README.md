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

Check the deployment status:
```
kubectl get deployment kivera-proxy

Output:
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
kivera-proxy   1/1     1            1           60s
```

Check the service status:
```
kubectl get svc kivera-proxy

Output:
NAME           TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                         AGE
kivera-proxy   LoadBalancer   10.0.0.0        50.0.0.0      8080:32048/TCP,8090:30758/TCP   60s
```

Check the pod status:
```
kubectl get pod -l app=kivera

Output:
NAME                            READY   STATUS    RESTARTS   AGE
kivera-proxy-5c5b8dd947-ldd8q   1/1     Running   0          60s
```

If deployed with service type *LoadBlancer*.

```
export PROXY_ENDPOINT=$(kubectl get svc kivera-proxy -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
```

If deployed with service type *ClusterIP*.

```
export POD_NAME=$(kubectl get pod -l app=kivera -o jsonpath="{.items[0].metadata.name}")
kubectl port-forward $POD_NAME 8080:8080 &
kubectl port-forward $POD_NAME 8090:8090 &
export PROXY_ENDPOINT=localhost
```

Verify that the proxy is running by invoking the proxy version endpoint.

```
curl $PROXY_ENDPOINT:8090/version
```
