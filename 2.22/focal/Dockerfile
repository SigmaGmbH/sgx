FROM ubuntu:20.04 as sdk

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && apt-get install -y \
                build-essential \
                libcurl4-openssl-dev \
                libprotobuf-dev \
                libssl-dev \
                pkg-config \
                wget \
        && rm -rf /var/lib/apt/lists/*


ENV INTEL_SGX_URL "https://download.01.org/intel-sgx"
ENV LINUX_SGX_VERSION "2.22"

ARG INSTALL_ROOT_DIR="/opt/intel"
ARG SGX_SDK="${INSTALL_ROOT_DIR}/sgxsdk"
ENV SGX_SDK ${SGX_SDK}

# prebuilt binutils
RUN set -eux; \
    pkg="as.ld.objdump.r4.tar.gz"; \
    url="${INTEL_SGX_URL}/sgx-linux/${LINUX_SGX_VERSION}/${pkg}"; \
    sha256="1c4ab5814db1e79516985c6128405f92d131b0125e5f3fc5948e94a319e92985"; \
    wget "${url}" --progress=dot:giga; \
    echo "${sha256} *${pkg}" | sha256sum --strict --check -; \
    tar -xvf ${pkg} --directory /usr/local/bin/; \
    rm -f ${pkg};

# SDK
RUN set -eux; \
    distro="ubuntu20.04-server"; \
    version="2.22.100.3"; \
    pkg="sgx_linux_x64_sdk_${version}.bin"; \
    url="${INTEL_SGX_URL}/sgx-linux/${LINUX_SGX_VERSION}/distro/${distro}/${pkg}"; \
    sha256="7f6d8a0ece65757ccd714c46c0e8baa7f1c6a8428bc7b85bf42b5fd834bf19d0"; \
    wget -O sdk.bin "${url}" --progress=dot:giga; \
    echo "$sha256 *sdk.bin" | sha256sum --strict --check -; \
    chmod +x sdk.bin; \
    echo -e "no\n/${INSTALL_ROOT_DIR}" | ./sdk.bin; \
    echo "source ${SGX_SDK}/environment" >> /root/.bashrc; \
    rm -f sdk.bin;

WORKDIR ${SGX_SDK}


# PSW
FROM sdk as psw

RUN set -eux; \
    distro="focal"; \
    url="${INTEL_SGX_URL}/sgx_repo/ubuntu"; \
    echo "deb [arch=amd64] ${url} ${distro} main" \
                | tee /etc/apt/sources.list.d/intel-sgx.list; \
    wget -qO - "${url}/intel-sgx-deb.key" | apt-key add -; \
    apt-get update; \
    apt-get install -y --no-install-recommends \
                libsgx-headers \
                libsgx-ae-epid \
                libsgx-ae-le \
                libsgx-ae-pce \
                libsgx-aesm-epid-plugin \
                libsgx-aesm-launch-plugin \
                libsgx-aesm-pce-plugin \
                libsgx-aesm-quote-ex-plugin \
                libsgx-enclave-common \
                libsgx-enclave-common-dev \
                libsgx-epid \
                libsgx-epid-dev \
                libsgx-launch \
                libsgx-launch-dev \
                libsgx-quote-ex \
                libsgx-quote-ex-dev \
                libsgx-uae-service \
                libsgx-urts \
                sgx-aesm-service; \
    rm -rf /var/lib/apt/lists/*;


# SGX SSL
FROM psw as ssl

# NOTE Versions for openssl and sgx ssl should match.
# See the intel-sgx-ssl repo tags for more information.
ARG OPENSSL_VERSION="3.0.10"
ARG SGX_SSL_COMMIT="ef50655895c869146bf73a889604fe462867a7ce"
ARG SGX_MODE=SIM
ARG SGX_SSL="${INSTALL_ROOT_DIR}/sgxssl"

ENV SGX_SSL ${SGX_SSL}

RUN apt-get update && apt-get install -y \
                git \
                nasm \
        && rm -rf /var/lib/apt/lists/*

WORKDIR ${SGX_SSL}

ENV PKG_CONFIG_PATH ${SGX_SDK}/pkgconfig
ENV LD_LIBRARY_PATH ${SGX_SDK}/sdk_libs
ENV PATH ${PATH}:${SGX_SDK}/bin:${SGX_SDK}/bin/x64

RUN set -eux; \
    git clone https://github.com/intel/intel-sgx-ssl.git ${SGX_SSL}; \
    git checkout ${SGX_SSL_COMMIT}; \
    \
    pkg="openssl-${OPENSSL_VERSION}.tar.gz"; \
    openssl_url="https://www.openssl.org/source/${pkg}"; \
    sha256="1761d4f5b13a1028b9b6f3d4b8e17feb0cedc9370f6afe61d7193d2cdce83323"; \
    wget ${openssl_url} -P openssl_source; \
    echo "${sha256} openssl_source/${pkg}" | sha256sum --strict --check -; \
    \
    make -C Linux sgxssl_no_mitigation SGX_MODE=${SGX_MODE}; \
    DESTDIR=${SGX_SSL} make -C Linux install;
