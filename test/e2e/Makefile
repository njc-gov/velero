# Copyright 2020 The Kubernetes Authors.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# If you update this file, please follow:
# https://suva.sh/posts/well-documented-makefiles/

# Use GOPROXY environment variable if set

.DEFAULT_GOAL:=help

ARCH ?= $(shell go env GOOS)-$(shell go env GOARCH)
platform_temp = $(subst -, ,$(ARCH))
GOOS = $(word 1, $(platform_temp))
GOARCH = $(word 2, $(platform_temp))

GOPROXY := $(shell go env GOPROXY)
ifeq ($(GOPROXY),)
GOPROXY := https://proxy.golang.org
endif
export GOPROXY

REPO_ROOT := $(shell git rev-parse --show-toplevel)

help:  ## Display this help
	@awk 'BEGIN {FS = ":.*##"; printf "\nUsage:\n  make \033[36m<target>\033[0m\n"} /^[a-zA-Z0-9_-]+:.*?##/ { printf "  \033[36m%-25s\033[0m %s\n", $$1, $$2 } /^##@/ { printf "\n\033[1m%s\033[0m\n", substr($$0, 5) } ' $(MAKEFILE_LIST)

## --------------------------------------
## Binaries
## --------------------------------------

TOOLS_DIR := $(REPO_ROOT)/hack/tools
BIN_DIR := bin
TOOLS_BIN_DIR := $(TOOLS_DIR)/$(BIN_DIR)
GINKGO := $(GOPATH)/bin/ginkgo
KUSTOMIZE := $(TOOLS_BIN_DIR)/kustomize
OUTPUT_DIR := _output/$(GOOS)/$(GOARCH)/bin
GINKGO_FOCUS ?=
VELERO_CLI ?=$$(pwd)/../../_output/bin/$(GOOS)/$(GOARCH)/velero
VELERO_IMAGE ?= velero/velero:main
CREDS_FILE ?=
BSL_BUCKET ?=
BSL_PREFIX ?=
BSL_CONFIG ?=
VSL_CONFIG ?=
PLUGIN_PROVIDER ?=

.PHONY:ginkgo
ginkgo: # Make sure ginkgo is in $GOPATH/bin
	go get github.com/onsi/ginkgo/ginkgo

.PHONY: run
run: ginkgo
		@[ "${CREDS_FILE}" ] && echo "Using credentials from ${CREDS_FILE}" || \
			( echo "A credentials file is required to run E2E tests, please re-run the make target with CREDS_FILE=<PathToCredentialsFile>"; exit 1 )
		@[ "${BSL_BUCKET}" ] && echo "Using bucket ${BSL_BUCKET} to store backups from E2E tests" || \
			(echo "Bucket to store the backups from E2E tests is required, please re-run with BSL_BUCKET=<BucketName>"; exit 1 )
		@[ "${PLUGIN_PROVIDER}" ] && echo "Using plugin provider ${PLUGIN_PROVIDER} for object storage and volume snaphotting" || \
			(echo "Plugin provider  to store the backups from E2E tests is required, please re-run with PLUGIN_PROVIDER=<pluginProviderName>"; exit 1 )
	@$(GINKGO) -v -focus="$(GINKGO_FOCUS)" . -- -velerocli=$(VELERO_CLI) \
		-velero-image=$(VELERO_IMAGE) \
		-credentials-file=$(CREDS_FILE) \
		-bucket=$(BSL_BUCKET) \
		-prefix=$(BSL_PREFIX) \
		-bsl-config=$(BSL_CONFIG) \
		-vsl-config=$(VSL_CONFIG) \
		-plugin-provider=$(PLUGIN_PROVIDER)

build: ginkgo
	mkdir -p $(OUTPUT_DIR)
	$(GINKGO) build . 
