# permission-manager

Managing Kubernetes access means creating users, assigning permissions, and getting a working kubeconfig to each person. permission-manager puts those tasks behind a web UI, with permission templates backed by Kubernetes RBAC resources.

## Features

- Create and delete users through a web interface.
- Assign users permissions within namespaces or across the cluster.
- Generate kubeconfig YAML files for distribution to users.
- Define reusable permission templates with Kubernetes ClusterRoles.

## Installation

You need access to a Kubernetes cluster and permission to install the application's resources and configure RBAC. For local TypeScript development, install Node.js and npm.

From a checkout of this project's repository, install dependencies and inspect the available scripts:

```sh
cd permission-manager
npm install
npm run
```

Use the scripts declared in `package.json` for local development. The supplied project information does not specify startup scripts, so no particular script name is assumed here.

For cluster deployment, consult the [upstream installation guide](https://github.com/sighupio/permission-manager/blob/master/docs/installation.md). Check its manifests and configuration against this project's checkout before applying them.

**Review RBAC permissions before deployment.** This application manages cluster access; the permissions you grant it and the templates you expose deserve the same review as other administrative tooling.

## Usage

After deployment, connect to the web UI through the Service configured in your installation. For example, if that Service is named `permission-manager`, is in the `permission-manager` namespace, and exposes port `80`, run:

```sh
kubectl --namespace permission-manager port-forward service/permission-manager 8080:80
```

Replace the namespace, Service name, and Service port with your deployment's values. Open `http://localhost:8080` and use the UI to create a user, select a namespace and permission template, then obtain the user's kubeconfig YAML.

Templates are ClusterRoles whose names start with `template-namespaced-resources___`. For example, this template grants read access to pods and their logs:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: template-namespaced-resources___pod-reader
rules:
  - apiGroups: [""]
    resources: ["pods", "pods/log"]
    verbs: ["get", "list", "watch"]
```

Save it as `pod-reader.yaml` and apply it to the cluster:

```sh
kubectl apply -f pod-reader.yaml
```

Select that template when assigning permissions in the UI. The template system uses ClusterRoles, RoleBindings, and ClusterRoleBindings. User records are custom resources of kind `permissionmanagerusers.permissionmanager.user`.

## License

Licensed under BSD-3-Clause. This project is based on [sighupio/permission-manager](https://github.com/sighupio/permission-manager), originally developed by SIGHUP. Retain the upstream copyright notice, license conditions, and disclaimer when redistributing source or binaries, as required by the license.
