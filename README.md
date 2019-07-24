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


## Datasource configuration

### Kubernetes

```
url: https://192.168.6.95:6443

TLS Client Autho: true

Skip TLS Verify: true
```
-- TLS Auth Details
```
Client Cert:
-----BEGIN CERTIFICATE-----
MIIC8jCCAdqgAwIBAgIIeL3bKLZZBrYwDQYJKoZIhvcNAQELBQAwFTETMBEGA1UE
AxMKa3ViZXJuZXRlczAeFw0xOTAyMjIxMjM3MDZaFw0yMDAyMjIxMjM3MDhaMDQx
FzAVBgNVBAoTDnN5c3RlbTptYXN0ZXJzMRkwFwYDVQQDExBrdWJlcm5ldGVzLWFk
bWluMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA9cHTRsqqzRims/2a
DaNEQbAUsoskYzV41aK3g+p2km6W93dtAEOkz71nGJVBQSdSS20DHqS91DFeFDUT
Zm+/4wNI13PurFSREMs79m88RtP3u7GercL16DlQr327xBdMs0QnUU15YJGQcTEb
Q/9ZUqisQGyBZMRp0c8tEo5lpsa6kHVgp/pnpeJHZkMCHS7Y2KMbjS+0yI3fkcZy
NI2vHqKnC9tXbtG0V9it05Kcoio0cHA/Q+ZvvKg3IdUHjYZlE0gdJdgmwVwec0Bw
9XssqZ5ZHASQ1LXHZhXeIv8/rIHaKQ+4DiogQFvz6U7J/Xp4Wg48lhUK9LI6ASaE
khOrIwIDAQABoycwJTAOBgNVHQ8BAf8EBAMCBaAwEwYDVR0lBAwwCgYIKwYBBQUH
AwIwDQYJKoZIhvcNAQELBQADggEBAHUk5PwhYL6gpgayCVZLLbn4czdojj0Ty4NP
to9ON0UK7mxbOBF5x6PKiFFIQwhySSeOOpBYwHzwdTywHF6Ja8XE2q0tOyC4SbPd
q7DKZNMTnCq/Poi/plk/s0bUV6HHIS67rslXXdCGAMSrODCd3gcRTz5sbNHH3LgF
6CFshxrfj7PLy6fHXh/X9KTAncUmsF5tQGYUP/neoFMapNV3lFsvCsfJoSkaUkOz
umFgn/q2KTu8IGnRNemDZobDT9sxkjin+I+66AF4nLW4K2Revi0cIi7tywOHEPW7
sWtFEIlE0yXMvxPFND75/rRnCFo0F7sxvi92HCPJDEjTHU+HJeI=
-----END CERTIFICATE-----

 
Client Key: 
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEA9cHTRsqqzRims/2aDaNEQbAUsoskYzV41aK3g+p2km6W93dt
AEOkz71nGJVBQSdSS20DHqS91DFeFDUTZm+/4wNI13PurFSREMs79m88RtP3u7Ge
rcL16DlQr327xBdMs0QnUU15YJGQcTEbQ/9ZUqisQGyBZMRp0c8tEo5lpsa6kHVg
p/pnpeJHZkMCHS7Y2KMbjS+0yI3fkcZyNI2vHqKnC9tXbtG0V9it05Kcoio0cHA/
Q+ZvvKg3IdUHjYZlE0gdJdgmwVwec0Bw9XssqZ5ZHASQ1LXHZhXeIv8/rIHaKQ+4
DiogQFvz6U7J/Xp4Wg48lhUK9LI6ASaEkhOrIwIDAQABAoIBACC5sb+whzQOf0xi
jdwZDKLOpsLrwmmvmiqgo11eoHF5ZoMHlS0+1LiRGSRt46WgbdX7aznuaBTUihmY
w7+VS/EX4+BE4Nhz3mllFtQHFfi8izWkPmQXHRXSZAsqbBF9pMoOXkn2Th5s49Ye
2umgHC3kpiNiD4zylsDInNDmw2SEuU9boyFpKU7B/6BLcHbFQdtYCwcbnJYJk9++
Ib/pfWm5+cPEDrhmn3H+vAx9lLAxADZ1FD9BaV6XFl7q9N11FmpKmbR+6o/reLNp
ccx+smD2kkY4K8LKZw88eYVmPwCx4HB2xVYW9jdHDb9p/FkI76hbHUL3l23izgUS
i9V0ONECgYEA+LhcXfKr6w3Hz9z/XYrlr/1Kc8zLBF20C9Gbon81SIX7IBuBQ14U
zGdew0QrycIbBbesbWwbZaEJ6aY9G+HuQKatETsLbFiURuzb2DIAlcr67VA8I1IQ
+/QQsOsK0795iJYFDfXiJNCQRiqmkCm4jiItJ+IqbZgRaSPAKjwfNbkCgYEA/PND
RFPZgo0nGUbR+s7XAGqlpZONRHLELlrRNcpP8WZOMJ2iboTNsJFIBHSo4RRmljMU
huJViZgwAczeec/aUvdxtvZeV3g9IaBwAGc8ySiXapl2LNRVtsZ8uIfWuTkwEAgb
xaxSiJYKIWzhjU/VQVUaUr4KhMuIvDeWBhZ5VbsCgYBDa442NULe65RfRzO9wpny
c8GL1Fav70qP7Zi3mq3x48en82y9uzH+GoM4gTExdrlmelx2KNjgWp/aQyLLfRnd
UpEVW6EEFJrVAv2xBBTehfAxBg/XLzbFZWpk2sHLllq2aJwkJaPQgOyq6ILQD08k
0CTXa9o+bPtDOdqsWDHJmQKBgHx1XSWjdCROO2yucebMGvGzh6l+fkWtimWcfc/P
qaIHSnWVOjTS1zoHYb3/gJCurwM8Qt9TQe8fmI9qNBUPdkbYRXVWp3i2Sq3e+PzZ
zwjTFh13QLQyDbKO2xMYk0gzoThiJPgQH9Pgrz9fCWO0YiNxMjCAHUDVvIOPfhuk
tzK9AoGBANb8+BSGDwS3XGAxbMfa/GBGTzt8lOvKdcUPq/K63G+hyxRLvE7bucO3
5AS2CrxsVhUvb7QoO1NM3zEdaKnCVeXQOj5TjZhmbhPovNB9lhLGrRV5o6xwvtIM
+HjBc+a9DLwi+wTCU5bqs1LxIjDu405SDgrYgrvAZEDmsHVwPv4s
-----END RSA PRIVATE KEY-----

```

### Credentials Prometheus
```
url: http://prometheus:9090
TLS Client Autho: false

Skip TLS Verify: false
```

### Credentials to Datasource influxDB
```
url: http://192.168.6.95:32339
user: root
pass: root
database: k8s
```



#### REFERÊNCIAS

https://medium.com/htc-research-engineering-blog/monitoring-kubernetes-clusters-with-grafana-e2a413febefd

https://github.com/grafana/kubernetes-app/issues/35

https://github.com/kubernetes/kube-state-metrics/issues/295

https://medium.com/faun/configuring-ha-kubernetes-cluster-on-bare-metal-servers-monitoring-logs-and-usage-examples-3-3-340357f21453