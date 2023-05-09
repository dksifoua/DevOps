# API Versions

Everything under the api are the api groups such as `/apps`, `/extensions`, `networking.k8s.io`. Each api group has 
different  versions. When an api group is at `/v1` version, that means it's a GA (generally available) stable version. 
The api groups may have other versions such as `beta` or `alpha` as `/v1alpha1` or `/v1beta1` respectively .

The `alpha` version is when the api is first developed and merged to the k8s code base and become part of the k8s 
release for the very first time. At this stage, the api version has alpha in its name indicating that it's an alpha 
release. This api group is not enable by default. The alpha version may lack end-to-end tests and may have bugs as such 
it is not very reliable. Also, there's no guarantee that will be available in the future releases. It may be dropped 
without any notice in future releases. As such, his is only really for expert users who're interested in testing and 
giving early feedback for the api group.

After some time the alpha version and once all the major bug are fixed, and it has end-to-end tests, it moves to the 
beta stage. This is where the api version gets beta in name. The api groups in beta stage are enabled by default. It is 
GA, but it may still have minor bugs. However, there is commitment from the project that it may be moved to GA in the 
future.

After being in beta stage for a few months and with a few releases, api groups move to the GA or stable version. This is
 where the version number no longer has alpha or beta in it. Instead, it's just v1. It's of course enabled by default in
 the api group, and it's part of conformance tests and has all the tests written. Now, since most bugs are fixed, these 
api groups are highly reliable and they will be supported and present in many feature releases to come. And of course, 
this is when it's available to all users.

An api group can support multiple versions at the same time. For example, the `/apps` group can have `/v1`, `/v1alpha1`,
 `/v1beta1` all at the same time. This would mean that we will be able to create the same object using any of these 
version in our yaml file.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: [deployment-name]
spec:
  ...
```

```yaml
apiVersion: apps/v1alpha1
kind: Deployment
metadata:
  name: [deployment-name]
spec:
  ...
```

```yaml
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: [deployment-name]
spec:
  ...
```

Even though there are multiple versions supported at the same time, only one can be the preferred or storage version. It
 is that preferred version that is used when we retrieve objects from k8s using kubectl get command. The storage version
 is the version in which an object is stored in etcd, respected of the version used in the yaml definition file.

```shell
$ kubectl explain deployment
KIND:     Deployment
VERSION:  apps/v1 <- preferred api version
```

Only one version can be the preferred or storage version. Usually they are the same, bu they can be different.

`$ curl https://[kube-apiserver]:[port]/apis/batch`

As of now, it is not possible to see which is the storage version of a particular api through an api or a command. The 
one way to find that out is by looking at the stored object in etcd itself by querying the etcd database library.

```shell
$ ETCDCTL_API=3 etcdctl
  --endpoints=https://[kube-apiserver]:[port]
  --cacert=
  --cert
  --key=
  get "/registry/deployments/[namespace]/[deployment-name]"
```

To enable or disable a specific version, we must add it to the runtime config param of the kube apiserver service.

```shell
$ ExecStart=/usr/local/bin/kube-apiserver \\
  --runtime-config=batch/v2alpha1,...
  ...
```