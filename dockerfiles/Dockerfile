# Builds watchtower from the checked-out source. Cross-compiles on the build
# platform, so multi-arch builds need no emulation.
FROM --platform=$BUILDPLATFORM golang:1.27.1-alpine3.24 AS builder

RUN apk add --no-cache ca-certificates tzdata

WORKDIR /src
COPY go.mod go.sum ./
RUN go mod download
COPY . .

ARG TARGETOS
ARG TARGETARCH
ARG VERSION=dev
RUN CGO_ENABLED=0 GOOS=$TARGETOS GOARCH=$TARGETARCH \
    go build -trimpath \
    -ldflags "-s -w -X github.com/containrrr/watchtower/internal/meta.Version=${VERSION}" \
    -o /watchtower .

FROM scratch
LABEL "com.centurylinklabs.watchtower"="true"
LABEL org.opencontainers.image.source="https://github.com/MHutatah/watchtower"

COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/ca-certificates.crt
COPY --from=builder /usr/share/zoneinfo /usr/share/zoneinfo
COPY --from=builder /watchtower /watchtower

EXPOSE 8080

HEALTHCHECK CMD [ "/watchtower", "--health-check"]

ENTRYPOINT ["/watchtower"]
