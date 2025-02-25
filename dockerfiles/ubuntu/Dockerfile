ARG base_image
ARG builder_image=concourse/golang-builder

FROM ${builder_image} AS builder
WORKDIR /src
COPY . .
RUN go mod download
RUN go build -o /assets/out ./cmd/out
RUN go build -o /assets/in ./cmd/in
RUN go build -o /assets/check ./cmd/check
RUN set -e; for pkg in $(go list ./...); do \
		go test -o "/tests/$(basename $pkg).test" -c $pkg; \
	done

FROM ${base_image} AS resource
RUN apt update && apt upgrade -y -o Dpkg::Options::="--force-confdef"
RUN apt update && apt install -y --no-install-recommends \
    ca-certificates \
  && rm -rf /var/lib/apt/lists/*
COPY --from=builder /assets /opt/resource

FROM resource AS tests
COPY --from=builder /tests /tests
RUN set -e; for test in /tests/*.test; do \
		$test; \
	done

FROM resource
