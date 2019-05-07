### K8S-GRAFANA-PROMETHEUS

kubectl apply -f namespace.yaml

create the service account and cluster role with the permission of getting cluster info for prometheus.

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

FYI, I finally got this working. (Woo hoo!) Details are as follows:

all the needed data is stored in your .kube/config YAML file
paste the url stored in the "clusters.cluster.server" .kube/config field into the "HTTP URL" field for the plugin
check the boxes "TLS Client Auth" and "With CA Cert"
fill out the "CA Cert", "Client Cert", and "Client Key" fields like so:
the CA Cert value is obtained by taking the value of the "clusters.cluster.certificate-authority-data" .kube/config field and base64 decoding it (e.g., pipe it into "base64 -d")
For Client Cert, same as previous step, but use "users.user.client-certificate-data"
For Client Key, same as previous step, but use "users.user.client-key-data"
Hope this is helpful.

FYI to Grafana people: it would probably be good if somebody documents this somewhere!

#### REFERÊNCIAS

https://medium.com/htc-research-engineering-blog/monitoring-kubernetes-clusters-with-grafana-e2a413febefd

https://github.com/grafana/kubernetes-app/issues/35

https://github.com/kubernetes/kube-state-metrics/issues/295