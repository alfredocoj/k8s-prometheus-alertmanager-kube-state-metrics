### K8S-GRAFANA-PROMETHEUS

kubectl apply -f namespace.yaml

$ kubectl apply -f prometheus-rbac.yaml

Se houve falha

$ kubectl create clusterrolebinding cluster-admin-binding --clusterrole=cluster-admin --user=${YOUR_EMAIL_ADDRESS}

create the config for prometheus to collect data from Kubernetes


$ kubectl apply -f prometheus-config.yaml

create the deployment of prometheus

$ kubectl apply -f prometheus-deploy.yaml

create the prometheus service

$ kubectl -f apply prometheus-svc.yaml


create the grafana deployment

kubectl apply -f grafana.yaml


expose grafana deployment with kubectl expose

$ kubectl expose deployment grafana --type=LoadBalancer --namespace=monitoring

Get the IP via `$ kubectl get svc -n monitoring`

Now you can access Grafana via http://{IP}:3000/


create node-exporter daemon set to export node information

kubectl apply -f node-exporter.yaml


create kube-state-metrics deployment to collect metrics about the cluster

kubectl apply -f state-metrics-deploy.yaml


create the service account and cluster role with the permission of getting cluster info for state-metrics.

kubectl apply -f state-metrics-rbac.yaml


Now, configure your cluster setting on Grafana.

Enable the app at http://{IP}:3000/plugins/grafana-kubernetes-app/edit
Add a data source of prometheus type: http://{IP}:3000/datasources/new


Add a cluster at http://{IP}:3000/plugins/grafana-kubernetes-app/page/cluster-config with username/password and CA certificate.

#### REFERÊNCIAS

https://medium.com/htc-research-engineering-blog/monitoring-kubernetes-clusters-with-grafana-e2a413febefd