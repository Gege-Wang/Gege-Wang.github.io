---
layout: single
title:  "对于构建分布式 dora 的一些思考"
date:  2024-06-24
categories: dora
---

> 关于怎么在分布式的情况下管理 working_dir，这里写下了我的一些思考。

# 1. 关于 dataflow.yml 的一些改动
添加了以下条目： 
```
local: Option<bool> # 判断这个节点是否和 cli 在同一个机器上
working_dir: Option<String> # 这个节点所在的 dataflow 在这个机器上的工作目录
```

## 1.1 为什么要添加 local 这个条目？

本来想了两个方法来自行判断是否为 local:
1. 在 cli 中判断，cli 从 coordiantor 中获取 `remote machine` 的信息。但其实这样判断不了，因为coordinator 记录的 daemon 的地址，如果和 coordiantor一样就是 127.0.0.1， 如果不一样就是 daemon 自己的 ip。 当 cli 获取了这些 ip 之后，他还是没法判断 哪个 daemon 和自己在一起 （cli 和 daemon 在一起，但是不和 coordinator 在一起）。如果通过 id 判断， cli 并不知道自己的机器是否有 id 或者 id 是多少。

2. 在 coordinator 中判断
在 coordinator 中记录 cli 的地址，这样每次把 cli 和 daemon 的地址比较一下就知道哪个 daemon 在本地。但是我没法获得 cli 的地址。因为 coordinator 在获取事件的时候会将事件从各个流中拿出来，之后再去处理。这个中间是解耦的。所以如果我想在 start dataflow 的时候就获得 cli 的地址这是不行的，因为所有从 cli 来的事件都在处理 connection 的时候放到了 task 当中。我的 start dataflow 的事件是从 task 中拿出来的，这里就没法看到当时的 tcpstream 了。

综上所述，我就让用户自己去配置了。

## 1.2 为什么要添加 working_dir 这个条目？

一开始的方案是一个 daemon 配置一个 working_dir， 默认是 daemon 启动的目录。如果想配置的话就可以像 --coordinator-addr 一样在命令行输入。但是呢，这样有两个不好处：

1. 如果一个 daemon 上有多个 dataflow 的节点的话，它们都使用同一个 daemon 的 working_dir，这不合理。
2. 所有 source 的相对目录都是相对于 daemon 的 working_dir 写的，这不合理。（举一个例子，如果所有的节点都在本地，source 是相对于 dataflow.yml 写的，这对用户很方便，但是现在我们去需要指定 dataflow.yml 所在的目录是工作目录才行。）

理论上，对于一个 dataflow 来说，它所涉及到的每个 daemon 都应该配置 这个 dataflow 自己特定的工作目录。所以我仿照 machine_id, 在 node 的 deploy 中 添加了这两个条目

这里还有两个待解决的问题：

1. dataflow 的 Deploy 和 Node 的 Deploy 有什么关系

2. daemon 的 working_dir 到底是在 node 的配置里面写，还是在 总体上写。这里是一个分配律

# 2. 关于 check_dataflow 的一些逻辑

1. local -> 即 cli 和 这个 daemon 在一起， 按照以前的逻辑 检查 path exist
2. !local -> Some(Node.deploy.working_dir) ，不做任何检查，路径可以是 relative ,也可以是 absolute; None 必须是绝对路径，如果是相对路径抛出错误。
3. coordinator 中可以不用 check_dataflow 了，因为 这个东西必须是 daemon check 才行， 否则 path exist 和 python 环境在 coordinator 中没有必要存在。

# 3. 关于 在 daemon 中改变 working_dir 的逻辑

在 daemon 的 spawn_node 的时候 改变 working_dir. 如果是 Some(working_dir), 那么改变 working_dir（这里使用的是 dirs, 都是按照 daemon 自己的操作系统类型配置的 路径，不存在类似 path is_relative 的 兼容性问题）。 如果是 None, 看是不是 local ， 如果是 local ,那么使用 cli 的 working_dir, 如果 !local,那么使用默认的 working_dir（这里我隐式修改了 working_dir ,或许也可以直接抛出错误，告诉用户必须指定working_dir）

在 spawn_node 的时候开始如果是绝对路径就使用绝对路径，如果不是 就按照相对路径的逻辑（因为这里的 working_dir 都已经配置好了）

# fix peer_ip 的问题
已修复

# path is_relative 的问题
未提交

# how to use dora
we use the multiple-daemons to test.
the dataflow are like this:

```yaml
nodes:
  - id: rust-node
    _unstable_deploy:
      machine: A
      local: true
    custom:
      build: cargo build -p multiple-daemons-example-node
      # The path is for CI, replace it with your source absolute path here
      source: ../../target/debug/multiple-daemons-example-node
      inputs:
        tick: dora/timer/millis/10
      outputs:
        - random
  - id: runtime-node
    _unstable_deploy:
      machine: B
      local: false
      working_dir: /Users/runner/work/dora/dora/examples/multiple-daemons
    operators:
      - id: rust-operator
        build: cargo build -p multiple-daemons-example-operator
        # The path is for CI, replace it with your source absolute path here
        shared-library: ../../target/debug/multiple_daemons_example_operator
        inputs:
          tick: dora/timer/millis/100
          random: rust-node/random
        outputs:
          - status
  - id: rust-sink
    _unstable_deploy:
      machine: A
      local: true
    custom:
      build: cargo build -p multiple-daemons-example-sink
      # The path is for CI, replace it with your source absolute path here
      source: ../../target/debug/multiple-daemons-example-sink
      inputs:
        message: runtime-node/rust-operator/status
```

There are some things that we should pay attention to.
- I add two item for every node. local(whether they are the same as cli) and working_dir(the working_dir of the daemon for this dataflow. maybe we should configure them in the head of dataflow.yml, so we don't need to write it in every node)
the local(default true) and working_dir(no default value) are all option.
- when you use absolute path, if you specify the working_dir, we will use this working_dir, if you not specify, we will change the working_dir to the home_directory(such as in linux,/home/miyamo). I do this because if you don't specify the working_dir but use absolute path, the dataflow can still run.(if don't want to change working_dir implicit, I  can alse throw an error when users don't specify working_dir )
- when you use relative path, if the node are local, we can use the dataflow.yml working_dir from cli, just like now. if the node is not local, we must specify working_dir for it(I alse check this in check_dataflow), otherwise dora will throw an error.

# how to start a distributed dataflow
You can pull docker image and run some containers.
for two machine_id, there are some cases.

|      num   | cli         | coordinator |   daemon A  |   daemon B |
| ---------- | ----------- | ----------- | ----------- | ---------- |
|     1      | local       | local       |   local     |   local    |
|     2      | local       | local       |   local     |   remote   |
|     3      | local       | local       |   remote    |   remote   |
|     4      | local       | remote      |   local     |   local    |
|     5      | local       | remote      |   local     |   remote   |
|     6      | local       | remote      |   remote    |   remote   |

for number 2, you should run two container, and get the ip address of every container.

1. get the docker image, **Note that the container ubuntu is Arm.**
```
docker pull miyamo505/dora-distributed:v1.0
```

2. run docker container
```
docker run -dt --name 01_dora miyamo505/dora-distributed:v1.0 
docker run -dt --name 02_dora miyamo505/dora-distributed:v1.0  
```

3. get into the container and get its ip
```
docker run -it --name 01_dora /bin/bash
hostname -i 
```
start the coordinator and daemon in every machine.

```shell
# in machine A
cd 
cd dora
git remote -v  # see remote repo
git checkout miyamo/configure-working-directory
cargo install --path binaries/cli --locked
dora coordinator coordinator
dora daemon --machine-id A 
```

```shell
in machine B
cd 
cd dora
git remote -v  # see remote repo
git checkout miyamo/configure-working-directory
cargo install --path binaries/cli --locked
dora daemon --machine-id B --coordinator-addr <01_doraIP>:53290
```

start the dataflow in any machine you want.
```shell
dora start ./examples/multiple-daemons/dataflow.yml
```

you can change the dataflow.yml and test.
