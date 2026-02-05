# jenkins-setup
Install Minikube or Kind (for local Kubernetes cluster)

Minikube: brew install minikube && minikube start
Kind: brew install kind && kind create cluster
Install ArgoCD

kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
Expose ArgoCD API server (for local access)

kubectl port-forward svc/argocd-server -n argocd 8080:443
Login to ArgoCD UI

Open http://localhost:8080
Get admin password: kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
admin 8o7aky1l9hfn6aeV
Install ArgoCD CLI (optional, for CLI usage)

brew install argocd
Add Jenkins Helm repo and fetch latest chart

helm repo add jenkins https://charts.jenkins.io
helm repo update
Prepare a basic Jenkins values.yaml (you can override values as needed, e.g., admin password, persistence, etc.)

Create a Git repo (or local folder) with a HelmRelease manifest for Jenkins, referencing the latest chart and your values.yaml.

#kubectl port-forward svc/argocd-server -n argocd 8080:443

Example Application manifest (app.yaml):

Apply the Application manifest:
```
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: jenkins
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://charts.jenkins.io
    chart: jenkins
    targetRevision: latest
    helm:
      values: |
        controller:
          adminPassword: admin
          installPlugins:
            - kubernetes:latest
            - workflow-aggregator:latest
            - configuration-as-code:latest
            # ...add more as needed from your plugins.txt
  destination:
    server: https://kubernetes.default.svc
    namespace: jenkins
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
      selfHeal: true
```
kubectl create namespace jenkins
kubectl apply -f app.yaml
Access Jenkins:

kubectl port-forward svc/jenkins -n jenkins 8081:8080
Open http://localhost:8081
You can further customize the Jenkins setup by referencing your jcasc, jobs, and shared-library folders in the values.yaml using the Jenkins Configuration as Code (JCasC) plugin and Kubernetes ConfigMaps or Secrets.

Let me know if you want a ready-to-use values.yaml or app.yaml with your plugin list and JCasC integration!