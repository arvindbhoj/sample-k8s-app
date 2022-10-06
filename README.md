# sample-k8s-app
This is a demo to show how Flux CD can be used to deploy different flavors of apps to different k8s environments.
>This assumes that you already have Flux CD deployed and configured. Also it uses Traefik 2.x for ingress.
>A DKP Enterprise cluster already has this setup and ready to go
 
Simply follow the steps given below to deploy these to your cluster
### Step 1
Link this git repository to the two k8s clusters that will be used in this demo

```
kubectl create ns appdev #Note: If using DKP Enterprise then the namespace can be a project

export NAMESPACE=appdev
kubectl apply -f - <<EOF
apiVersion: source.toolkit.fluxcd.io/v1beta1
kind: GitRepository
metadata:
  name: demo-repo-ship
  namespace: ${NAMESPACE}
spec:
  interval:  5s
  ref:
    branch: master
  timeout: 20s
  url: https://github.com/arbhoj/sample-k8s-app
EOF
```

### Step: 2

Apply the kustomization to the first cluster

```
k apply -f - <<EOF
apiVersion: kustomize.toolkit.fluxcd.io/v1beta2
kind: Kustomization
metadata:
  name: shipwelcome
  namespace: ${NAMESPACE}
spec:
  patches:
   - patch: |
       - op: replace
         path: /metadata/namespace
         value: ${NAMESPACE}
     target:
       kind: Deployment
   - patch: |
       - op: replace
         path: /metadata/namespace
         value: ${NAMESPACE}
     target:
       kind: Service
   - patch: |
       - op: replace
         path: /metadata/namespace
         value: ${NAMESPACE}
     target:
       kind: ConfigMap
   - patch: |
       - op: replace
         path: /metadata/namespace
         value: ${NAMESPACE}
     target:
       kind: Ingress
   - patch: |
       - op: replace
         path: /metadata/namespace
         value: ${NAMESPACE}
     target:
       kind: Middleware
  postBuild:
    substitute:
      middleware: ${NAMESPACE}-welcomeglobal-stripprefixes@kubernetescrd      
  interval: 5s
  path: ./ships/allure
  prune: true
  serviceAccountName: ${NAMESPACE}
  sourceRef:
   kind: GitRepository
   name: demo-repo-ship
   namespace: ${NAMESPACE}
EOF
```

Step 3:

Apply the kustomization to the second cluster

```
k apply -f - <<EOF
apiVersion: kustomize.toolkit.fluxcd.io/v1beta2
kind: Kustomization
metadata:
  name: shipwelcome
  namespace: ${NAMESPACE}
spec:
  patches:
   - patch: |
       - op: replace
         path: /metadata/namespace
         value: ${NAMESPACE}
     target:
       kind: Deployment
   - patch: |
       - op: replace
         path: /metadata/namespace
         value: ${NAMESPACE}
     target:
       kind: Service
   - patch: |
       - op: replace
         path: /metadata/namespace
         value: ${NAMESPACE}
     target:
       kind: ConfigMap
   - patch: |
       - op: replace
         path: /metadata/namespace
         value: ${NAMESPACE}
     target:
       kind: Ingress
   - patch: |
       - op: replace
         path: /metadata/namespace
         value: ${NAMESPACE}
     target:
       kind: Middleware
  postBuild:
    substitute:
      middleware: ${NAMESPACE}-welcomeglobal-stripprefixes@kubernetescrd      
  interval: 5s
  path: ./ships/harmony
  prune: true
  serviceAccountName: ${NAMESPACE}
  sourceRef:
   kind: GitRepository
   name: demo-repo
   namespace: ${NAMESPACE}
EOF
```

Now open a browser and view the deployed application at the following endpoint for both the clusters:
https://<cluster-url>/welcomeglobal/ 

Now to demo changes, Make changes to the configmap at the global level or the ship level and commit and see it automaically reflect in the application
