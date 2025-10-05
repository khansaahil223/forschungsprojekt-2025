.PHONY: create-cluster delete-cluster install-clabernetes setup deploy-topology clean status check-nodes help clabvert remove-topology recreate

# Configuration variables (can be overridden)
CLUSTER_NAME ?= kind
NAMESPACE ?= c9s-vlan
TOPOLOGY_FILE ?= clabverter2.yaml
clabverter = sudo docker run --user $$(id -u) -v $$(pwd):/clabernetes/work --rm ghcr.io/srl-labs/clabernetes/clabverter
helm = docker run --network host -ti --rm -v $$(pwd):/apps -w /apps -v ~/.kube:/root/.kube -v ~/.helm:/root/.helm -v ~/.config/helm:/root/.config/helm -v ~/.cache/helm:/root/.cache/helm  alpine/helm:3.12.3

# Default target
help:
	@echo "Available targets:"
	@echo "  setup           - Create cluster and install Clabernetes"
	@echo "  clabvert-and-deploy - Convert containerlab topology and deploy it"
	@echo "  status          - Show cluster and pod status"
	@echo "  clean           - Delete cluster"
	@echo "  recreate        - Delete and recreate everything"
	@echo "  clabvert        - Convert containerlab topology to Kubernetes"
	@echo "  deploy-topology - Deploy the network topology"
	@echo "  create-cluster  - Create Kind cluster only"
	@echo "  install-clabernetes - Install Clabernetes only"
	@echo "  remove-topology - Remove the deployed topology"
	@echo "  check-nodes     - Verify pod placement on correct nodes"
	@echo "  help            - Show this help message"

setup: create-cluster install-clabernetes
	@echo "✅ Cluster setup complete!"
	@echo "💡 Run 'make clabvert-and-deploy' to deploy your network topology"

create-cluster:
	@echo "🚀 Creating Kind cluster..."
	@if kind get clusters | grep -q "^$(CLUSTER_NAME)$$"; then \
		echo "⚠️  Cluster $(CLUSTER_NAME) already exists. Use 'make recreate' to recreate it."; \
		exit 1; \
	fi
	@if [ -z "$(KIND_CONFIG)" ]; then \
		echo "Error: KIND_CONFIG is not set!"; \
		echo "Available configurations:"; \
		ls kind-config; \
		exit 1; \
	fi
	kind create cluster --name $(CLUSTER_NAME) --config kind-config/$(KIND_CONFIG).yaml 
	@echo "⏳ Waiting for cluster to be ready..."
	kubectl wait --for=condition=Ready nodes --all --timeout=300s
	@echo "✅ Cluster created successfully"

delete-cluster:
	@echo "🗑️  Deleting Kind cluster..."
	kind delete cluster --name $(CLUSTER_NAME) || true

install-clabernetes:
	@echo "📦 Installing Clabernetes..."
	$(helm) upgrade --install --create-namespace --namespace c9s \
		clabernetes oci://ghcr.io/srl-labs/clabernetes/clabernetes
	@echo "⏳ Waiting for Clabernetes to be ready..."
	kubectl wait --for=condition=available --timeout=300s \
		deployment/clabernetes-manager -n c9s
	@echo "✅ Clabernetes installed successfully"

clabvert:
	@echo "Clabverting containerlab to Kubernetes..."
	@if [ -z "$(TOPO)" ]; then \
		echo "Error: Please specify TOPO. Example: make convert-lab TOPO=2router"; \
		echo "Available topologies:"; \
		ls topologies; \
		exit 1; \
	fi
	@if [ ! -d "topologies/$(TOPO)" ]; then \
		echo "Error: Directory 'topologies/$(TOPO)' not found!"; \
		exit 1; \
	fi
	cd topologies/$(TOPO) && $(clabverter) --stdout --naming non-prefixed > clabverter.yaml
	@echo "Lab converted to clabverter.yaml"
	cd ..
	@echo "💡 Run 'make deploy-topology' to deploy the converted lab"

deploy-topology:
	@echo "🌐 Deploying topology..."
	@if [ -z "$(TOPO)" ]; then \
		echo "Error: Please specify TOPO. Example: make convert-lab TOPO=2router"; \
		echo "Available topologies:"; \
		ls topologies; \
		exit 1; \
	fi
	@if [ ! -f topologies/$(TOPO)/clabverter.yaml ]; then \
		echo "❌ Topology file topologies/$(TOPO)/clabverter.yaml not found!"; \
		exit 1; \
	fi
	kubectl apply -f topologies/$(TOPO)/clabverter.yaml
	@echo "✅ Topology deployed"
	@echo "💡 Run 'make status' to check deployment status"

clabvert-and-deploy: clabvert deploy-topology

recreate-and-deploy:
	@if [ -z "$(KIND_CONFIG)" ]; then \
		echo "Error: KIND_CONFIG is not set!"; \
		echo "Available configurations:"; \
		ls kind-config; \
		exit 1; \
	fi
	@if [ -z "$(TOPO)" ]; then \
		echo "Error: TOPO is not set! Please specify TOPO. Example: make setup-and-deploy TOPO=2router KIND_CONFIG=yourconfig"; \
		echo "Available topologies:"; \
		ls topologies; \
		exit 1; \
	fi
	$(MAKE) recreate
	$(MAKE) clabvert-and-deploy

remove-topology:
	@echo "🗑️  Removing topology..."
	@if [ -z "$(TOPO)" ]; then \
		echo "Error: Please specify TOPO. Example: make convert-lab TOPO=2router"; \
		echo "Available topologies:"; \
		ls topologies; \
		exit 1; \
	fi
	@if [ ! -f topologies/$(TOPO)/clabverter.yaml ]; then \
		echo "❌ Topology file topologies/$(TOPO)/clabverter.yaml not found!"; \
		exit 1; \
	fi
	kubectl delete -f topologies/$(TOPO)/clabverter.yaml
	@echo "✅ Topology removed"
	@echo "💡 Run 'make status' to check deployment status"

status:
	@echo "📊 Cluster Status:"
	@echo "--- Nodes ---"
	kubectl get nodes -n ${NAMESPACE}
	@echo ""
	@echo "--- Pods ---"
	kubectl get pods -n $(NAMESPACE) -o wide

check-nodes:
	@echo "🔍 Verifying pod placement on correct nodes:"
	@echo ""
	@echo "Node Labels:"
	kubectl get nodes --show-labels | grep node-type || echo "No node-type labels found"
	@echo ""
	@echo "Pod Placement:"
	kubectl get pods -o custom-columns="POD:.metadata.name,NODE:.spec.nodeName,NODE_SELECTOR:.spec.nodeSelector" | grep -v "<none>" || echo "No pods with node selectors found"

recreate:
	@if [ -z "$(KIND_CONFIG)" ]; then \
		echo "Error: KIND_CONFIG is not set!"; \
		echo "Available configurations:"; \
		ls kind-config; \
		exit 1; \
	fi
	$(MAKE) clean
	$(MAKE) setup
	@echo "🔄 Cluster recreated successfully!"

clean: delete-cluster
	@echo "🧹 Cleanup complete"
