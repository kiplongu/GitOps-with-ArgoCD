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


# Install kubeseal CLI version 0.18.0. For help installing KubeSeal, click on the KubeSeal button at the top of the workspace.

wget https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.18.0/kubeseal-0.18.0-linux-amd64.tar.gz
tar -xvzf kubeseal-0.18.0-linux-amd64.tar.gz kubeseal
sudo install -m 755 kubeseal /usr/local/bin/kubeseal


# Extract the sealed-secrets controller TLS certificate from the kube-system namespace and store it in a file named /root/sealedSecret-publicCert.crt.


kubectl -n kube-system get secret sealed-secrets-keyfxtkf -o json | jq -r .data'."tls.crt"' | base64 -d > /root/sealedSecret-publicCert.crt


# Create /root/mysql-password_k8s-secret.yaml specification file for a generic Kubernetes secret called app-crds and add the secret data below to it.


   username: admin-dev-group

   password: paSsw0rD-1erT-diS

   apikey: zaCELgL-0imfnc8mVLWwsAawjYr4Rx-Af50DDqtlx
# solution
kubectl create secret generic app-crds --from-literal=apikey=zaCELgL-0imfnc8mVLWwsAawjYr4Rx-Af50DDqtlx --from-literal=username=admin-dev-group --from-literal=password=paSsw0rD-1erT-diS -o yaml --dry-run=client > mysql-password_k8s-secret.yaml



# Use kubeseal CLI to create a sealed secret from the previously created definition file i.e mysql-password_k8s-secret.yaml. The sealed secrets should be stored in mysql-password_sealed-secret.yaml file.


Further find below more details:


   Scope: cluster-wide

   Format: yaml

   Certificate: /root/sealedSecret-publicCert.crt

   # solution

   kubeseal -o yaml --scope cluster-wide --cert sealedSecret-publicCert.crt < mysql-password_k8s-secret.yaml > mysql-password_sealed-secret.yaml

Replace the contents of gitops-argocd/sealed-secret/secret.yaml definition file in gitops-argocd repository with the contents of mysql-password_sealed-secret.yaml file that you just created in the previous question. You can either directly update this file on Gitea, or clone and add/push your changes from the command line.



# Install argocd-vault-plugin version 1.12.0. For help, click the ArgocdVaultPlugin button at the top of the workspace.

curl -Lo argocd-vault-plugin https://github.com/argoproj-labs/argocd-vault-plugin/releases/download/v1.12.0/argocd-vault-plugin_1.12.0_linux_amd64
chmod +x argocd-vault-plugin
mv argocd-vault-plugin /usr/local/bin


There is a Kubernetes secret definition file /root/secret.yaml. The file has placeholders for apikey:, username: and password:. We already have secrets added in the Vault server that contain the original values for these placeholders.


# Use argocd-vault-plugin and generate a secret manifest/definition file called /root/secret_updated.yaml with all the placeholders replaced with the actual values from the Vault server. Find below some useful details:


   Vault Env File: /root/vault.env

   Placeholder Secret Definition File: /root/secret.yaml

   Final Secret Definition File: /root/secret_updated.yaml

   # solution

   argocd-vault-plugin generate -c /root/vault.env - < /root/secret.yaml > /root/secret_updated.yaml



   Edit the argocd-repo-server deployment to add an init container to make argocd-vault-plugin utility available within this pod. To do this, use the following steps.



Add the Initcontainer below:


      - name: download-tools
        image: alpine:3.8
        command: [sh, -c]
        env:
          - name: AVP_VERSION
            value: "1.7.0"
        args:
          - >-
            wget -O argocd-vault-plugin
            https://github.com/argoproj-labs/argocd-vault-plugin/releases/download/v${AVP_VERSION}/argocd-vault-plugin_${AVP_VERSION}_linux_amd64 &&
            chmod +x argocd-vault-plugin &&
            mv argocd-vault-plugin /custom-tools/
        volumeMounts:
          - mountPath: /custom-tools
            name: custom-tools



Add a Volume:

- name: custom-tools
   emptyDir: {}



Mount this volume under argocd-repo-server container:

- name: custom-tools
  mountPath: /usr/local/bin/argocd-vault-plugin
  subPath: argocd-vault-plugin



Edit argocd-cm Configmap to add below data section:

  configManagementPlugins: |-
    - name: argocd-vault-plugin
      generate:
        command: ["argocd-vault-plugin"]
        args: ["generate", "./"]



You can take further help from here.

# solution

Edit argocd-repo-server deployment:


kubectl edit deployments.apps -n argocd argocd-repo-server



Under initContainers: add a new init container as below:


      - name: download-tools
        image: alpine:3.8
        command: [sh, -c]
        env:
          - name: AVP_VERSION
            value: "1.7.0"
        args:
          - >-
            wget -O argocd-vault-plugin
            https://github.com/argoproj-labs/argocd-vault-plugin/releases/download/v${AVP_VERSION}/argocd-vault-plugin_${AVP_VERSION}_linux_amd64 &&
            chmod +x argocd-vault-plugin &&
            mv argocd-vault-plugin /custom-tools/
        volumeMounts:
          - mountPath: /custom-tools
            name: custom-tools



Under volumes: add a Volume as below:


- name: custom-tools
   emptyDir: {}



For argocd-repo-server container add a VolumeMount as below:


- name: custom-tools
  mountPath: /usr/local/bin/argocd-vault-plugin
  subPath: argocd-vault-plugin



Finally save the changes


Now edit argocd-cm Configmap:


kubectl edit -n argocd cm argocd-cm



At the end of the file add a data section, take care of the indentation to make sure it is aligned with the metadata: section:


data:
  configManagementPlugins: |-
    - name: argocd-vault-plugin
      generate:
        command: ["argocd-vault-plugin"]
        args: ["generate", "./"]