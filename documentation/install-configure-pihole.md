# Install and Configure Pihole for Kubernetes

- Deploy [Persistent Volume](../helmCharts/pihole/pihole-persistentvolume.yaml) ## Is this needed with longhorn?
- Deploy [Persistent Volume Claim](../helmCharts/pihole/pihole-persistentvolumeclaim.yaml)

- Add repo to help and update

```bash
helm repo add mojo2600 https://mojo2600.github.io/pihole-kubernetes/ && helm repo update
```

-- Create Secret for pihole

```bash
kubectl create secret generic pihole-secret \
    --from-literal='password=<ADMIN_PASSWORD>' \
    --namespace pihole
```

- Make any changes required to [pihole values file](../helmCharts/pihole/pihole-values.yaml) to match required setup

- Install the helm chart
```bash
helm install pihole mojo2600/pihole \
  --namespace pihole \
  --values helmCharts/pihole/pihole-values.yaml
```