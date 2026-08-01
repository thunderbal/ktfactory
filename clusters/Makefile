-include .env
export

SHELL=/bin/bash

.PHONY: config secrets ctl-node wkr-node controlplane worker talosconfig
config: talosconfig controlplane worker
secrets: secrets.yaml
talosconfig: gen-talosconfig
controlplane: gen-controlplane.yaml
worker: gen-worker.yaml
ctl-node: ctl-$(HOST).yaml
wkr-node: wkr-$(HOST).yaml

wkr-%.yaml: gen-worker.yaml
	talosctl machineconfig patch $< \
		$(foreach f, $(wildcard patches/all.*.yml), --patch $(f)) \
		$(foreach f, $(wildcard patches/wkr.*.yml), --patch $(f)) \
		$(foreach f, $(wildcard patches/$*.*.yml), --patch $(f)) \
		--output $@
	# ===== $@: OK =====

ctl-%.yaml: gen-controlplane.yaml
	talosctl machineconfig patch $< \
		$(foreach f, $(wildcard patches/all.*.yml), --patch $(f)) \
		$(foreach f, $(wildcard patches/ctl.*.yml), --patch $(f)) \
		$(foreach f, $(wildcard patches/$*.*.yml), --patch $(f)) \
		--output $@
	# ===== $@: OK =====

gen-%.yaml: secrets.yaml
	@talosctl gen config --output-types $* \
		--with-secrets $< --output $@ --force \
		$${CLUSTER_NAME} https://$${K8S_ENDPOINT}:6443
	# ===== $@: OK =====

gen-talosconfig: secrets.yaml
	@talosctl gen config --output-types talosconfig \
		--with-secrets $< --output $@ --force \
		$${CLUSTER_NAME} https://$${K8S_ENDPOINT}:6443
	# ===== $@: OK =====

secrets.yaml: .env
	@[ -f "$@" ] || talosctl gen secrets -o $@
	@touch $@
	# ===== $@: OK =====

.PHONY: clean
clean:
	rm gen-* controlplane-*.yaml worker-*.yaml

.PHONY: new
new:
	@if [ -f ".env" ]; then \
		printf "'$@' target should not be executed in a cluster folder !\nRun it from parent folder.\n\n"; \
		exit 1; \
	fi
	@read -p "Enter cluster name ? " CLUSTER_NAME && [ -d "$${CLUSTER_NAME}" ] && echo "$${CLUSTER_NAME} already exists !" && exit || mkdir -p "$${CLUSTER_NAME}/patches"; \
	read -p "Enter domain ? " DOMAIN_NAME; \
	ln Makefile "$${CLUSTER_NAME}/Makefile"; \
	printf "CLUSTER_NAME=%s\nDOMAIN=%s\nK8S_ENDPOINT=%s.%s" \
		"$${CLUSTER_NAME}" "$${DOMAIN_NAME}" "$${CLUSTER_NAME}" "$${DOMAIN_NAME}" > "$${CLUSTER_NAME}/.env"; \
	$(MAKE) -C $${CLUSTER_NAME}	
