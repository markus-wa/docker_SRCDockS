FROM registry.gitlab.steamos.cloud/steamrt/sniper/platform:latest-container-runtime-depot AS entrypoint

RUN apt-get update \
		&& apt-get install -y gcc g++

COPY ./srcds/main.cpp ./
RUN gcc -o main main.cpp


FROM registry.gitlab.steamos.cloud/steamrt/sniper/platform:latest-container-runtime-depot
ENV  DEBIAN_FRONTEND=noninteractive

ENV APP_NAME=csgo
ENV SRCDS_ARGS=""
ENV NO_BSP_CVAR=0
ENV SRCDS_RUN=0
ENV IP=0.0.0.0
ENV PORT=27015
ENV STOCK_SM_PLUGINS=""

VOLUME [ "/repo" ]

ARG UID=1000
ARG apt_install_packages
ARG run_before_apt

RUN $run_before_apt

RUN apt update \
		&& apt install -y tar curl jq gcc g++ iproute2 gdb telnet net-tools unzip expect netcat tzdata ca-certificates openssl locales \
		&& apt clean \
		&& rm -rf /var/lib/apt/lists/*

RUN mkdir /overlays

WORKDIR /srcds

COPY ./srcds/*.sh ./
COPY --from=entrypoint main ./

RUN useradd -m -u $UID srcds && mkdir srv && chown srcds srv && chmod +rx *.sh

USER srcds

# Not creating this folder will cause various error messages to be logged on the first start.
RUN mkdir -p ~/Steam ~/.steam/sdk32/

EXPOSE 27015/tcp 27015/udp

ENTRYPOINT [ "/srcds/main" ]

HEALTHCHECK --interval=15s --timeout=5s --start-period=10s --retries=2 CMD [ "/srcds/healthcheck.sh" ]
