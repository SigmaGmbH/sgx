# docker linux-sgx
Dockerfiles for [linux-sgx](https://github.com/intel/linux-sgx).

Provided versions:

SGX version | OS | SDK | PSW | SGX SSL
--- | --- | --- | --- | ---
2.19 | <br>Ubuntu 22.04</br><br>Debian 10</br>| :heavy_check_mark: | :heavy_check_mark: | <br>:heavy_check_mark:</br><br>:x:</br>
2.18 | <br>Ubuntu 20.04</br><br>Ubuntu 22.04</br> | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark:
2.17.1 | <br>Ubuntu 20.04</br> | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark:
2.16 | <br>Ubuntu 20.04</br> | :heavy_check_mark: | :heavy_check_mark: | :x:
2.15.1 | <br>Ubuntu 20.04</br> | :heavy_check_mark: | :heavy_check_mark: | :x:
2.14 | <br>Ubuntu 20.04</br> | :heavy_check_mark: | :heavy_check_mark: | :x:
2.13.3 | <br>Ubuntu 18.04</br><br>Ubuntu 20.04</br> | :heavy_check_mark: | :heavy_check_mark: | :x:

Please refer to the official repository,
https://github.com/intel/linux-sgx, for other versions.

Images are available on under [ghcr.io/sigmagmbh/linux-sgx][ghcr.io/sigmagmbh/linux-sgx].


## Usage

```dockerfile
FROM ghcr.io/sigmagmbh/sgx:2.19-buster

# ...
```

## sgx aesm service
There's a dedicated image to run the sgx aesm service in a container.

It can be used with docker compose. For example:

```yml
version: '3.9'

services:

  aesmd:
    image: ghcr.io/sigmagmbh/sgx-aesm:2.19-buster
    volumes:
      - aesmd-socket:/var/run/aesmd
    devices:
      - /dev/sgx_enclave
      - /dev/sgx_provision

  sample-enclave:
    image: sample-enclave
    depends_on:
      aesmd:
        condition: service_started
    volumes:
      - aesmd-socket:/var/run/aesmd
    devices:
      - /dev/sgx_enclave

volumes:
  aesmd-socket:
    driver: local
    driver_opts:
      type: "tmpfs"
      device: "tmpfs"
      o: "rw"
```

Complete example under [examples/sample-enclave](examples/sample-enclave).


## Older versions
The following versions are available on DockerHub at
https://hub.docker.com/r/initc3/linux-sgx.

SGX version | OS | SDK | PSW | SGX SSL
--- | --- | --- | --- | ---
2.12 | Ubuntu 18.04 | :heavy_check_mark: | :heavy_check_mark: | :x:
2.11 | Ubuntu 18.04 | :heavy_check_mark: | :heavy_check_mark: | :x:
2.9.1 | Ubuntu 18.04 | :heavy_check_mark: | :heavy_check_mark: | :x:
2.7.1 | Ubuntu 18.04 | :heavy_check_mark: | :heavy_check_mark: | :x:
2.6 | <br>Ubuntu 16.04</br><br>Ubuntu 18.04</br> | :heavy_check_mark: | :heavy_check_mark: | :x:
2.3.1 | <br>Ubuntu 16.04</br><br>Ubuntu 18.04</br> | <br>:heavy_check_mark:</br><br>:heavy_check_mark:</br> | <br>:x:</br><br>:heavy_check_mark:</br> | :x:
2.2 | Ubuntu 16.04 | :heavy_check_mark: | :x: | :x:
2.1.3 | Ubuntu 16.04 | :heavy_check_mark: | :x: | :x:


[ghcr.io/sigmagmbh/linux-sgx]: https://github.com/sigmagmbh/sgx/pkgs/container/linux-sgx
