# Architectural Documentation

Archeobot is an operator made with the [Operator SDK](https://sdk.operatorframework.io/docs/). It is an Ansible-based operator.

## CRD Definitions

### Service Accounts

When creating a service account, the CRD accepts the following information:

- a `name` (required)
- a description (optional)

It also makes use of the namespace license plate, hereby referred to as `namespace`. It also generates a unique 6-digit key to assure the uniqueness of the service account name  (hereby referred to as `nameplate`) and another 6-digit key which is regenerated and replaced when the password is updated (hereby referred to as `secretplate`)

Once the operator has completed reconciliation, the following objects will have been created:

- a service account within artifactory, with access to all cached repositories, and a name of the format `[name]-[namespace]-[nameplate]`
- a key/value secret in the relevant openshift namespace with a name of the format `artifacts-[name]-[secretplate]`, containing the username and password for the artifactory service account.
- a pull secret in the relevant openshift namespace with a name of the format `artifacts-pull-[name]-[secretplate]` which can be used as a pull secret in deployments and builds.

These are all fully usable the moment the ArtSA object is fully reconciled.

### Projects

When creating a project, the CRD accepts the following information:

- a `name` (required)
- a `admin-username` (required)
- a description (optional)

It also makes use of the namespace license plate, hereby referred to as `namespace`.

How should we approve the project?
- we can make a web-based request system where we approve the request and the system reaches out to create the CR in the appropriate namespace.
- teams can create the CR on their own and we patch it to approve (but then how do they request changes to things like quota?)
- teams can create one CR called like "ArtifactoryProjectRequest" and have full edit access to that and, once we approve those changes, the operator creates a second one called "ArtifactoryProjectApproved" which actually creates the project.

Once the operator has completed reconciliation of the approval, the following objects will have been created:

- a project in artifactory with a name of the format `[namespace]-[name]`, with the `admin-username` user granted administrative control over the project, with 5GB of quota.

From there, the admin user can perform all the relevant other tasks, such as making new repositories and adding other users (including service accounts) to the project. 
