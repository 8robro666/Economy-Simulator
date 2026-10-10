FROM ubuntu:22.04 AS builder

RUN apt-get update && apt-get install -y \
    build-essential \
    cmake \
    libsdl2-dev \
    libosmesa6-dev \
    libpng-dev \
    libx11-dev \
    libcrypto++-dev \
    libssl-dev \
    libopenal-dev \
    libgl1-mesa-dev \
    libcrypt-dev \
    liblz4-dev \
    libcurl4-gnutls-dev \
    wget \
    xvfb \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /usr/src/app

COPY services/RCCService/ ./RCCService

WORKDIR /usr/src/app/RCCService

RUN chmod +x ./rcc 2>/dev/null || echo "Linux RCC binary (./rcc) not found - you must provide a Linux build of RCCService"

FROM ubuntu:22.04 AS runner

RUN apt-get update && apt-get install -y \
    libsdl2-2.0-0 \
    libosmesa6 \
    libpng16-16 \
    libx11-6 \
    libcrypto++6 \
    libssl3 \
    libopenal1 \
    libgl1 \
    libcrypt1 \
    liblz4-1 \
    libcurl4 \
    socat \
    wget \
    xvfb \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /usr/src/app

COPY --from=builder /usr/src/app/RCCService ./RCCService
COPY services/game-server/docker/start.sh ./start.sh

RUN chmod +x ./start.sh
RUN chmod +x ./RCCService/rcc 2>/dev/null || true

ENTRYPOINT ["/usr/src/app/start.sh"]