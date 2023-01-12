This gist is based on
[Kubernetes Dashboard](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/}
deploy docs. I think you have installed the
[Nginx Iingress Controller](https://kubernetes.github.io/ingress-nginx).

## 1) Deploy the Dashboard UI

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0/aio/deploy/recommended.yaml
```

## 2) Creating the Service Account and ClusterRoleBinding

```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
EOF
```

## 3) Get a Bearer Token

Now we need to find token we can use to log in. Execute following command:

For Bash:

```bash
kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')
```

For Powershell:
```powershell
kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | sls admin-user | ForEach-Object { $_ -Split '\s+' } | Select -First 1)
```

It should print the data with line like:

```
token: <YOUR TOKEN HERE>
```

Now save it. You need to use it whe login the dashboard.


## 4) Create the ingress controller

```bash
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  namespace: kubernetes-dashboard
  name: kubernetes-dashboard-ingress
  annotations:
    cert-manager.io/cluster-issuer: 'letsencrypt-prod'
    kubernetes.io/tls-acme: 'true'
    kubernetes.io/ingress.class: 'nginx'
    nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
spec:
  tls:
    - hosts:
        - dashboard.khuonviet.vn
      secretName: 'kubernetes-dashboard-cert'
  rules:
    - host: dashboard.khuonviet.vn
      http:
        paths:
          - pathType: Prefix
            path: '/'
            backend:
              service:
                name: kubernetes-dashboard
                port:
                  number: 443
EOF
```

## 5) Login to dashboard

Go to `https://<YOUR DOMAIN>` and insert the previous created token into `Enter token` field.
