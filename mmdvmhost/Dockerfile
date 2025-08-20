# Build stage
FROM --platform=$BUILDPLATFORM alpine:latest AS builder

# Install build dependencies
RUN apk add --no-cache \
    git \
    gcc \
    g++ \
    make \
    cmake \
    linux-headers \
    musl-dev

# Clone and build MMDVMHost
WORKDIR /build
RUN git clone https://github.com/g4klx/MMDVMHost.git
WORKDIR /build/MMDVMHost
RUN make clean && make

# Runtime stage  
FROM alpine:latest

RUN apk add --no-cache libstdc++ libgcc
COPY --from=builder /build/MMDVMHost/MMDVMHost /usr/local/bin/
RUN addgroup -g 1001 mmdvm && adduser -D -u 1001 -G mmdvm mmdvm

USER mmdvm
CMD ["/usr/local/bin/MMDVMHost"]
