1- install k3s:
Install
PUBLIC_IP=$(curl -s https://ipv4.icanhazip.com)

curl -sfL https://get.k3s.io | sh -s - server \
 --tls-san "$PUBLIC_IP" --disable traefik

get worker registration token:
sudo cat /var/lib/rancher/k3s/server/node-token

register worker:
curl -sfL https://get.k3s.io | \
 K3S_URL=https://167.172.105.159:6443 \
 K3S_TOKEN=K100cfc8c562b71cfe2f9fa502be60a52df54a667fb42419bcddced02b647824047::server:b482c8d35ccc0d0535f156f80f5be866 \
 sh -

export KUBECONFIG=/etc/rancher/k3s/k3s.yaml

uninstall k3s on worker:
sudo /usr/local/bin/k3s-agent-uninstall.sh

uninstall k3s on control plane:
/usr/local/bin/k3s-uninstall.sh

2- install argocd:

- kubectl create namespace argocd
- kubectl apply -n argocd --server-side --force-conflicts -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

3- access argocd:

- kubectl -n argocd port-forward svc/argocd-server 8080:443
- https://localhost:8080
- Get the initial admin password: kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
- Login username is: admin
- After changing the password, delete the initial secret: kubectl -n argocd delete secret argocd-initial-admin-secret

4 - apply argocd health: kubectl apply -f argocd-application-health.yaml

5 - Restart the relevant Argo CD components:

- kubectl -n argocd rollout restart deployment argocd-server
- kubectl -n argocd rollout restart statefulset argocd-application-controller

6- Connect to github through the UI

7- kubectl apply -f argocd/root-app.yaml

# CLEANING

- kubectl -n argocd delete application platform-root
- kubectl delete namespace database
- kubectl delete namespace cnpg-system
- kubectl patch svc barman-cloud -n cnpg-system --type=merge -p '{"metadata":{"finalizers":[]}}'
- kubectl delete svc barman-cloud -n cnpg-system
- kubectl delete namespace cert-manager
- kubectl get crd -o name | Select-String "postgresql.cnpg.io|barmancloud.cnpg.io|cert-manager.io|acme.cert-manager.io" | ForEach-Object { kubectl delete $\_.Line }
