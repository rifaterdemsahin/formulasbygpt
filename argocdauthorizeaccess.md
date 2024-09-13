xTo demonstrate proof of concept for authorizing access to user information with Argo CD, you can follow these steps:

1. **Set Up Argo CD**:
   - Ensure you have Argo CD installed and running in your Kubernetes cluster. You can follow the [official installation guide](https://argo-cd.readthedocs.io/en/stable/getting_started/) for this.

2. **Configure Access Control**:
   - Argo CD uses Kubernetes RBAC for access control. To configure user access, you need to create and apply appropriate RBAC roles and role bindings. For example:
   
     ```yaml
     apiVersion: rbac.authorization.k8s.io/v1
     kind: Role
     metadata:
       name: argocd-role
       namespace: argocd
     rules:
       - apiGroups: ["argoproj.io"]
         resources: ["applications"]
         verbs: ["get", "list", "watch"]
     ---
     apiVersion: rbac.authorization.k8s.io/v1
     kind: RoleBinding
     metadata:
       name: argocd-role-binding
       namespace: argocd
     subjects:
       - kind: User
         name: my-user
         apiGroup: rbac.authorization.k8s.io
     roleRef:
       kind: Role
       name: argocd-role
       apiGroup: rbac.authorization.k8s.io
     ```

3. **Create and Configure `argocd-rbac-cm` ConfigMap**:
   - Argo CD uses a ConfigMap named `argocd-rbac-cm` to define RBAC policies. Update this ConfigMap to grant access permissions. For instance:

     ```yaml
     apiVersion: v1
     kind: ConfigMap
     metadata:
       name: argocd-rbac-cm
       namespace: argocd
     data:
       policy.csv: |
         p, role:admin, applications, get, /api/v1/users/*, allow
         p, role:admin, applications, list, /api/v1/users/*, allow
         p, role:admin, applications, watch, /api/v1/users/*, allow
       policy.default: role:readonly
     ```

4. **Apply Configuration**:
   - Apply the Role, RoleBinding, and ConfigMap configurations to your cluster using `kubectl apply -f <file>.yaml`.

5. **Test Access**:
   - Verify that the user `my-user` has the appropriate access to user information via Argo CD. You can do this by logging in as `my-user` and checking if the access permissions are applied correctly.

These steps provide a basic outline for configuring and testing user access with Argo CD. Adjust the RBAC rules and policies according to your specific needs.