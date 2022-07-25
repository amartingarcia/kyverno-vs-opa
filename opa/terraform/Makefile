# Declare Vars
#CLUSTER_NAME ?= cluster.k8s.co-live.internal
#PATH_KEY ?= ./key.pub

# Export Vars
.EXPORT_ALL_VARIABLES:
CLUSTER_NAME ?= admission-controller
NAMESPACE ?= opa
CONF_PATH ?= config
CERTS_PATH ?= certs
POLICY_PATH ?= policies
OBJECTS_PATH ?= objects

# Makefile functions
terraform-init:
	@terraform init
	@terraform plan --out tfplan.binary


deploy: download-opa create-cluster create-namespace-opa create-certs create-opa-policy build-opa-bundle deploy-opa show-logs create-resources

.PHONY: download-opa create-cluster create-namespace-opa create-certs create-opa-policy build-opa-bundle deploy-opa show-logs create-resources delete
