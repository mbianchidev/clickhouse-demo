These commands work based on your cluster name, in this case I've provisioned it and called it clickhouse-demo

```bash
# the name of the config will be matching your cluster name
mv civo-clickhouse-demo-kubeconfig $HOME/.kube/
export CIVOCONFIG=$HOME/.kube/civo-clickhouse-demo-kubeconfig
kubectl --kubeconfig=$CIVOCONFIG config get-contexts
# Connect to the Kubernetes cluster
kubectl --kubeconfig=$CIVOCONFIG config use-context clickhouse-demo
```
