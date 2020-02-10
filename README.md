# istio-tutorial

# Setup namespace

kubectl create namespace tutorial
  
kubectl config set-context $(kubectl config current-context) --namespace=tutorial


# Clone Repo


#git clone https://github.com/redhat-developer-demos/istio-tutorial

cd istio-tutorial

# Deploy
kubectl apply -f <(istioctl kube-inject -f customer/kubernetes/Deployment.yml) -n tutorial

kubectl create -f customer/kubernetes/Service.yml -n tutorial

kubectl create -f customer/kubernetes/Gateway.yml -n tutorial

kubectl apply -f <(istioctl kube-inject -f preference/kubernetes/Deployment.yml)  -n tutorial

kubectl create -f preference/kubernetes/Service.yml -n tutorial

kubectl apply -f <(istioctl kube-inject -f recommendation/kubernetes/Deployment.yml) -n tutorial

kubectl create -f recommendation/kubernetes/Service.yml -n tutorial

kubectl get pods -w -n tutorial

# Check ingress and get the GATEWAY_URL

kubectl get svc istio-ingressgateway -n istio-system

export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].nodePort}')

export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].nodePort}')

export INGRESS_HOST=$(minikube ip)

export GATEWAY_URL=$INGRESS_HOST:$INGRESS_PORT

# Check its running

./scripts/run.sh $GATEWAY_URL/customer

# View the dashboards

istioctl dashboard kiali

istioctl dashboard jaeger

kubectl -n istio-system port-forward $(kubectl -n istio-system get pod -l app=grafana -o jsonpath='{.items[0].metadata.name}') 3000:3000 &

http://localhost:3000/dashboard/db/istio-mesh-dashboard 

# Deploy v2 - to see load balanced traffic across v1 & v2

kubectl apply -f <(istioctl kube-inject -f recommendation/kubernetes/Deployment-v2.yml) -n tutorial

kubectl scale --replicas=2 deployment/recommendation-v2 -n tutorial

kubectl get pods

# Move all traffic to v2

kubectl create -f istiofiles/destination-rule-recommendation-v1-v2.yml -n tutorial

kubectl create -f istiofiles/virtual-service-recommendation-v2.yml -n tutorial

# Move traffic back to v1

kubectl replace -f istiofiles/virtual-service-recommendation-v1.yml -n tutorial

# Back to v1 & v2

kubectl delete -f istiofiles/virtual-service-recommendation-v1.yml -n tutorial
