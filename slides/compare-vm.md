name: inverse
layout: true
class: center, middle, inverse

---

.percent80[.center[![bg](img/WiseMonkeys-1.jpg)]]

# 性能評比／與 VM 比較

???

Img src: http://freer.com/bits/wp-content/uploads/2013/11/Three-Wise-Monkeys-in-the-Internet-Age.jpg

---

layout: false

# Lab setup

--

.pull-left[
## VMs

1. `centos`:
   - `up`
   - `ssh`

2. `main`:
   - `up`
   - `ssh`

3. `registry`:
   - `up`
]

--

.pull-right[
## Lab directory
- workshop root
]

---

template: inverse

# "Virtualization" <br/> vs <br/> "Containerization"

---

# Similar?

.pull-right[
## Docker
- Dependency
- Isolation
]

--

.pull-left[
## Virtual machine
- Dependency
- Isolation
]

---

.pull-left[
## Virtual machine
]

.pull-right[
## Docker
]
<br clear="all">

```
 `+------------------------+`         |  `+------------------------+`
 `|  App                   |`         |  `|  App                   |`
 `+------------------------+`         |  `+------------------------+`

     `+----------------------+`       |      `+----------------------+`
     `|  PL library/package/ |`       |      `|  PL library/package/ |`
     `|  module/extension    |`       |      `|  module/extension    |`
     `+----------------------+`       |      `+----------------------+`

 `+---------+`   `+-----------------+`  |  `+---------+`   `+-----------------+`
 `| PL      |`   `| special-purpose |`  |  `| PL      |`   `| special-purpose |`
 `| runtime |`   `| runtime lib     |`  |  `| runtime |`   `| runtime lib     |`
 `+---------+`   `+-----------------+`  |  `+---------+`   `+-----------------+`

 `+-------------------------------+`  |  `+-------------------------------+`
 `|  libc & generic runtime lib   |`  |  `|  libc & generic runtime lib   |`
 `+-------------------------------+`  |  `+-------------------------------+`

 `+-------------------------------+`  |  `+-------------------------------+`
 `|  root file system             |`  |  `|  root file system             |`
 `+-------------------------------+`  |  `+-------------------------------+`

 `+-------------------------------+`  |  +-------------------------------+
 `|  Linux kernel                 |`  |  |  Linux kernel ≥ 3.10          |
 `+-------------------------------+`  |  +-------------------------------+
```


---

## In Practice

- Startup time

- Kernel

- Runtime performance


---

template: inverse

# Task #1: Startup time

---

.pull-left[
## Virtual machine
]

.pull-right[
## Docker
]
<br clear="all">

```
 `+------------------------+`         |  `+------------------------+`
 `|  App                   |`         |  `|  App                   |`
 `+------------------------+`         |  `+------------------------+`

     `+----------------------+`       |      `+----------------------+`
     `|  PL library/package/ |`       |      `|  PL library/package/ |`
     `|  module/extension    |`       |      `|  module/extension    |`
     `+----------------------+`       |      `+----------------------+`

 `+---------+`   `+-----------------+`  |  `+---------+`   `+-----------------+`
 `| PL      |`   `| special-purpose |`  |  `| PL      |`   `| special-purpose |`
 `| runtime |`   `| runtime lib     |`  |  `| runtime |`   `| runtime lib     |`
 `+---------+`   `+-----------------+`  |  `+---------+`   `+-----------------+`

 `+-------------------------------+`  |  `+-------------------------------+`
 `|  libc & generic runtime lib   |`  |  `|  libc & generic runtime lib   |`
 `+-------------------------------+`  |  `+-------------------------------+`

 `+-------------------------------+`  |  `+-------------------------------+`
 `|  root file system             |`  |  `|  root file system             |`
 `+-------------------------------+`  |  `+-------------------------------+`

 `+-------------------------------+`  |  +-------------------------------+
 `|  Linux kernel                 |`  |  |  Linux kernel ≥ 3.10          |
 `+-------------------------------+`  |  +-------------------------------+
```

---

# 前置作業

- Rollback to clean state

  ```bash
  % vagrant snapshot back  main
  % vagrant snapshot back  centos
  ```
--

- Shutdown all VMs

  ```bash
  % vagrant halt
  ```

--

- Disable auto-check, to be fair...

   ```yaml
   Vagrant.configure(2) do |config|

      config.vm.define "centos" do |node|
          node.vm.box_check_update = false  ☚ 加上這行！
      end

   end
   ```

---

# Task 1A: VM startup time

測量以下時間：

```bash
% vagrant up  centos
```

---

# Task 1B: Container startup time

先備妥環境：

```bash
% vagrant up   registry
% vagrant up   main
% vagrant ssh  main

$ # 視網路狀況而定...
$ DOCKER pull  centos:5.11
$ docker images
```

--

再測量以下時間：

```bash
$ docker run centos:5.11   echo "Hello!"
```


---

template: inverse

# Task #2: Kernel

---

.pull-left[
## Virtual machine
]

.pull-right[
## Docker
]
<br clear="all">

```
 `+------------------------+`         |  `+------------------------+`
 `|  App                   |`         |  `|  App                   |`
 `+------------------------+`         |  `+------------------------+`

     `+----------------------+`       |      `+----------------------+`
     `|  PL library/package/ |`       |      `|  PL library/package/ |`
     `|  module/extension    |`       |      `|  module/extension    |`
     `+----------------------+`       |      `+----------------------+`

 `+---------+`   `+-----------------+`  |  `+---------+`   `+-----------------+`
 `| PL      |`   `| special-purpose |`  |  `| PL      |`   `| special-purpose |`
 `| runtime |`   `| runtime lib     |`  |  `| runtime |`   `| runtime lib     |`
 `+---------+`   `+-----------------+`  |  `+---------+`   `+-----------------+`

 `+-------------------------------+`  |  `+-------------------------------+`
 `|  libc & generic runtime lib   |`  |  `|  libc & generic runtime lib   |`
 `+-------------------------------+`  |  `+-------------------------------+`

 `+-------------------------------+`  |  `+-------------------------------+`
 `|  root file system             |`  |  `|  root file system             |`
 `+-------------------------------+`  |  `+-------------------------------+`

 `+-------------------------------+`  |  +-------------------------------+
 `|  Linux kernel                 |`  |  |  Linux kernel ≥ 3.10          |
 `+-------------------------------+`  |  +-------------------------------+
```

---

# Task 2A: VM kernel

.pull-left[
## CentOS 5.11

```bash
% vagrant ssh  centos

$ uname -a
```
]

--

.pull-right[
## Ubuntu 14.04

```bash
% vagrant ssh  main

$ uname -a
```
]

---

# Task 2B: Container's view of kernel

.center[ `% vagrant ssh  main` ]


--

.pull-left[
## CentOS 5.11 image

```bash
$ docker run  centos:5.11  \
      uname -a
```
]

--

.pull-right[
## Ubuntu 14.04 image

```bash
$ DOCKER pull  ubuntu:14.04

$ docker run   ubuntu:14.04  \
      uname -a
```
]

---

template: inverse

# Task #3: Runtime performance

---

# To be fair...

Bare metal machine is needed...

.pull-right[
## ➤ Task for Docker

Software stack:

- *Dockerized app*
- *Docker Engine*
- Linux
- Bare metal machine
]


.pull-left[
## ➤ Task for VM

Software stack:

- *App*
- *Virtual Machine*
- Linux
- Bare metal machine

]

---

template: inverse

# Recap
## Virtualization vs Containerization

---

class: center, middle

# Similarities

Dependency

Isolation

--

# Differences

Resource consumption

Kernel

Startup time

Runtime performance

---

class: center, middle

# Questions?
