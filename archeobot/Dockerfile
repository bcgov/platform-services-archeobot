FROM quay.io/operator-framework/ansible-operator:v1.8.0

COPY requirements.yaml ${HOME}/requirements.yaml
RUN ansible-galaxy collection install -r ${HOME}/requirements.yaml \
 && chmod -R ug+rwx ${HOME}/.ansible

COPY watches.yaml ${HOME}/watches.yaml
COPY roles/ ${HOME}/roles/
COPY playbooks/ ${HOME}/playbooks/
COPY test/ ${HOME}/test/
