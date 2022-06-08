# Nume Enclave KMS SDK 

## License

This project is licensed under the Apache-2.0 License.

## Dependencies
| name                       | version              | link                                              |
|----------------------------|----------------------|---------------------------------------------------|
| aws-lc                     | v1.0.2               | https://github.com/awslabs/aws-lc/                |
| S2N                        | v1.3.11              | https://github.com/aws/s2n-tls.git                |
| aws-c-common               | v0.6.20              | https://github.com/awslabs/aws-c-common           |
| aws-c-sdkutils             | v0.1.2               | https://github.com/awslabs/aws-c-sdkutils         |
| aws-c-io                   | v0.10.21             | https://github.com/awslabs/aws-c-io               |
| aws-c-compression          | v0.2.14              | https://github.com/awslabs/aws-c-compression      |
| aws-c-http                 | v0.6.13              | https://github.com/awslabs/aws-c-http             |
| aws-c-cal                  | v0.5.17              | https://github.com/awslabs/aws-c-cal              |
| aws-c-auth                 | v0.6.11              | https://github.com/awslabs/aws-c-auth             |
| aws-nitro-enclaves-nsm-api | v0.2.1               | https://github.com/aws/aws-nitro-enclaves-nsm-api |
| json-c                     | json-c-0.16-20220414 | https://github.com/json-c/json-c                  |

## Building

Building this step project is multistep process. First we need to fetch all dependencies mentioned above, and build them individually , then copy them inside nume-enclave-sdk directory. From here we should build AWS Nitro Enclaves SDK for C . Building this step would generate our binary inside directory

```
aws-nitro-enclaves-sdk-c/build/bin/kmstool-enclave
```

### Linux - Using containers:
Following commnads should be executed in sequence to get desired output

Tools setup

```

amazon-linux-extras enable epel

yum clean -y metadata && yum install -y epel-release

yum install -y cmake3 gcc git tar make

curl https://sh.rustup.rs -sSf | sh -s -- -y --profile minimal --default-toolchain 1.60

yum install -y gcc-c++

yum install -y go

yum install -y ninja-build

mkdir /tmp/crt-builder

```

Building Dependencies
```
git clone -b v1.0.2 https://github.com/awslabs/aws-lc.git aws-lc #

cmake3 -DCMAKE_PREFIX_PATH=/usr -DCMAKE_INSTALL_PREFIX=/usr -GNinja -S aws-lc -B aws-lc/build .

go env -w GOPROXY=direct

cmake3 --build aws-lc/build --target install




git clone -b v1.3.11 https://github.com/aws/s2n-tls.git

cmake3 -DCMAKE_PREFIX_PATH=/usr -DCMAKE_INSTALL_PREFIX=/usr -S s2n-tls -B s2n-tls/build

cmake3 --build s2n-tls/build --target install




git clone -b v0.6.20 https://github.com/awslabs/aws-c-common.git

cmake3 -DCMAKE_PREFIX_PATH=/usr -DCMAKE_INSTALL_PREFIX=/usr -GNinja -S aws-c-common -B aws-c-common/build

cmake3 --build aws-c-common/build --target install




git clone -b v0.1.2 https://github.com/awslabs/aws-c-sdkutils.git

cmake3 -DCMAKE_PREFIX_PATH=/usr -DCMAKE_INSTALL_PREFIX=/usr -GNinja -S aws-c-sdkutils -B aws-c-sdkutils/build

cmake3 --build aws-c-sdkutils/build --target install




git clone -b v0.5.17 https://github.com/awslabs/aws-c-cal.git

cmake3 -DCMAKE_PREFIX_PATH=/usr -DCMAKE_INSTALL_PREFIX=/usr -GNinja -S aws-c-cal -B aws-c-cal/build

cmake3 --build aws-c-cal/build --target install




git clone -b v0.10.21 https://github.com/awslabs/aws-c-io.git

cmake3 -DUSE_VSOCK=1 -DCMAKE_PREFIX_PATH=/usr -DCMAKE_INSTALL_PREFIX=/usr -GNinja -S aws-c-io -B aws-c-io/build

cmake3 --build aws-c-io/build --target install




git clone -b v0.2.14 http://github.com/awslabs/aws-c-compression.git

cmake3 -DCMAKE_PREFIX_PATH=/usr -DCMAKE_INSTALL_PREFIX=/usr -GNinja -S aws-c-compression -B aws-c-compression/build

cmake3 --build aws-c-compression/build --target install




git clone -b v0.6.13 https://github.com/awslabs/aws-c-http.git

cmake3 -DCMAKE_PREFIX_PATH=/usr -DCMAKE_INSTALL_PREFIX=/usr -GNinja -S aws-c-http -B aws-c-http/build

cmake3 --build aws-c-http/build --target install





git clone -b v0.6.11 https://github.com/awslabs/aws-c-auth.git

cmake3 -DCMAKE_PREFIX_PATH=/usr -DCMAKE_INSTALL_PREFIX=/usr -GNinja -S aws-c-auth -B aws-c-auth/build

cmake3 --build aws-c-auth/build --target install




git clone -b json-c-0.16-20220414 https://github.com/json-c/json-c.git

cmake3 -DCMAKE_PREFIX_PATH=/usr -DCMAKE_INSTALL_PREFIX=/usr -DBUILD_SHARED_LIBS=OFF -GNinja -S json-c -B json-c/build

cmake3 --build json-c/build --target install





git clone -b v0.2.1 https://github.com/aws/aws-nitro-enclaves-nsm-api.git

source $HOME/.cargo/env && cd aws-nitro-enclaves-nsm-api && cargo build --release -p nsm-lib

mv aws-nitro-enclaves-nsm-api/target/release/libnsm.so /usr/lib64

mv aws-nitro-enclaves-nsm-api/target/release/nsm.h /usr/include

```


Copy all generated binaries to CWD of package
```
cp -r /tmp/crt-builder/* /home/ec2-user/aws-nitro-enclaves-sdk-c
```


Build Nitro-Enclave-SDK
```
cmake3 -DCMAKE_PREFIX_PATH=/usr -DCMAKE_INSTALL_PREFIX=/usr -GNinja 	-S aws-nitro-enclaves-sdk-c -B aws-nitro-enclaves-sdk-c/build

sudo cmake3 --build aws-nitro-enclaves-sdk-c/build --target install

sudo cmake3 --build aws-nitro-enclaves-sdk-c/build --target docs

cmake3 --build aws-nitro-enclaves-sdk-c/build --target test
```

Copy to Enclave codebase
```
cp ../aws-nitro-enclaves-sdk-c/build/bin/kmstool-enclave/kmstool_enclave .
```

### Windows
Note that this SDK is currently not supported on Windows.  Only the client side sample application (kmstool_instance) is supported on Windows.

## Samples
 * [kmstool](docs/kmstool.md)
 * [kmstool-enclave-cli](docs/kmstool.md#kmstool-enclave-cli)

## Security issue notifications

If you discover a potential security issue in the Nitro Enclaves SDK for C, we ask that you notify AWS
Security via our
[vulnerability reporting page](https://aws.amazon.com/security/vulnerability-reporting/).
Please do **not** create a public GitHub issue.
