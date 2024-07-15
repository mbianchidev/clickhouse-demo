```bash
# Connect to the Kubernetes cluster
mv civo-kubeconfig.yaml $HOME/.kube/
export KUBECONFIG=$HOME/.kube/civo-kubeconfig.yaml
kubectl config get-contexts
kubectl config use-context <context-name>
```