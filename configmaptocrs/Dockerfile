# syntax=docker/dockerfile:1.2

FROM --platform=$BUILDPLATFORM docker.io/golang:1.20.0 AS builder
ARG GIT_COMMIT=dev
ARG GIT_BRANCH=dev
WORKDIR $GOPATH/go.universe.tf/metallb

# Cache the downloads
COPY go.mod go.sum ./
RUN go mod download

# COPY internals
COPY internal internal
COPY api api

# COPY configmaptocrs
COPY configmaptocrs/*.go configmaptocrs/

ARG TARGETARCH
ARG TARGETOS
ARG TARGETPLATFORM

# have to manually convert as building the different arms can cause issues
# Extract variant
RUN case ${TARGETPLATFORM} in \
  "linux/arm/v6") export VARIANT="6" ;; \
  "linux/arm/v7") export VARIANT="7" ;; \
  *) export VARIANT="" ;; \
  esac

# Cache builds directory for faster rebuild
RUN --mount=type=cache,target=/root/.cache/go-build \
  --mount=type=cache,target=/go/pkg \
  CGO_ENABLED=0 GOOS=$TARGETOS GOARCH=$TARGETARCH GOARM=$VARIANT \
  go build -v -o /build/configmaptocrs \
  -ldflags "-X 'go.universe.tf/metallb/internal/version.gitCommit=${GIT_COMMIT}' -X 'go.universe.tf/metallb/internal/version.gitBranch=${GIT_BRANCH}'" \
  go.universe.tf/metallb/configmaptocrs

FROM docker.io/alpine:latest

COPY --from=builder /build/configmaptocrs /configmaptocrs
COPY LICENSE /

LABEL org.opencontainers.image.authors="metallb" \
  org.opencontainers.image.url="https://github.com/metallb/metallb" \
  org.opencontainers.image.documentation="https://metallb.universe.tf" \
  org.opencontainers.image.source="https://github.com/metallb/metallb" \
  org.opencontainers.image.vendor="metallb" \
  org.opencontainers.image.licenses="Apache-2.0" \
  org.opencontainers.image.description="Metallb Configmap to CRs converter" \
  org.opencontainers.image.title="configmap to crs" \
  org.opencontainers.image.base.name="docker.io/alpine:latest"

ENTRYPOINT ["/configmaptocrs"]
