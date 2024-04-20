![alt text](image.png)

![alt text](image-1.png)


![Notifications](image-2.png)

# Create the ArgoCD service monitors below within the argocd namespace.


   1. argocd-metrics

   2. argocd-server-metrics

   3. argocd-repo-server-metrics

   4. argocd-applicationset-controller-metrics

Set metadata.labels.release to kode-kloud-prometheus-stack

# solution


Create a yaml definition for argocd-metrics:


vi argocd-metrics.yaml



Add below code in it:


apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: argocd-metrics
  namespace: argocd
  labels:
    release: kode-kloud-prometheus-stack
spec:
  selector:
    matchLabels:
      app.kubernetes.io/name: argocd-metrics
  endpoints:
  - port: metrics



Create a yaml definition for argocd-server-metrics:


vi argocd-server-metrics.yaml



Add below code in it:


apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: argocd-server-metrics
  namespace: argocd
  labels:
    release: kode-kloud-prometheus-stack
spec:
  selector:
    matchLabels:
      app.kubernetes.io/name: argocd-server-metrics
  endpoints:
  - port: metrics



Create a yaml definition for argocd-repo-server-metrics:


vi argocd-repo-server-metrics.yaml



Add below code in it:


apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: argocd-repo-server-metrics
  namespace: argocd
  labels:
    release: kode-kloud-prometheus-stack
spec:
  selector:
    matchLabels:
      app.kubernetes.io/name: argocd-repo-server
  endpoints:
  - port: metrics



Create a yaml definition for argocd-applicationset-controller-metrics:


vi argocd-applicationset-controller-metrics.yaml



Add below code in it:


apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: argocd-applicationset-controller-metrics
  namespace: argocd
  labels:
    release: kode-kloud-prometheus-stack
spec:
  selector:
    matchLabels:
      app.kubernetes.io/name: argocd-applicationset-controller
  endpoints:
  - port: metrics



To apply all templates run the below commands:


kubectl apply -f argocd-metrics.yaml
kubectl apply -f argocd-server-metrics.yaml
kubectl apply -f argocd-repo-server-metrics.yaml
kubectl apply -f argocd-applicationset-controller-metrics.yaml

![alt text](image-3.png)


Click on the Grafana button to access the Grafana UI, and log in using the credentials below.


   Username: admin

   Password: Fetch the password using below command
   kubectl -n monitoring get secrets kode-kloud-prometheus-stack-grafana -o json | jq .data'."admin-password"' -r | base64 -d

   Import https://grafana.com/grafana/dashboards/14584-argocd/ dashboard in Grafana with default options.

   ![alt text](image-4.png)
