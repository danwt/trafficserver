ARG IMAGE="ubuntu:20.10"
FROM $IMAGE

# Core parts needed to compile and run c++ within container, with tests, coverage, debugger. ----------------------------------------------------------

RUN set -eux; \
    echo "Installing essentials ..."; \
    apt-get update; \
    export DEBIAN_FRONTEND=noninteractive; \
    apt-get -y install --no-install-recommends wget curl xz-utils ca-certificates make ninja-build cmake ccache\
    ;


RUN set -eux; \
    echo "Installing Traffic Server deps ..."; \
    # https://docs.trafficserver.apache.org/en/latest/getting-started/index.en.html#ubuntu
    apt-get update; \
    export DEBIAN_FRONTEND=noninteractive; \
    apt-get -y install --no-install-recommends \
    pkg-config \
    libtool \
    openssl \
    libssl-dev \
    libpcre3 \
    libpcre3-dev \
    libcap-dev \
    flex \
    hwloc \
    liblua5.3-dev \
    zlib1g-dev \
    libncurses5-dev \
    git \
    autoconf \
    automake \
    ;

RUN set -eux; \
    echo "Installing Clang 10 ..."; \
    apt-get update; \
    export DEBIAN_FRONTEND=noninteractive; \
    apt-get -y install --no-install-recommends \
    clang-10 \
    clang-tidy-10 \
    clang-format-10 \
    llvm-10 \
    ; \
    update-alternatives --install /usr/bin/clang clang /usr/bin/clang-10 100; \
    update-alternatives --install /usr/bin/cc cc /usr/bin/clang-10 100; \
    update-alternatives --install /usr/bin/clang++ clang++ /usr/bin/clang++-10 100; \
    update-alternatives --install /usr/bin/c++ c++ /usr/bin/clang++-10 100; \
    update-alternatives --install /usr/bin/clang-tidy clang-tidy /usr/bin/clang-tidy-10 100; \
    update-alternatives --install /usr/bin/clang-format clang-format /usr/bin/clang-format-10 100; \
    update-alternatives --install /usr/bin/llvm-cov llvm-cov /usr/bin/llvm-cov-10 100; \
    update-alternatives --install /usr/bin/llvm-profdata llvm-profdata /usr/bin/llvm-profdata-10 100

RUN set -eux; \
    echo "Installing CMake ..."; \
    CMAKE_URL=https://github.com/Kitware/CMake/releases/download/v3.18.0/cmake-3.18.0-Linux-x86_64.tar.gz; \
    mkdir -p /opt/cmake; \
    wget --no-verbose -c ${CMAKE_URL} -O - | tar -xz --strip-components 1 -C /opt/cmake/

ENV PATH="/opt/cmake/bin:${PATH}"


ARG VSCODE="false"

RUN set -eux; \
    if [ "$VSCODE" = "true" ]; then \
    echo "Installing extra IDE tools ..."; \
    export DEBIAN_FRONTEND=noninteractive; \
    apt-get -y install --no-install-recommends clangd-10 gdb; \
    update-alternatives --install /usr/bin/clangd clangd /usr/bin/clangd-10 100; \
    echo "Set disable_coredump false" >> /etc/sudo.conf; \
    fi

# Additional libraries -----------------------------


RUN set -eux; \
    echo "Installing Infer ..."; \
    # https://fbinfer.com/docs/getting-started/
    export DEBIAN_FRONTEND=noninteractive; \
    apt-get -y install --no-install-recommends \
    # https://github.com/facebook/infer/issues/1168#issuecomment-722381509
    libtinfo5 \
    python2.7; \
    DIR=/additional-deps/infer; \
    mkdir -p $DIR; \
    cd $DIR; \
    VERSION=1.0.0; \
    curl -sSL "https://github.com/facebook/infer/releases/download/v$VERSION/infer-linux64-v$VERSION.tar.xz" \
    | sudo tar -C /opt -xJ && \
    ln -s "/opt/infer-linux64-v$VERSION/bin/infer" /usr/local/bin/infer \
    ;


RUN set -eux; \
    echo "Downloading OCLint ..."; \
    # https://github.com/oclint/oclint/releases
    # https://docs.oclint.org/en/stable/intro/installation.html
    export DEBIAN_FRONTEND=noninteractive; \
    DIR=/additional-deps/oclint; \
    mkdir -p $DIR; \
    cd $DIR; \
    PACKAGE=oclint-20.11-llvm-11.0.0-x86_64-linux-ubuntu-20.04.tar.gz \
    URL=https://github.com/oclint/oclint/releases/download/v20.11/$PACKAGE; \
    curl -sSLO $URL; \
    tar -xvzf $PACKAGE;

RUN set -eux; \
    echo "Installing IKOS and IKOS dependencies ..."; \
    # https://github.com/NASA-SW-VnV/ikos/blob/master/doc/install/UBUNTU_20.04.md
    export DEBIAN_FRONTEND=noninteractive; \
    apt-get -y install --no-install-recommends \
    gcc \
    g++ \
    libgmp-dev \
    libboost-dev \
    libboost-filesystem-dev \
    libboost-thread-dev \
    libboost-test-dev \
    python3-pygments \
    python3-distutils \
    libsqlite3-dev \
    libtbb-dev \
    libz-dev \
    libedit-dev \
    llvm-9 \
    llvm-9-dev \
    llvm-9-tools \
    clang-9 \
    ; \
    # https://github.com/NASA-SW-VnV/ikos/blob/master/README.md#build-and-install
    DIR=/additional-deps/ikos/download; \
    mkdir -p $DIR; \
    cd $DIR; \
    PACKAGE=ikos-3.0.tar.gz \
    URL=https://github.com/NASA-SW-VnV/ikos/releases/download/v3.0/$PACKAGE; \
    curl -sSLO $URL; \
    tar -xvzf $PACKAGE; \
    cd ikos-3.0; \
    mkdir build; \
    cd build; \
    INSTALL_DIR=/additional-deps/ikos/install; \
    cmake \
    -DCMAKE_INSTALL_PREFIX=$INSTALL_DIR \
    -DLLVM_CONFIG_EXECUTABLE="/usr/bin/llvm-config-9" \
    ..;\
    make; \
    make install;


RUN set -eux; \
    echo "Downloading Coderrect ..."; \
    # https://coderrect.com/documentation/
    export DEBIAN_FRONTEND=noninteractive; \
    DIR=/additional-deps/coderrect; \
    mkdir -p $DIR; \
    cd $DIR; \
    PACKAGE=coderrect-linux-hpc-0.9.1.tar.gz \
    URL=https://public-installer-pkg.s3.us-east-2.amazonaws.com/$PACKAGE; \
    wget $URL; \
    tar -xvzf $PACKAGE;

RUN set -eux; \
    echo "Installing Traffic Server additional tools ..."; \
    # https://cwiki.apache.org/confluence/display/TS/Profiling
    export DEBIAN_FRONTEND=noninteractive; \
    apt-get -y install --no-install-recommends \
    oprofile \
    valgrind \
    kcachegrind \
    doxygen \
    ;