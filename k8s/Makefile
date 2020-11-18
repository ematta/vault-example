SHELL := /bin/bash
SLEEP := 60
CLUSTER_KEYS := dist/cluster-keys.json
CONSUL_VALUES := config/helm/values/consul.yml
VAULT_VALUES := config/helm/values/vault.yml
VAULT_WEBAPP := config/k8s/webapp.yml

.PHONY: install
install:
	helm install vault hashicorp/vault --values ${VAULT_VALUES}
	helm install consul hashicorp/consul --values ${CONSUL_VALUES}

.PHONY: uninstall
uninstall:
	helm uninstall consul
	helm uninstall vault

.PHONY: recreate
recreate:
	@-minikube stop && minikube delete && minikube start
	@echo "Waiting ${SLEEP} seconds for things to start"
	@sleep ${SLEEP}
	$(MAKE) install

.PHONY: shell
vault-shell:
	kubectl exec -it vault-0 -- /bin/sh

.PHONY: status
status:
	kubectl get pods

.PHONY: init
init:
	@mkdir -p dist
	kubectl exec vault-0 -- vault operator init -key-shares=1 -key-threshold=1 -format=json > ${CLUSTER_KEYS}

.PHONY: keys
keys:
	@echo "VAULT_ROOT_KEY: $(shell cat ${CLUSTER_KEYS} | jq -r ".root_token")"

.PHONY: unseal
unseal:
	kubectl exec vault-1 -- vault operator unseal $(shell cat ${CLUSTER_KEYS} | jq -r ".unseal_keys_b64[]")
	kubectl exec vault-2 -- vault operator unseal $(shell cat ${CLUSTER_KEYS} | jq -r ".unseal_keys_b64[]")
	kubectl exec vault-0 -- vault operator unseal $(shell cat ${CLUSTER_KEYS} | jq -r ".unseal_keys_b64[]")
