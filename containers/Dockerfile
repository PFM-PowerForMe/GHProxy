FROM docker.io/library/alpine:edge AS git
ARG TAG
ENV TAG=${TAG:-main}
RUN apk update && apk add --no-cache \
	git
WORKDIR /
RUN git -c advice.detachedHead=false clone --branch $TAG --depth=1 --recurse-submodules https://github.com/pfm-powerforme/ghproxy.git source
WORKDIR /source/
RUN git -c advice.detachedHead=false clone --branch main --depth=1 --recurse-submodules https://github.com/pfm-powerforme/ghproxy-frontend.git pages
RUN rm -rf pages/.git/


FROM docker.io/library/golang:alpine AS builder
ARG TAG
ENV TAG=${TAG:-main}
ENV GO111MODULE=on \
    CGO_ENABLED=0 \
    GOOS=linux
WORKDIR /
COPY --from=git /source/  /build/
WORKDIR /build/
RUN go build -ldflags "-s -w -X main.version=$TAG" -o ghproxy


FROM docker.io/library/alpine:latest AS runtime
COPY --from=git /source/config/config.toml /data/ghproxy/config.toml
COPY --from=git /source/config/blacklist.json /data/ghproxy/blacklist.json
COPY --from=git /source/config/whitelist.json /data/ghproxy/whitelist.json
COPY --from=builder /build/ghproxy /data/ghproxy/ghproxy
RUN mkdir -p /data/www && mkdir -p /data/ghproxy/log
RUN chmod +x /data/ghproxy/ghproxy
CMD ["/data/ghproxy/ghproxy"]
