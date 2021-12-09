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

It also makes use of the namespace license plate, hereby referred to as `namespace`.

When the user creates an ArtifactoryProject, it is assigned a `key` which is the first letter of the name and the first three letters of the namespace - so an Artifactory Project called "test" in "devops-artifactory" would have a key of `tdev`. It is also assigned a quota. 

Once the operator has completed reconciliation of the creation of the object, the following will have occurred:
- an ArtifactoryProjectApproval object with the name `[name]` will have been created in the same namespace.
- a message will have been sent to the Platform Services team over RocketChat requesting approval for the new object.

The platform team may then either approve or reject the request.

If the request is rejected, the `approval_status` of your ArtifactoryProject object will be updated to indicate the rejection. The rejection may be acknowledged by reverting all changes.

If the request is approved, once the operator has completed reconciliation of the approval, the following objects will have been created:
- a project in artifactory with a name of the format `[namespace]-[name]`, with the `admin-username` user granted administrative control over the project, with 5GB of quota.

From there, the admin user can perform all the relevant other tasks, such as making new repositories and adding other users (including service accounts) to the project. 

A user may change the quota listed in the ArtifactoryProject object. Doing so triggers another request for approval, with a process effectively identical to the process for approving the creation of a new project.

## Project Approval

A simple object which serves solely as a source of truth for the current approval status of the ArtifactoryProject object with the same name. These objects can be viewed by users, but all CRUD tasks are limited only to the Platform Services team. 

There is a similar `approval_status` field in the ArtifactoryProject object, but this exists solely for the ease of the user - editing this field does nothing and any edits will eventually be replaced by the `approval_status` from the ArtifactoryProjectApproval object.
