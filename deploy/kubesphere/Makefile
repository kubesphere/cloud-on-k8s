NAMESPACE ?= kubesphere-logging-system
NAME_PREFFIX ?= kubesphere-eck

# install
install:
	@echo "======install start======"
	helm upgrade --install elastic-operator . -n $(NAMESPACE)  -f values-for-kubesphere.yaml
	kubectl apply -f ./es-cluster-with-auth.yaml
	@echo "======install done======"

# patch
patch: createSecrets  pullConfig handleConfig  output  apply 


ES_USERNAME ?= elastic
ES_PASSWORD ?= $(shell kubectl -n $(NAMESPACE) get secrets $(NAME_PREFFIX)-es-elastic-user -o go-template="{{.data.elastic | base64decode}}")
ENDPOINT := $(NAME_PREFFIX)-es-http.$(NAMESPACE).svc
OUTPUT ?={"spec":{"es":{"host":"$(ENDPOINT)","httpPassword":{"valueFrom":{"secretKeyRef":{"key":"password","name":"elasticsearch-credentials"}}},"httpUser":{"valueFrom":{"secretKeyRef":{"key":"username","name":"elasticsearch-credentials"}}}}}}

createSecrets:
	@echo "\n======start to create secrets elasticsearch-credentials====="
	-kubectl -n $(NAMESPACE) delete secret generic elasticsearch-credentials 
	kubectl -n $(NAMESPACE) create secret generic elasticsearch-credentials --from-literal="username=$(ES_USERNAME)" --from-literal="password=$(ES_PASSWORD)" --type=kubernetes.io/basic-auth
	
pullConfig:
	@echo "\n======start to pull kubesphere config====="
	kubectl -n kubesphere-system get cm kubesphere-config -o yaml > /tmp/kubesphere-config.yaml
	-kubectl -n kubesphere-logging-system get cm elasticsearch-logging-curator-elasticsearch-curator-config -o yaml > /tmp/curator.yaml 

PATCH_HOST := 's/elasticsearch-logging-data.kubesphere-logging-system.svc/$(ENDPOINT)/g'
PATCH_CONTENT:='/^(\s+logging:)|(\s+events:)|(\s+auditing:)/a\      basicAuth: true\n      username: $(ES_USERNAME)\n      password: $(ES_PASSWORD)'

handleConfig:
	@echo "\n======start to patch configmap kubesphere-config for logging======"
	sed -i -e $(PATCH_HOST)  /tmp/kubesphere-config.yaml
	-sed -i -e $(PATCH_HOST)  /tmp/curator.yaml
	sed -i -E $(PATCH_CONTENT) /tmp/kubesphere-config.yaml	

output: 
	@echo "\n======start to patch fluentbit outputs======="
	-kubectl patch output/es -n $(NAMESPACE) --type merge -p '$(OUTPUT)'
	-kubectl patch output/es-events -n $(NAMESPACE) --type merge -p '$(OUTPUT)'
	-kubectl patch output/es-auditing -n $(NAMESPACE) --type merge -p '$(OUTPUT)'

apply:
	@echo "\n======start to apply configs======="
	kubectl apply -f /tmp/kubesphere-config.yaml
	-kubectl apply -f /tmp/curator.yaml
	kubectl -n kubesphere-system rollout restart deployment ks-apiserver

access: access-curl

access-curl:
	 @echo "Step 1, run the following cmdline in anthor terminal:\n\tkubectl -n $(NAMESPACE) port-forward service/$(NAME_PREFFIX)-es-http 9200"
	 @echo "Step 2, run the following cmdline in the current terminal:\n\tcurl -u elastic:$(ES_PASSWORD) localhost:9200/_cluster/health?pretty"

access-crd:
	kubectl get elasticsearches.elasticsearch.k8s.elastic.co -A

uninstall:
	@echo "start to uninstall"
	helm uninstall elastic-operator -n $(NAMESPACE)
	
