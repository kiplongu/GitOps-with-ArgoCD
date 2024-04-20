![alt text](image.png)

![alt text](image-1.png)


# Deploy the Bitnami sealed secret controller by creating an Argocd application using either UI or CLI and synchronize the app using the following details.


   Application Name: sealed-secrets

   Project Name: default

   Sync Policy: auto

   Repository URL: https://bitnami-labs.github.io/sealed-secrets

   Repository Type: Helm

   Chart: sealed-secrets

   Version: 2.2.0

   Cluster URL: https://kubernetes.default.svc

   Namespace: kube-system


Make sure the application is in Healthy state before hitting the Check button.



You can access the ArgoCD UI and ArgoCD CLI by using the following credentials.

    User: admin

    Password: admin123