# setup for operator development

Install/Update your operator-sdk cli tool
https://sdk.operatorframework.io/docs/installation/

Building a new scaffolding will automatically create an upgraded folder structure.

``` bash
mkdir archeobot
cd archeobot
operator-sdk init \
--plugins ansible \
--domain devops.gov.bc.ca

operator-sdk create api \
--group=artifactory \
--version=v1alpha1 \
--kind=ArtifactoryServiceAccount \
--generate-role \
--generate-playbook

operator-sdk create api \
--group=artifactory \
--version=v1alpha1 \
--kind=ArtifactoryRepository \
--generate-role \
--generate-playbook
```

Inside the directory, run the following command to prep your local development pre-reqs
``` bash
ansible-galaxy collection install -r requirements.yml
```

