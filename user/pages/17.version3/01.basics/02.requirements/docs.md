---
title: System Requirements
taxonomy:
    category: docs
---

#### System Requirements
* Minimum 1GB of memory for controller, manager or all-in-one container; 1GB for enforcer.
* Shared CPU core for standard workloads, dedicated CPU (one or more) for enforcer for higher network throughput in Protect mode, or controller for high volume (10K+) image scanning.
* Registry image scanning is performed by the controller and the image is pulled and expanded in memory. If expanded image sizes larger than 500MB are expected, consider increasing the controller memory to 1.5GB or more to provide capacity and headroom for the controller.
* Recommended browser: Chrome for better performance

#### Supported Platforms
* Officially supported linux distributions, Ubuntu, CentOS/Red Hat (Including all RHEL version e.g. 6/7/8), Debian, Rancher OS, CoreOS, AWS 'Bottlerocket', SUSE and Photon.
* Officially supported container management systems, Kubernetes, native docker, docker swarm, docker UCP/DataCenter, docker Cloud, Rancher, Google GKE/Kubernetes,  Amazon AWS ECS/EKS, Microsoft Azure AKS, VMware Photon, RedHat OpenShift 3.x,4.x, IBM IKS, SUSE CaaS.
* Docker run-time version: 1.9.0 and up; Docker API version: 1.21, CE and EE.
* Containerd and CRI-O run-times (requires changes to volume paths in sample yamls). See changes required for Containerd in the Kubernetes deployment section and CRI-O in the OpenShift deployment section.
* Console: Chrome or Firefox browser recommended. IE 11 not supported due to performance issues.

#### Not Supported
* Docker on Mac
* Docker on Windows
* Rkt (container linux) from CoreOS
* VMWare Integrated Containers (VIC) except in nested mode
* CloudFoundry
* Console: IE 11 not supported due to performance issues.
* Nested container host in a container tools used for simple testing. For example, deployment of a Kubernetes cluster using 'kind' https://kind.sigs.k8s.io/docs/user/configuration/.

Note 1: PKS is field tested and requires enabling privileged containers to the plan/tile, and changing the yaml hostPath as follows for Allinone, Controller, Enforcer:
<pre>
<code>      hostPath:
            path: /var/vcap/sys/run/docker/docker.sock</code>
</pre>

Note 2: NeuVector supports running on linux-based VMs on Mac/Windows using Vagrant, VirtualBox, VMware or other virtualized environments.

#### Performance and Scaling
As always, performance planning for NeuVector containers will depend on several factors, including:
+ (Controller) Number and size of images in registry to be scanned (by Controller) initially
+ (Enforcer) Services mode (Discover, Monitor, Protect), where Protect mode runs as an inline firewall 
+ (Enforcer) Type of network connections for workloads in Protect mode

In Monitor mode (network filtering similar to a mirror/tap), there is no performance impact and the Enforcer handles traffic at line speed, generating alerts as needed. In Protect mode (inline firewall), the Enforcer requires CPU and memory to filter connections with deep packet inspection and hold them to determine whether they should be blocked/dropped. Generally, with 1GB of memory and a shared CPU, the Enforcer should be able to handle most environments while in Protect mode. 

For throughput or latency sensitive environments, additional memory and/or a dedicated CPU core can be allocated to the NeuVector Enforcer container.

For performance tuning of the Controller for registry scanning, see System Requirements above.

##### Throughput
As the chart below shows, basic throughput benchmark tests showed a maximum throughput of 1.3 Gbps PER NODE on a small public cloud instance with 4 CPU cores. For example, a 10 node cluster would then be able to handle a maximum of 13 Gbps of throughput for the entire cluster for services in Protect mode. 

![Throughput](throughput.png)

This throughput would be projected to scale up as dedicated a CPU is assigned to the Enforcer, or the CPU speed changes, and/or additional memory is allocated. Again, the scaling will be dependent on the type of network/application traffic of the workloads.

##### Latency 
Latency is another performance metric which depends on the type of network connections. Similar to throughput, latency is not affected in Monitor mode, only for services in Protect (inline firewall) mode. Small packets or simple/fast services will generate a higher latency by NeuVector as a percentage, while larger packets or services requiring complex processing will show a lower percentage of added latency by the NeuVector enforcer.

The table below shows the average latency of 2-10% benchmarked using the Redis benchmark tool. The Redis Benchmark uses fairly small packets, so the the latency with larger packets would expected to be lower.

| Test | Monitor | Protect | Latency |
| ------ | ----------- | -------------- | ------- |
| PING_INLINE | 34,904 | 31,603 | 9.46% |
| SET | 38,618 | 36,157 | 6.37% |
| GET | 36,055 | 35,184 | 2.42% |
| LPUSH | 39,853 | 35,994 | 9.68% |
| RPUSH | 37,685 | 36,010 | 4.45% |
| LPUSH (LRANGE Benchmark) | 37,399 | 35,220 | 5.83% |
| LRANGE_100 | 25,539 | 23,906 | 6.39% |
| LRANGE_300 | 13,082 | 12,277 | 6.15% |


The benchmark above shows average TPS of Protect mode versus Monitor mode, and the latency added for Protect mode for several tests in the benchmark. The main way to lower the actual latency (microseconds) in Protect mode is to run on a system with a faster CPU. You can find more details on this open source Redis benchmark tool at https://redis.io/topics/benchmarks.



