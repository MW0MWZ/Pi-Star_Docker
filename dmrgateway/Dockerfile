# Build stage
FROM alpine:latest AS builder

# Install build dependencies
RUN apk add --no-cache \
    git \
    gcc \
    g++ \
    make \
    cmake \
    linux-headers \
    musl-dev \
    binutils

# Build arguments for cross-compilation
ARG TARGETPLATFORM
ARG BUILDPLATFORM
RUN echo "Building for $TARGETPLATFORM on $BUILDPLATFORM"

# Clone and build DMRGateway
WORKDIR /build
RUN git clone https://github.com/g4klx/DMRGateway.git
WORKDIR /build/DMRGateway
RUN make clean && make

# Strip the binary to reduce size
RUN strip DMRGateway

# Runtime stage  
FROM alpine:latest

# Install runtime dependencies only
RUN apk add --no-cache \
    libstdc++ \
    libgcc \
    tzdata

# Create Pi-Star directory structure
RUN mkdir -p /opt/pi-star/bin \
             /opt/pi-star/etc \
             /opt/pi-star/log \
             /opt/pi-star/DMRAudio

# Create non-root user
RUN addgroup -g 1001 pistar && \
    adduser -D -u 1001 -G pistar pistar

# Copy files from build stage to Pi-Star bin directory
COPY --from=builder /build/DMRGateway/DMRGateway /opt/pi-star/bin/
COPY --from=builder /build/DMRGateway/Audio/* /opt/pi-star/DMRAudio/

# Set ownership of Pi-Star directories
RUN chown -R pistar:pistar /opt/pi-star

# Add Pi-Star bin to PATH
ENV PATH="/opt/pi-star/bin:$PATH"

# Switch to non-root user
USER pistar

# Set working directory
WORKDIR /opt/pi-star

# Expose ports for inter-container communication
# Dynamic TG Control
EXPOSE 3769/udp
# Rremote Control
EXPOSE 7643/udp
# DMR network communication
EXPOSE 62031/udp

# Use the Pi-Star path structure
CMD ["/opt/pi-star/bin/DMRGateway", "/opt/pi-star/etc/DMRGateway.ini"]