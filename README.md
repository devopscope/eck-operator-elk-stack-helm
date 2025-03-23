# eck-operator-elk-stack-helm
A working example configuration of ECK operator with ELK stack.
These links below helped a bit:
- https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-quickstart.html
- https://discuss.elastic.co/t/kubernetes-metadata-missing-with-add-kubernetes-metadata-enabled/254314/6 (especially with add_kubernetes_metadata, sa, crb, cr topics)

# Install kind
brew install kind
kind create cluster -n elk
k get nodes

# Install ECK Operator
helm repo add elastic https://helm.elastic.co
helm repo update
helm install elastic-operator elastic/eck-operator -n elastic-system --create-namespace
kubens elastic-system
k logs sts/elastic-operator
k get pods
k get pods
k get all
k get elastic
k get elasticsearch
helm show values elastic/eck-operator

# Install elasticsearch
k apply -f elasticsearch.yaml
k get pods
k describe crd elasticsearch
k get elasticsearch
k get svc quickstart-es-http
PASSWORD=$(kubectl get secret quickstart-es-elastic-user -o go-template='{{.data.elastic | base64decode}}')
k port-forward service/quickstart-es-http 9200
curl -u "elastic:$PASSWORD" -k "https://localhost:9200"

# Install kibana
k apply -f kibana.yaml
k get kibana
k get pod --selector='kibana.k8s.elastic.co/name=quickstart'
k get service quickstart-kb-http
k port-forward service/quickstart-kb-http 5601
k get secret quickstart-es-elastic-user -o=jsonpath='{.data.elastic}' | base64 --decode; echo

# Install beat
k apply -f beat.yaml
k get beat
k port-forward service/quickstart-es-http 9200
curl -u "elastic:$PASSWORD" -k "https://localhost:9200/filebeat-*/_search"
k get beat
k logs -f statefulset.apps/elastic-operator
k get po

# Install logstash
k apply -f logstash.yaml

# Install a pod printing to console in 10 secs perionds
k apply -f pod-hede-printer.yaml
k logs -f hede-printer

# Create index
Check if there are indexes starting with logstash-*
curl -X GET -u "elastic:$PASSWORD" -k "https://localhost:9200/_cat/indices?v"
Go to Discover => Data View => Create a data view, give "logstash-*" as index pattern
