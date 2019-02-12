TEST?=$$(go list ./... |grep -v 'vendor')
GOFMT_FILES?=$$(find . -name '*.go' |grep -v vendor)
WEBSITE_REPO=github.com/hashicorp/terraform-website
PKG_NAME=cloudfoundry

default: build

release:
	rm -fr bin
	mkdir -p bin
	CGO_ENABLED=0 GOARCH=amd64 GOOS=windows go build -o bin/terraform-provider-cf_windows_amd64.exe
	CGO_ENABLED=0 GOARCH=amd64 GOOS=linux go build -o bin/terraform-provider-cf_linux_amd64
	CGO_ENABLED=0 GOARCH=amd64 GOOS=darwin go build -o bin/terraform-provider-cf_darwin_amd64

build: check
	go install

test: check
	go test -i $(TEST) || exit 1
	echo $(TEST) | \
		xargs -t -n4 go test $(TESTARGS) -timeout=30s -parallel=4

testacc: check
	TF_ACC=1 go test $(TEST) -v $(TESTARGS) -timeout 180m

fmt:
	gofmt -w $(GOFMT_FILES)

check:
	@gometalinter.v2 --config .gometalinter.json --deadline 120s

vendor-status:
	@govendor status

test-compile:
	@if [ "$(TEST)" = "./..." ]; then \
		echo "ERROR: Set TEST to a specific package. For example,"; \
		echo "  make test-compile TEST=./aws"; \
		exit 1; \
	fi
	go test -c $(TEST) $(TESTARGS)

website:
ifeq (,$(wildcard $(GOPATH)/src/$(WEBSITE_REPO)))
	echo "$(WEBSITE_REPO) not found in your GOPATH (necessary for layouts and assets), get-ting..."
	git clone https://$(WEBSITE_REPO) $(GOPATH)/src/$(WEBSITE_REPO)
endif
	@$(MAKE) -C $(GOPATH)/src/$(WEBSITE_REPO) website-provider PROVIDER_PATH=$(shell pwd) PROVIDER_NAME=$(PKG_NAME)

website-test:
ifeq (,$(wildcard $(GOPATH)/src/$(WEBSITE_REPO)))
	echo "$(WEBSITE_REPO) not found in your GOPATH (necessary for layouts and assets), get-ting..."
	git clone https://$(WEBSITE_REPO) $(GOPATH)/src/$(WEBSITE_REPO)
endif
	@$(MAKE) -C $(GOPATH)/src/$(WEBSITE_REPO) website-provider-test PROVIDER_PATH=$(shell pwd) PROVIDER_NAME=$(PKG_NAME)


.PHONY: build test testacc fmt check vendor-status test-compile website website-test
