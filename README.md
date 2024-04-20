# GitOps-with-ArgoCD
GitOps with ArgoCD

# Important commands

Create a new ArgoCD account using the following credentials.


    Account Name: alice

    Capabilities: apikey, login

solution - kubectl -n argocd patch configmap argocd-cm --patch='{"data":{"accounts.alice": "apiKey,login"}}



Create a new role with create application access using the details below and assign it to user alice.


    Role Name: create-app

    Project: any

Solution   kubectl -n argocd patch configmap argocd-rbac-cm \
--patch='{"data":{"policy.csv":"p, role:create-app, applications, create, *, allow\ng, alice, role:create-app"}}'