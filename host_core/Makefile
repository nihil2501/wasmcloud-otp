.PHONY: build build-image buildx-cross-image run run-interactive test release help deps
.DEFAULT: help

CARGO ?= cargo --color always
EXTRA_TEST_ARGS ?=
DOCKERFILESTATIC ?= host_core/Dockerfile.static
TAG ?= latest
STATIC_NAME ?= wasmcloud/host_core_static

help:  ## Display this help
	@awk 'BEGIN {FS = ":.*##"; printf "\nUsage:\n  make \033[36m<target>\033[0m\n"} /^[a-zA-Z_\-.*]+:.*?##/ { printf "  \033[36m%-15s\033[0m %s\n", $$1, $$2 } /^##@/ { printf "\n\033[1m%s\033[0m\n", substr($$0, 5) } ' $(MAKEFILE_LIST)

deps: ## Fetch mix dependencies
	mix deps.get

build: deps ## Compile host_core
	mix compile

build-image: ## Compile host_core docker image
	docker build \
		--build-arg BUILDER_IMAGE=elixir:1.14.3-alpine \
		--build-arg RELEASE_IMAGE=alpine:3.17.2 \
		--build-arg MIX_ENV=prod \
		-t host_core:alpine \
		.

build-image-static: ## build statically compiled docker image
	cd ../ \
	&& docker build -f Dockerfile.localstatic -t localstatic . \
	&& docker build -f $(DOCKERFILESTATIC) . -t $(STATIC_NAME):$(TAG)

# erlang 24 until 25 stops segfaulting under qemu
buildx-cross-image: ## Compile host_core docker image using buildx for amd64 and arm64
	docker buildx build  \
		--build-arg BUILDER_IMAGE=elixir:1.14.3-otp-24-alpine \
		--build-arg RELEASE_IMAGE=alpine:3.17.2 \
		--build-arg MIX_ENV=prod \
		-t host_core:alpine \
		--platform linux/amd64,linux/arm64 \
		.

run: build ## Run host_core
	mix

run-interactive: build ## Run host_core with an iex session
	iex -S mix

credo: build
	mix credo

test: build ## Run test suite, launch NATS with docker-compose
	docker compose -f ./test/docker-compose.yml up --detach
	MIX_ENV=test mix test $(EXTRA_TEST_ARGS)
	docker compose -f ./test/docker-compose.yml down

release: deps ## Creates a mix release for host_core
	MIX_ENV=prod mix release
