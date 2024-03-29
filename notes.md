#########03/06/2023###########

https://kubernetes.github.io/ingress-nginx/user-guide/exposing-tcp-udp-services/

https://www.nginx.com/blog/load-balancing-tcp-and-udp-traffic-in-kubernetes-with-nginx/

#########03/06/2023###########

ToDo

- [ ] Research the audit logs in kafka

## Set up helm
helm repo add bitnami https://charts.bitnami.com/bitnami

## Deploy Zoo Keeper
helm install zookeeper bitnami/zookeeper \
  --set replicaCount=3 \
  --set auth.enabled=false \
  --set allowAnonymousLogin=true

Output

ZooKeeper can be accessed via port 2181 on the following DNS name from within your cluster:

    zookeeper.default.svc.cluster.local

To connect to your ZooKeeper server run the following commands:

    export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=zookeeper,app.kubernetes.io/instance=zookeeper,app.kubernetes.io/component=zookeeper" -o jsonpath="{.items[0].metadata.name}")
    kubectl exec -it $POD_NAME -- zkCli.sh

To connect to your ZooKeeper server from outside the cluster execute the following commands:

    kubectl port-forward --namespace default svc/zookeeper 2181:2181 &
    zkCli.sh 127.0.0.1:2181


## Deploy Kafka brokers
helm install kafka bitnami/kafka \
  --set zookeeper.enabled=false \
  --set replicaCount=3 \
  --set externalZookeeper.servers=zookeeper.default.svc.cluster.local

  Output

  Kafka can be accessed by consumers via port 9092 on the following DNS name from within your cluster:

    kafka.default.svc.cluster.local

Each Kafka broker can be accessed by producers via port 9092 on the following DNS name(s) from within your cluster:

    kafka-0.kafka-headless.default.svc.cluster.local:9092
    kafka-1.kafka-headless.default.svc.cluster.local:9092
    kafka-2.kafka-headless.default.svc.cluster.local:9092

To create a pod that you can use as a Kafka client run the following commands:

    kubectl run kafka-client --restart='Never' --image docker.io/bitnami/kafka:3.4.0-debian-11-r2 --namespace default --command -- sleep infinity
    kubectl exec --tty -i kafka-client --namespace default -- bash

    PRODUCER:
        kafka-console-producer.sh \
            --broker-list kafka-0.kafka-headless.default.svc.cluster.local:9092,kafka-1.kafka-headless.default.svc.cluster.local:9092,kafka-2.kafka-headless.default.svc.cluster.local:9092 \
            --topic test

    CONSUMER:
        kafka-console-consumer.sh \
            --bootstrap-server kafka.default.svc.cluster.local:9092 \
            --topic test \
            --from-beginning


Extract pod name

export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=kafka,app.kubernetes.io/instance=kafka,app.kubernetes.io/component=kafka" -o jsonpath="{.items[0].metadata.name}")

***Create new topic***

kubectl --namespace default exec -it kafka-0 -- kafka-topics.sh --create --bootstrap-server kafka.default.svc.cluster.local:9092 --replication-factor 1 --partitions 1 --topic mytopic

***Start Consumer***

kubectl --namespace default exec -it kafka-0 -- kafka-console-consumer.sh --bootstrap-server kafka.default.svc.cluster.local:9092 --topic mytopic --consumer.config /opt/bitnami/kafka/conf/consumer.properties


***Start Producer***

kubectl --namespace default exec -it kafka-0 -- kafka-console-producer.sh --broker-list kafka.default.svc.cluster.local:9092 --topic mytopic --producer.config /opt/bitnami/kafka/conf/producer.properties

start a local consumer

kafka-console-consumer.sh --bootstrap-server kafka.default.svc.cluster.local:9092 --topic mytopic

**Adding new topic** 

kubectl --kubeconfig /Users/soumikdas/work/Rancher/cpe6041nkep.yaml --namespace esb exec -it kafka-0 -- kafka-topics.sh --create --bootstrap-server kafka-0.kafka-headless.esb.svc.cluster.local:9092 --replication-factor 1 --partitions 1 --topic mytopic


kubectl --kubeconfig /Users/soumikdas/work/Rancher/cpe6041nkep.yaml --namespace esb exec -it kafka-0 -- kafka-console-producer.sh --broker-list 10.250.107.3:9092 --topic mytopic


kubectl --kubeconfig /Users/soumikdas/work/Rancher/cpe6041nkep.yaml --namespace esb exec -it kafka-0 -- kafka-console-consumer.sh --broker-list 10.250.107.3:9092 --topic mytopic


kubectl --kubeconfig /Users/soumikdas/work/Rancher/cpe6041nkep.yaml --namespace esb exec -it kafka-0 -- kafka-topics.sh --list --bootstrap-server 10.250.107.3:9092


kafka-0.kafka-headless.esb.svc.cluster.local:9092

############ AKS Cluster ##############

az group create --name K8S --location eastus ; az aks create -g K8S -n aksdemocluster --enable-managed-identity --node-count 3 --enable-addons monitoring --enable-msi-auth-for-monitoring  --generate-ssh-keys;rm /Users/soumikdas/work/Rancher/aksconfig.yaml;az aks get-credentials --resource-group K8S --name aksdemocluster --file /Users/soumikdas/work/Rancher/aksconfig.yaml


istioctl install --set profile=demo -y

kubectl label namespace default istio-injection=enabled


apply -f ~/work/istio-1.17.1/samples/addons

##############

k create secret docker-registry acr-secret --namespace default --docker-server=tempesb.azurecr.io --docker-username=f1f840bb-b126-49e4-926b-d51468a13cf3 --docker-password=max8Q~HnlsKBFkfl_3irCaPrYg2KJ.PGTKvEfaSU

k apply -f sample.yaml

k apply -f sample-gateway.yaml


docker run -d --restart=unless-stopped \
  -p 80:80 -p 443:443 \
  --privileged \
  rancher/rancher:latest


### Set kubernetes secret from keyvault

az group create --location "westus" --name "DemoKeyVault"

az keyvault secret set --name value3 --vault-name "DemoKeyVaultESBMod" --value "from keyvault"

az keyvault create --name "DemoKeyVaultESBMod" --resource-group "DemoKeyVault"

APP_ID=$(az ad app create --display-name "external-secret-read" --query appId | tr -d \")

APP_ID a5f8fd60-e781-4f19-a213-e13987c7ef4d

SERVICE_PRINCIPAL=$(az ad sp create --id $APP_ID --query objectId | tr -d \")

SERVICE_PRINCIPAL

az ad app permission add --id $APP_ID --api-permissions f53da476-18e3-4152-8e01-aec403e6edc0=Scope --api cfa8b339-82a2-471a-a3c9-0fc0be7a4093
