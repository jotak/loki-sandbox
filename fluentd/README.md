## Fluentd + Loki

First, set up ovn-k in Kind: see https://github.com/ovn-org/ovn-kubernetes/blob/master/docs/kind.md

```bash
kubectl apply -f ./fluentd-netflow.yaml

FD_IP=`kubectl get svc fluentd -ojsonpath='{.spec.clusterIP}'` && echo $FD_IP
kubectl set env daemonset/ovnkube-node -c ovnkube-node -n ovn-kubernetes OVN_NETFLOW_TARGETS="$FD_IP:2055"
# or:
kubectl set env daemonset/ovnkube-node -c ovnkube-node -n ovn-kubernetes OVN_IPFIX_TARGETS="$FD_IP:2055"

helm upgrade --install loki grafana/loki-stack --set promtail.enabled=false
```

## Check Grafana

See also https://grafana.com/docs/loki/latest/installation/helm/#deploy-grafana-to-your-cluster

```bash
helm install loki-grafana grafana/grafana
kubectl get secret loki-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
kubectl port-forward svc/loki-grafana 3000:80
```

Open http://localhost:3000/
Login with admin + printed password
Add datasource => Loki => http://loki:3100/

Example of queries:

- Top 10 sources by volumetry (1 min-rate):

`topk(10, (sum by(src) ( rate({ source="netflow" } | logfmt | __error__="" | unwrap in_bytes [1m]) )))`

- Top 10 destinations for a given source (1 min-rate):

`topk(10, (sum by(dst) ( rate({ source="netflow",src="10.244.1.12" } | logfmt | __error__="" | unwrap in_bytes [1m]) )))`


## Build fluentd image

E.g. with podman / user `jotak` (different user? => update also the image name in deployment yaml)

```bash
podman build -t quay.io/jotak/fluentd-flows-loki:netflow .
podman push quay.io/jotak/fluentd-flows-loki:netflow
```

## TODO

- kube enrichment
