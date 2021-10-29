## RBAC

We will look at are in the rbac.authorization.k8s.io group. We actually have four resources: ClusterRole, Role, ClusterRoleBinding, and RoleBinding. They are used for Role Based Access Control (RBAC) to Kubernetes.

```shell
$ curl localhost:8080/apis/rbac.authorization.k8s.io/v1
```

```yaml
... 
    "groupVersion": "rbac.authorization.k8s.io/v1",
    "resources": [ 
... 
    "kind": "ClusterRoleBinding" 
... 
    "kind": "ClusterRole" 
... 
    "kind": "RoleBinding" 
... 
    "kind": "Role" 
...
```

These resources allow us to define Roles within a cluster and associate users to these Roles. For example, we can define a Role for someone who can only read pods in a specific namespace, or a Role that can create deployments, but no services.
