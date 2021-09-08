FROM ubuntu:18.04

# XTB and CREST version
ARG XTB_VER=6.4.1
ARG CREST_VER=2.11.1

# Timezone settings
ARG DEBIAN_FRONTEND=noninteractive
RUN apt update && apt install \
    -y tzdata \
    && apt clean \
    && rm -rf /var/lib/apt/lists/*
ENV TZ=Asia/Tokyo

# apt install for Python
RUN apt update && apt install -y --no-install-recommends \
    ca-certificates \
    wget \
    xz-utils \
    unzip \
    && apt clean \
    && rm -rf /var/lib/apt/lists/*

# install xtb and crest
RUN cd /opt \
    && wget https://github.com/grimme-lab/xtb/releases/download/v${XTB_VER}/xtb-${XTB_VER}-linux-x86_64.tar.xz \
    && tar -xf xtb-${XTB_VER}-linux-x86_64.tar.xz \
    && rm xtb-${XTB_VER}-linux-x86_64.tar.xz \ 
    && wget https://github.com/grimme-lab/crest/releases/download/v${CREST_VER}/crest-binary-${CREST_VER}.zip \
    && unzip crest-binary-${CREST_VER}.zip \
    && mv crest xtb-${XTB_VER}/bin \
    && rm crest-binary-${CREST_VER}.zip 

# set env and path for xtb
ENV XTBHOME=/opt/xtb-${XTB_VER}
ENV XTBPATH=${XTBHOME}/share/xtb:${XTBHOME} MANPATH=${MANPATH}:${XTBHOME}/share/man \
    PATH=${PATH}:${XTBHOME}/bin \
    LD_LIBRARY_PATH=${LD_LIBRARY_PATH}:${XTBHOME}/lib64 \
    PYTHONPATH=${PYTHONPATH}:${XTBHOME}/python

COPY crestrun ${XTBHOME}/bin
RUN chmod +x ${XTBHOME}/bin/crestrun

WORKDIR /work
CMD /bin/bash
