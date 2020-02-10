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

