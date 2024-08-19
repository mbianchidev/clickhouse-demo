```bash
# the name of the config will be matching your cluster name
mv civo-clickhouse-demo-kubeconfig $HOME/.kube/
export CIVOCONFIG=$HOME/.kube/civo-clickhouse-demo-kubeconfig
kubectl --kubeconfig=$CIVOCONFIG config get-contexts
# Connect to the Kubernetes cluster
kubectl config use-context clickhouse-demo
```
