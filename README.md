K8S-JenkinCICD
Jenkins CICD 备忘

* Jenkins部署在集群外，触发编译时，会在jenkins中创建一个 jnlp节点。
* 由于公司的ceph，仅仅是用了Rook的一个试点。所以我将jnlp节点，通过node-selector固定在特定节点。
* jdk，maven，kubectl 镜像都是用自己最小封装后的镜像
