# 使用构建工具Bazel运行prysm测试网

[prysm](https://github.com/prysmaticlabs/prysm)是Prysmatic labs团队构建的以太坊Serenity客户端。该客户端主要由beacon节点和Validator客户端组成。

## 1 准备环境

查看当前的操作系系统版本

```
$ lsb_release
Distributor ID: Ubuntu
Description:    Ubuntu 18.04.1 LTS
Release:        18.04
Codename:       bionic
```

查看go语言版本

```
$ go version
go version go1.10.4 linux/amd64
```

## 2 安装bazel

[Bazel官网](https://docs.bazel.build/versions/master/install-ubuntu.html)提供两个安装选项。

我们采用二进制安装程序。

首先，安装必备的包。

```
$ sudo apt-get install pkg-config zip g++ zlib1g-dev unzip python
```

然后，开始下载bazel！

运行安装程序

我们前往[Github](https://github.com/bazelbuild/bazel/releases)下载bazel-0.25.0-installer-linux-x86_64.sh。

```
$ chmod +x bazel-<version>-installer-linux-x86_64.sh
$ ./bazel-<version>-installer-linux-x86_64.sh --user
```

设置环境变量

```
$ export PATH="$PATH:$HOME/bin"
```

检查bazel是否安装成功

```
$ bazel version
Extracting Bazel installation...
Starting local Bazel server and connecting to it...
Build label: 0.25.0
Build target: bazel-out/k8-opt/bin/src/main/java/com/google/devtools/build/lib/bazel/BazelServer_deploy.jar
Build time: Wed May 1 21:45:01 2019 (1556747101)
Build timestamp: 1556747101
Build timestamp as int: 1556747101
```

查看bazel功能

```
$ bazel --help
                                                          [bazel release 0.25.0]
Usage: bazel <command> <options> ...

Available commands:
  analyze-profile     Analyzes build profile data.
  aquery              Analyzes the given targets and queries the action graph.
  build               Builds the specified targets.
  canonicalize-flags  Canonicalizes a list of bazel options.
  clean               Removes output files and optionally stops the server.
  coverage            Generates code coverage report for specified test targets.
  cquery              Loads, analyzes, and queries the specified targets w/ configurations.
  dump                Dumps the internal state of the bazel server process.
  fetch               Fetches external repositories that are prerequisites to the targets.
  help                Prints help for commands, or the index.
  info                Displays runtime info about the bazel server.
  license             Prints the license of this software.
  mobile-install      Installs targets to mobile devices.
  print_action        Prints the command line args for compiling a file.
  query               Executes a dependency graph query.
  run                 Runs the specified target.
  shutdown            Stops the bazel server.
  sync                Syncs all repositories specified in the workspace file
  test                Builds and runs the specified test targets.
  version             Prints version information for bazel.

Getting more help:
  bazel help <command>
                   Prints help and options for <command>.
  bazel help startup_options
                   Options for the JVM hosting bazel.
  bazel help target-syntax
                   Explains the syntax for specifying targets.
  bazel help info-keys
                   Displays a list of keys used by the info command.
```

so far, so good. 我们安装好了bazel。

## 安装Prysm

拉取代码

$ cd $GOPATH/src && git clone -b 0.1.1 https://github.com/prysmaticlabs/prysm

$ cd prysm 

构建beacon节点

```
$ bazel build //beacon-chain:beacon-chain
DEBUG: Rule 'com_google_protobuf' indicated that a canonical reproducible form can be obtained by modifying arguments shallow_since = "1551387314 -0800"
INFO: Analyzed target //beacon-chain:beacon-chain (536 packages loaded, 15979 targets configured).
INFO: Found 1 target...
INFO: From Generating Descriptor Set proto_library @com_github_libp2p_go_libp2p_autonat//pb:autonat_pb_proto:
[libprotobuf WARNING external/com_google_protobuf/src/google/protobuf/compiler/parser.cc:564] No syntax specified for the proto file: pb/autonat.proto. Please use 'syntax = "proto2";' or 'syntax = "proto3";' to specify a syntax version. (Defaulted to proto2 syntax.)
INFO: From Generating Descriptor Set proto_library @com_google_protobuf//:struct_proto:
external/com_google_protobuf: warning: directory does not exist.
INFO: From Generating Descriptor Set proto_library @com_google_protobuf//:field_mask_proto:
external/com_google_protobuf: warning: directory does not exist.
INFO: From Generating Descriptor Set proto_library @com_google_protobuf//:duration_proto:
external/com_google_protobuf: warning: directory does not exist.
INFO: From Generating Descriptor Set proto_library @com_google_protobuf//:timestamp_proto:
external/com_google_protobuf: warning: directory does not exist.
INFO: From Generating Descriptor Set proto_library @com_google_protobuf//:empty_proto:
external/com_google_protobuf: warning: directory does not exist.
INFO: From Generating Descriptor Set proto_library @com_google_protobuf//:descriptor_proto:
external/com_google_protobuf: warning: directory does not exist.
INFO: From Generating Descriptor Set proto_library @com_google_protobuf//:wrappers_proto:
external/com_google_protobuf: warning: directory does not exist.
INFO: From Generating Descriptor Set proto_library @com_google_protobuf//:source_context_proto:
external/com_google_protobuf: warning: directory does not exist.
INFO: From Generating Descriptor Set proto_library @com_google_protobuf//:any_proto:
external/com_google_protobuf: warning: directory does not exist.
INFO: From Generating Descriptor Set proto_library @com_google_protobuf//:compiler_plugin_proto:
external/com_google_protobuf: warning: directory does not exist.
INFO: From Generating Descriptor Set proto_library //proto/beacon/p2p/v1:v1_proto:
external/com_google_protobuf: warning: directory does not exist.
INFO: From Generating Descriptor Set proto_library @go_googleapis//google/rpc:status_proto:
external/com_google_protobuf: warning: directory does not exist.
INFO: From Generating Descriptor Set proto_library @com_google_protobuf//:type_proto:
external/com_google_protobuf: warning: directory does not exist.
INFO: From Generating Descriptor Set proto_library //proto/sharding/p2p/v1:v1_proto:
external/com_google_protobuf: warning: directory does not exist.
INFO: From Generating Descriptor Set proto_library @com_google_protobuf//:api_proto:
external/com_google_protobuf: warning: directory does not exist.
INFO: From Generating Descriptor Set proto_library //proto/beacon/rpc/v1:v1_proto:
external/com_google_protobuf: warning: directory does not exist.
Target //beacon-chain:beacon-chain up-to-date:
  bazel-bin/beacon-chain/linux_amd64_stripped/beacon-chain
INFO: Elapsed time: 367.783s, Critical Path: 33.73s
INFO: 1961 processes: 1961 linux-sandbox.
INFO: Build completed successfully, 1992 total actions
```

日志显示beacon节点构建成功。生成的二进制程序位于bazel-bin目录下。

构建validator节点

```
$ bazel build //validator:validator
DEBUG: Rule 'com_google_protobuf' indicated that a canonical reproducible form can be obtained by modifying arguments shallow_since = "1551387314 -0800"
INFO: Analyzed target //validator:validator (7 packages loaded, 25 targets configured).
INFO: Found 1 target...
Target //validator:validator up-to-date:
  bazel-bin/validator/linux_amd64_pure_stripped/validator
INFO: Elapsed time: 21.923s, Critical Path: 17.85s
INFO: 193 processes: 193 linux-sandbox.
INFO: Build completed successfully, 196 total actions
```

日志显示validator客户端构建成功。生成的二进制程序位于bazel-bin目录下。

## 运行beacon节点

```
$ bazel run //beacon-chain -- --clear-db --datadir=/tmp/prysm-data
DEBUG: Rule 'com_google_protobuf' indicated that a canonical reproducible form can be obtained by modifying arguments shallow_since = "1551387314 -0800"
INFO: Analyzed target //beacon-chain:beacon-chain (0 packages loaded, 0 targets configured).
INFO: Found 1 target...
Target //beacon-chain:beacon-chain up-to-date:
  bazel-bin/beacon-chain/linux_amd64_stripped/beacon-chain
INFO: Elapsed time: 1.135s, Critical Path: 0.00s
INFO: 0 processes.
INFO: Build completed successfully, 1 total action
INFO: Build completed successfully, 1 total action
[2019-05-07 22:02:19]  INFO node: Using custom parameter configuration
[2019-05-07 22:02:20]  INFO node: Checking db path=/tmp/prysm-data/beaconchaindata
[2019-05-07 22:02:20]  INFO node: Fetching testnet cluster address endpoint=https://beta.prylabs.net/contract
[2019-05-07 22:02:22]  INFO node: Starting beacon node version=Git commit: Local build. Built at: Moments ago
[2019-05-07 22:02:22]  INFO registry: Starting 8 services: [*p2p.Server *powchain.Web3Service *attestation.Service *operations.Service *blockchain.ChainService *sync.Service *rpc.Service *prometheus.Service][2019-05-07 22:02:22]  INFO p2p: Starting service
[2019-05-07 22:02:24]  INFO powchain: Starting service endpoint=wss://goerli.prylabs.net/websocket
[2019-05-07 22:02:24]  INFO attestation: Starting service
[2019-05-07 22:02:24]  INFO operation: Starting service
[2019-05-07 22:02:24]  INFO blockchain: Waiting for ChainStart log from the Validator Deposit Contract to start the beacon chain...
[2019-05-07 22:02:24]  INFO sync: Starting service
[2019-05-07 22:02:24]  INFO rpc: Starting service
[2019-05-07 22:02:24]  INFO rpc: Listening on port port=4000
[2019-05-07 22:02:24]  WARN rpc: You are using an insecure gRPC connection! Provide a certificate and key to connect securely
[2019-05-07 22:02:24]  INFO prometheus: Starting service endpoint=:8080
[2019-05-07 22:02:25]  WARN p2p: Failed to open stream with newly connected peer address=/ip4/35.224.249.2/tcp/30000 error=protocol not supported peer=QmfAgkmjiZNZhr2wFN9TwaRgHouMTBT6HELyzE5A3BT2wK
[2019-05-07 22:02:25]  WARN p2p: Temporarily disabled -- not disconnecting peer. See https://github.com/prysmaticlabs/prysm/issues/2408
[2019-05-07 22:02:32]  INFO powchain: Minimum number of validators reached for beacon-chain to start ChainStartTime=2019-04-30 03:57:00 +0800 CST
[2019-05-07 22:02:32]  INFO blockchain: ChainStart time reached, starting the beacon chain!
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=0
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=1
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=2
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=3
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=4
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=5
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=6
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=7
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=8
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=9
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=10
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=11
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=12
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=13
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=14
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=15
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=16
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=17
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=18
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=19
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=20
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=21
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=22
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=23
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=24
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=25
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=26
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=27
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=28
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=29
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=30
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=31
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=32
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=33
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=34
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=35
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=36
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=37
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=38
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=39
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=40
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=41
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=42
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=43
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=44
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=45
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=46
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=47
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=48
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=49
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=50
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=51
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=52
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=53
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=54
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=55
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=56
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=57
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=58
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=59
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=60
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=61
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=62
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=63
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=64
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=65
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=66
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=67
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=68
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=69
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=70
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=71
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=72
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=73
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=74
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=75
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=76
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=77
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=78
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=79
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=80
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=81
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=82
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=83
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=84
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=85
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=86
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=87
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=88
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=89
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=90
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=91
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=92
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=93
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=94
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=95
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=96
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=97
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=98
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=99
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=100
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=101
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=102
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=103
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=104
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=105
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=106
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=107
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=108
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=109
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=110
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=111
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=112
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=113
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=114
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=115
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=116
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=117
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=118
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=119
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=120
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=121
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=122
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=123
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=124
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=125
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=126
[2019-05-07 22:02:32]  INFO validator: Validator activated activationEpoch=0 index=127
[2019-05-07 22:02:32]  INFO syncQuerier: State has been initialized
[2019-05-07 22:02:32]  INFO regular-sync: Polling peers for latest chain head...
[2019-05-07 22:02:34]  INFO syncQuerier: Received chain head from peer highestSlot=111655 peerID=QmeNguyd6RSUPZ553NY3t6cRhCZJHkkBnv283bAKwzL7pK
[2019-05-07 22:02:34]  INFO syncQuerier: Received chain head from peer highestSlot=111655 peerID=Qmf3mdFAMNGo51hA65exCk8867kPNgs8goLrppch8tHN2f
[2019-05-07 22:02:34]  INFO syncQuerier: Received chain head from peer highestSlot=111655 peerID=QmejcavTbNszCfoDLpH12uqU8dZLmu5JQYLBMLkS4bhwMr
[2019-05-07 22:02:34]  INFO syncQuerier: Received chain head from peer highestSlot=111655 peerID=QmTvPwcokSvR7VVnuhttkNiPGQ4EnizdEN261mtSdu5cMb
[2019-05-07 22:02:34]  INFO syncQuerier: Received chain head from peer highestSlot=111655 peerID=QmReCiTdS8vanfjHJfoCTHeov5wSoBtycKCFGC2qoMjaFU
^C[2019-05-07 22:02:34]  INFO node: Got interrupt, shutting down...
[2019-05-07 22:02:34]  INFO node: Stopping beacon node
[2019-05-07 22:02:34]  INFO prometheus: Stopping service
[2019-05-07 22:02:34]  INFO rpc: Stopping service
[2019-05-07 22:02:34]  INFO syncQuerier: Stopping service
[2019-05-07 22:02:34]  INFO initial-sync: Stopping service
[2019-05-07 22:02:34]  INFO regular-sync: Stopping service
[2019-05-07 22:02:34]  INFO blockchain: Stopping service
[2019-05-07 22:02:34]  INFO syncQuerier: Finished querying state of the network, importing blocks...
[2019-05-07 22:02:34]  INFO operation: Stopping service
[2019-05-07 22:02:34]  INFO attestation: Stopping service
[2019-05-07 22:02:34]  INFO powchain: Stopping service
[2019-05-07 22:02:34]  INFO p2p: Stopping service
[2019-05-07 22:02:34] ERROR powchain: Post https://goerli.prylabs.net: context cancele
```

## 运行Validator客户端

todo