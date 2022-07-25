# Declare Vars
.EXPORT_ALL_VARIABLES:
CLUSTER_NAME ?= admission-controller
NAMESPACE ?= opa
CONF_PATH ?= config
CERTS_PATH ?= certs
POLICY_PATH ?= policies
OBJECTS_PATH ?= objects

# Makefile functions
download-opa:
	@echo "[INFO] Download OPA"
	@curl -L -o opa https://openpolicyagent.org/downloads/v0.40.0/opa_linux_amd64_static
	@chmod 755 ./opa
	@mv opa $(POLICY_PATH)/opa
	@echo ""

create-cluster:
	@echo "[INFO] Configure cluster on Minikube"
	@minikube start -p $(CLUSTER_NAME) --memory 3072 --cpus 2
	@echo ""

	@echo "[INFO] Enable ingress on Minikube"
	@minikube -p $(CLUSTER_NAME) addons enable ingress
	@echo ""

create-namespace-opa:
	@echo "[INFO] Create namespace $(NAMESPACE)"
	@kubectl create namespace $(NAMESPACE) --dry-run=client -o yaml | kubectl apply -f -
	@echo ""

	@echo "[INFO] Configure context with namespace $(NAMESPACE)"
	@kubectl config set-context $(CLUSTER_NAME) --namespace $(NAMESPACE)
	@echo ""

create-certs:
	@echo "[INFO] Create CA"
	@mkdir -p $(CERTS_PATH)
	@openssl genrsa -out $(CERTS_PATH)/ca.key 2048
	@openssl req -x509 -new -nodes -key $(CERTS_PATH)/ca.key \
			-days 100000 									 \
			-out $(CERTS_PATH)/ca.crt 						 \
			-subj "/CN=admission_ca"
	@echo ""

	@echo "[INFO] Create Certs"
	@openssl genrsa -out $(CERTS_PATH)/server.key 2048
	@openssl req -new -key $(CERTS_PATH)/server.key 		 \
			-out $(CERTS_PATH)/server.csr           		 \
			-config $(CONF_PATH)/server.conf
	@openssl x509 -req -in $(CERTS_PATH)/server.csr 		 \
			-CA $(CERTS_PATH)/ca.crt                		 \
			-CAkey $(CERTS_PATH)/ca.key             		 \
			-CAcreateserial                         		 \
			-out $(CERTS_PATH)/server.crt           		 \
			-days 100000                            		 \
			-extensions v3_req                      		 \
			-extfile $(CONF_PATH)/server.conf
	@echo ""

	@echo "[INFO] Create Kubernetes secret with certs"
	@kubectl create secret tls opa-server 	                 \
			--cert=$(CERTS_PATH)/server.crt               	 \
			--key=$(CERTS_PATH)/server.key
	@echo ""

create-opa-policy:
	@echo "[INFO] Show ingress-allowlist.rego to restrict Hostnames"
	@cat $(POLICY_PATH)/ingress-allowlist.rego
	@echo ""

	@echo "[INFO] Show ingress-conflicts.rego to prohibit hostname conflicts"
	@cat $(POLICY_PATH)/ingress-conflicts.rego
	@echo ""

	@echo "[INFO] Show main.rego to combine policies"
	@cat $(POLICY_PATH)/main.rego
	@echo ""

build-opa-bundle:
	@echo "[INFO] Show .manifest to build OPA Bundle"
	@cat $(POLICY_PATH)/.manifest
	@echo ""

	@./$(POLICY_PATH)/opa build -b $(POLICY_PATH)
	@mv bundle.tar.gz $(POLICY_PATH)
	@docker run --rm --name bundle-server -d -p 8888:80 -v $(pwd)/$(CERTS_PATH):/usr/share/nginx/html:ro nginx:latest

deploy-opa:
	@echo "[INFO] Deploy admission controller"
	@cat $(CONF_PATH)/admission-controller.yaml
	@echo ""
	@kubectl apply -f $(CONF_PATH)/admission-controller.yaml
	@echo ""

	@echo "[INFO] Export CA.crt and replace in $(CONF_PATH)webhook-configuration file"
	@export CABUNDLE=$(cat $(CERTS_PATH)/ca.crt | base64| tr -d '\n')
	@sed -i -e 's/CABUNDLE/'"${CABUNDLE}"'/g' $(CONF_PATH)/webhook-configuration.yaml
	@echo ""

	@echo "[INFO] Labeling namespaces"
	@kubectl label ns kube-system openpolicyagent.org/webhook=ignore
	@kubectl label ns $(NAMESPACE) openpolicyagent.org/webhook=ignore
	@echo ""

	@echo "[INFO] Deploy webhook configuration"
	@cat $(CONF_PATH)/webhook-configuration.yaml
	@echo ""
	@kubectl apply -f $(CONF_PATH)/webhook-configuration.yaml
	@echo ""

show-logs:
	@echo "[INFO] Show logs"
#	@kubectl logs -l app=opa -c opa
	@echo ""

create-resources:
	@echo "[INFO] Create QA and Production namespaces"
	@kubectl apply -f $(OBJECTS_PATH)/qa-namespace.yaml 
	@kubectl apply -f $(OBJECTS_PATH)/production-namespace.yaml
	@echo ""

	@echo "[INFO] Create Ingress OK and Bad"
	@kubectl apply -f $(OBJECTS_PATH)/ingress-ok.yaml -n production
	@kubectl apply -f $(OBJECTS_PATH)/ingress-bad.yaml -n qa
	@echo ""

delete:
	@echo "[INFO] Delete environment"
	@minikube delete -p $(CLUSTER_NAME)
	@rm -rf $(CERTS_PATH)
	@rm $(POLICY_PATH)/opa
	@docker rm -f bundle-server

deploy: download-opa create-cluster create-namespace-opa create-certs create-opa-policy build-opa-bundle deploy-opa show-logs create-resources

.PHONY: download-opa create-cluster create-namespace-opa create-certs create-opa-policy build-opa-bundle deploy-opa show-logs create-resources delete
