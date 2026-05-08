```markdown
# NGINX Gateway Fabric on Rancher Desktop

This guide explains how to set up NGINX Gateway Fabric using the Kubernetes Gateway API within Rancher Desktop. Unlike kind, Rancher Desktop provides a built-in Load Balancer that maps directly to your localhost.

## Prerequisites

* **Rancher Desktop**: Running with Kubernetes enabled.
* **Disable Traefik**:
  * Go to **Settings > Kubernetes**.
  * Uncheck **Enable Traefik**.
  * *Why?* Traefik uses port 80 by default, which will prevent NGINX Gateway Fabric from binding to that port.
* **CLI Tools**: `kubectl` and `helm` installed.

## Step 1: Install Gateway API CRDs

The Gateway API resources (`GatewayClass`, `Gateway`, `HTTPRoute`) are not included in standard Kubernetes clusters by default. You must install these Custom Resource Definitions (CRDs) first.

```bash
kubectl kustomize "[https://github.com/nginx/nginx-gateway-fabric/config/crd/gateway-api/standard?ref=v1.6.0](https://github.com/nginx/nginx-gateway-fabric/config/crd/gateway-api/standard?ref=v1.6.0)" | kubectl apply -f -


Step 2: Install NGINX Gateway Fabric via Helm
Install the controller into the nginx-gateway namespace. We set the service type to LoadBalancer.

Bash
# Add and update helm repo if not already done
helm install ngf oci://ghcr.io/nginx/charts/nginx-gateway-fabric \
  --create-namespace \
  -n nginx-gateway \
  --set nginx.service.type=LoadBalancer


Step 3: Deploy the Example Application
Create a namespace and a simple "coffee" service.

Bash
kubectl apply -f - <<EOF
apiVersion: v1
kind: Namespace
metadata:
  name: cafe
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: coffee
  namespace: cafe
spec:
  replicas: 1
  selector:
    matchLabels:
      app: coffee
  template:
    metadata:
      labels:
        app: coffee
    spec:
      containers:
      - name: coffee
        image: nginxdemos/nginx-hello:plain-text
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: coffee
  namespace: cafe
spec:
  ports:
  - port: 80
    targetPort: 8080
  selector:
    app: coffee
EOF


Step 4: Create the Gateway and HTTPRoute
This configures NGINX to listen for traffic and route /coffee requests to our service.

Bash
kubectl apply -f - <<EOF
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: cafe-gateway
  namespace: cafe
spec:
  gatewayClassName: nginx
  listeners:
  - name: http
    port: 80
    protocol: HTTP
    allowedRoutes:
      namespaces:
        from: Same
---
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: coffee-route
  namespace: cafe
spec:
  parentRefs:
  - name: cafe-gateway
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /coffee
    backendRefs:
    - name: coffee
      port: 80
EOF


Step 5: Verify and Test
1. Check for the External IP
Bash
kubectl get svc -n nginx-gateway
In Rancher Desktop, the EXTERNAL-IP should show localhost or 127.0.0.1.

2. Verify Gateway Status
Bash
kubectl get gateway cafe-gateway -n cafe
Ensure the status shows Programmed: True.

3. Test the Route
Bash
curl http://localhost/coffee
Troubleshooting
  
External IP is Blank: Ensure you unchecked "Enable Traefik" in Rancher Desktop settings and restarted the cluster if necessary.

Port Conflict: Check if another application (like IIS, Apache, or Docker Desktop) is using port 80 on your host machine.

Manual Tunnel: If the LoadBalancer still won't assign an IP, you can use:

Bash
kubectl port-forward svc/ngf-nginx-gateway-fabric -n nginx-gateway 8080:80
Then access via http://localhost:8080/coffee.