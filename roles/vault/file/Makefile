SHELL := /bin/bash
CLUSTER_KEYS := dist/cluster-keys.json

.PHONY: build
build:
	@-docker-compose up -d --build
	@sleep 30

.PHONY: destroy
destroy:
	@rm -rf dist
	@-docker-compose down --rmi all

.PHONY: shell
shell:
	@-docker-compose exec vault bash

.PHONY: init
init:
	@mkdir -p dist
	@test -f ${CLUSTER_KEYS} || docker exec vault vault operator init -key-shares=1 -key-threshold=1 -format=json > ${CLUSTER_KEYS}
	@sleep 10
	
.PHONY: unseal
unseal:
	@-docker exec vault vault operator unseal $(shell cat ${CLUSTER_KEYS} | jq -r ".unseal_keys_b64[]")
	@sleep 10

.PHONY: enable
enable:
	@-docker exec vault vault login $(shell cat ${CLUSTER_KEYS} | jq -r ".root_token")
	@echo "Enabling github auth"
	@-docker exec vault vault auth enable github
	@-docker exec vault vault write auth/github/config organization=ematta
	@-docker exec vault vault write auth/github/map/teams/dev value=dev-policy
	@echo "Writing policies"
	@-docker exec vault vault policy write default /tmp/default.policy.hcl
	@echo "Creating KV store"
	@-docker exec vault vault secrets enable kv

.PHONY: root-key
root-key:
	@cat ${CLUSTER_KEYS} | jq -r ".root_token"

.PHONY: provision
provision: init unseal enable