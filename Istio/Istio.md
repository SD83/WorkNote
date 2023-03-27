minikube start --memory=16384 --cpus=4 --kubernetes-version=v1.26.1 --embed-certs
istioctl install --set profile=demo -y
kubectl label namespace default istio-injection=enabled