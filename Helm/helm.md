03/11/2023

## Monitoring stack

kube-prometheus-stack

## Set the helm repo
```
helm repo add prom-repo https://prometheus-community.github.io/helm-charts
```
## Install the helm repo

```
helm install monitoring prom-repo/kube-prometheus-stack
```


minikube service monitoring-grafana

helm upgrade monitoring prom-repo/kube-prometheus-stack --values=values.yaml


helm pull prom-repo/kube-prometheus-stack --untar


helm template ./kube-prometheus-stack/ --values=my_values.yaml > monitoring-stack.yaml

### All files in the template folder needs to be yaml

### It is written in GO lang

We can create our own chart using 

helm create <projectname>

The directory structure is like this

--charts
--templates
chart.yaml
values.yaml

all the core yaml files go in the template folder and whatever needs to be replaced using values we need to write within the {{ }} . The values is a hierarchical structure and we need to use "Values.<name>.<name>"

once the 

helm template . 

command is executed all the yaml files will be combined in to one yaml file and the out put will be provided

## Functions and Pipelines in the helm templates

we can use default function as {{ default <default value> .Values.value}}



