FROM ubuntu:24.04

# XTB and CREST version
ARG XTB_VER=6.7.1
ARG CREST_VER=3.0.1

# apt installs
RUN apt update && apt install -y --no-install-recommends \
    ca-certificates \
    wget \
    xz-utils \
    dos2unix \
    && apt clean \
    && rm -rf /var/lib/apt/lists/*

# set env and path for xtb
ENV XTBHOME=/opt/xtb-${XTB_VER}
ENV XTBPATH=${XTBHOME}/share/xtb:${XTBHOME} \
    MANPATH=${MANPATH}:${XTBHOME}/share/man \
    PATH=${PATH}:${XTBHOME}/bin \
    LD_LIBRARY_PATH=${LD_LIBRARY_PATH}:${XTBHOME}/lib

# install xtb and crest
RUN cd /opt \
    && wget https://github.com/grimme-lab/xtb/releases/download/v${XTB_VER}/xtb-${XTB_VER}-linux-x86_64.tar.xz \
    && tar -xf xtb-${XTB_VER}-linux-x86_64.tar.xz \
    && mkdir ${XTBHOME} \
    && mv xtb-dist/* ${XTBHOME} \
    && rm xtb-${XTB_VER}-linux-x86_64.tar.xz \ 
    && rm -r xtb-dist \
    && wget https://github.com/crest-lab/crest/releases/download/v${CREST_VER}/crest-latest.tar.xz \
    && tar -xf crest-latest.tar.xz \
    && mv crest ${XTBHOME}/bin \
    && rm crest-latest.tar.xz

COPY crestrun ${XTBHOME}/bin
RUN chmod +x ${XTBHOME}/bin/crestrun

WORKDIR /work
CMD /bin/bash
