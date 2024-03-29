# VERSION defines the project version for the bundle. 
# Update this value when you upgrade the version of your project.
# To re-generate a bundle for another specific version without changing the standard setup, you can:
# - use the VERSION as arg of the bundle target (e.g make bundle VERSION=0.0.2)
# - use environment variables to overwrite this value (e.g export VERSION=0.0.2)
VERSION ?= 0.0.1

# CHANNELS define the bundle channels used in the bundle. 
# Add a new line here if you would like to change its default config. (E.g CHANNELS = "preview,fast,stable")
# To re-generate a bundle for other specific channels without changing the standard setup, you can:
# - use the CHANNELS as arg of the bundle target (e.g make bundle CHANNELS=preview,fast,stable)
# - use environment variables to overwrite this value (e.g export CHANNELS="preview,fast,stable")
ifneq ($(origin CHANNELS), undefined)
BUNDLE_CHANNELS := --channels=$(CHANNELS)
endif

# DEFAULT_CHANNEL defines the default channel used in the bundle. 
# Add a new line here if you would like to change its default config. (E.g DEFAULT_CHANNEL = "stable")
# To re-generate a bundle for any other default channel without changing the default setup, you can:
# - use the DEFAULT_CHANNEL as arg of the bundle target (e.g make bundle DEFAULT_CHANNEL=stable)
# - use environment variables to overwrite this value (e.g export DEFAULT_CHANNEL="stable")
ifneq ($(origin DEFAULT_CHANNEL), undefined)
BUNDLE_DEFAULT_CHANNEL := --default-channel=$(DEFAULT_CHANNEL)
endif
BUNDLE_METADATA_OPTS ?= $(BUNDLE_CHANNELS) $(BUNDLE_DEFAULT_CHANNEL)

# BUNDLE_IMG defines the image:tag used for the bundle. 
# You can use it as an arg. (E.g make bundle-build BUNDLE_IMG=<some-registry>/<project-name-bundle>:<tag>)
BUNDLE_IMG ?= controller-bundle:$(VERSION)

CLUSTER ?= klab

# Image URL to use all building/pushing image targets
IMG ?= archeobot:latest
REGISTRY ?= image-registry.apps.$(CLUSTER).devops.gov.bc.ca
NAMESPACE ?= devops-archeobot

all: docker-build

# Run against the configured Kubernetes cluster in ~/.kube/config
run: ansible-operator
	$(ANSIBLE_OPERATOR) run

# # Install CRDs into a cluster
# install: kustomize
# 	$(KUSTOMIZE) build config/crd | kubectl apply -f -
#
# # Uninstall CRDs from a cluster
# uninstall: kustomize
# 	$(KUSTOMIZE) build config/crd | kubectl delete -f -

# Deploy cluster stuff
cluster-deploy: kustomize
	cd config/manager && $(KUSTOMIZE) edit set image controller=${REGISTRY}/${NAMESPACE}/${IMG}
	$(KUSTOMIZE) build config/cluster | kubectl apply -f -

# Undeploy cluster stuff
cluster-undeploy: kustomize
	$(KUSTOMIZE) build config/cluster | kubectl delete -f -

# Deploy controller only
deploy: kustomize
	cd config/manager && $(KUSTOMIZE) edit set image controller=${REGISTRY}/${NAMESPACE}/${IMG}
	$(KUSTOMIZE) build config/deploy/$(CLUSTER) | kubectl apply -f -

# Undeploy controller only
undeploy: kustomize
	$(KUSTOMIZE) build config/deploy/$(CLUSTER) | kubectl delete -f -

# Build the docker image
docker-build:
	docker login ${REGISTRY} -u $(shell oc whoami) -p $(shell oc whoami -t)
	docker build --platform=linux/amd64 -t ${REGISTRY}/${NAMESPACE}/${IMG} .

# Push the docker image
docker-push:
	docker login ${REGISTRY} -u $(shell oc whoami) -p $(shell oc whoami -t)
	docker push ${REGISTRY}/${NAMESPACE}/${IMG}

OS := $(shell uname -s | tr '[:upper:]' '[:lower:]')
ARCH := $(shell uname -m | sed 's/x86_64/amd64/')

# Download kustomize locally if necessary, preferring the $(pwd)/bin path over global if both exist.
.PHONY: kustomize
KUSTOMIZE = $(shell pwd)/bin/kustomize
kustomize:
ifeq (,$(wildcard $(KUSTOMIZE)))
ifeq (,$(shell which kustomize 2>/dev/null))
	@{ \
	set -e ;\
	mkdir -p $(dir $(KUSTOMIZE)) ;\
	curl -sSLo - https://github.com/kubernetes-sigs/kustomize/releases/download/kustomize/v3.5.4/kustomize_v3.5.4_$(OS)_$(ARCH).tar.gz | \
	tar xzf - -C bin/ ;\
	}
else
KUSTOMIZE = $(shell which kustomize)
endif
endif

# Download ansible-operator locally if necessary, preferring the $(pwd)/bin path over global if both exist.
.PHONY: ansible-operator
ANSIBLE_OPERATOR = $(shell pwd)/bin/ansible-operator
ansible-operator:
ifeq (,$(wildcard $(ANSIBLE_OPERATOR)))
ifeq (,$(shell which ansible-operator 2>/dev/null))
	@{ \
	set -e ;\
	mkdir -p $(dir $(ANSIBLE_OPERATOR)) ;\
	curl -sSLo $(ANSIBLE_OPERATOR) https://github.com/operator-framework/operator-sdk/releases/download/v1.5.0/ansible-operator_$(OS)_$(ARCH) ;\
	chmod +x $(ANSIBLE_OPERATOR) ;\
	}
else
ANSIBLE_OPERATOR = $(shell which ansible-operator)
endif
endif

.PHONY: bundle ## Generate bundle manifests and metadata, then validate generated files.
bundle: kustomize
	operator-sdk generate kustomize manifests -q
	cd config/manager && $(KUSTOMIZE) edit set image controller=$(IMG)
	$(KUSTOMIZE) build config/manifests | operator-sdk generate bundle -q --overwrite --version $(VERSION) $(BUNDLE_METADATA_OPTS)
	operator-sdk bundle validate ./bundle

.PHONY: bundle-build ## Build the bundle image.
bundle-build:
	docker build --platform=linux/arm64 -f bundle.Dockerfile -t $(BUNDLE_IMG) .
