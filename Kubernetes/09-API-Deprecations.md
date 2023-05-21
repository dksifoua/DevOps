# API Deprecations

A single api group can support multiple versions at a time. But why do we need multiple versions? And how many should we
 support? When can we remove an older version that is no longer required? These questions are answered by the api 
version deprecation policy.

Lets for example say that we are planning to contribute to the k8s project, so we create an api group called 
`/dksifoua.io` under which we have two resources called `/jenkins` and `/misc`. So we develop it in-house, and we test 
it, and we are ready to merge it to the k8s project. We raise a pull request and hopefully, it gets accepted, and we 
release  it as an alpha version. So we call it `/v1alpha1` because it's the first alpha version the v1 version.

```yaml
apiVersion: dksifoua.io/v1alpha1
kind: Jenkins
metadata:
  name: Jenkins
spec:
  ...
```

Now, let's say for instance, the misc object didn't go well with the users, and we decided to remove it. In the newt k8s 
release, can we just remove it from the v1alpha1 version? No. That's where the first rule of api deprecation policy 
comes into play. 

**API Deprecation Policy Rule #1:** Api elements may only be removed by incrementing the version of the api group. Which
means we can only remove the misc element from v1alpha2 version of the api group. It will continue to be present in the 
v1alpha1.

```yaml
apiVersion: dksifoua.io/v1alpha2
kind: Jenkins
metadata:
  name: Jenkins
spec:
  ...
```

At this point, the resource in the db is still at v1alpha1 but our api version has now changed into v1alpha2. So this 
is now going to be a problem. We will need to go back and change all the api versions in our yaml file from v1alpha1 to 
v1alpha2. Which is why the new release must support both v1alpha1 and v1alpha2. But the preferred or storage version 
could be v1alpha2. This means that the users can the same yaml file to create the resources, but internally, they would 
be converted and stored as v1alpha2.

That brings us to the second rule of api deprecation policy. 

**API Deprecation Policy Rule #2:** API objects must be able to round-trip between api versions in a given release 
without information loss, with the exception of whole rest resources that do not exist in some versions. If we create an
 object in the v1alpha1 version and it's converted to v1alpha2 and then back into v1alpha1, it should be the same as the 
 original v1alpha1 version.

So continue with our story, we fixed some more bugs and are ready for beta version. Our first beta version called 
`v1beta1` is now ready, and after a few months, we release the next beta version `v1beta2`, and finally, we release our 
stable version, the GA version, which we call as `v1`. So that's kind of how an api evolves over time. Starts with 
v1alpha1, then v1alpha2 and so on, and then to beta versions v1beta1, v1beta2, etc., and finally, the stable version v1.

We don't have to have all the versions available at all times. Of course, we must deprecate and remove all older 
versions as we release newer versions.

**API Deprecation Policy Rule #3:** An API version in a given track may not be deprecated in favor of a less stable API 
version.
- GA API versions can replace beta and alpha API versions.
- Beta API versions can replace earlier beta and alpha API versions, but may not replace GA API versions. 
- Alpha API versions can replace earlier alpha API versions, but may not replace GA or beta API versions.

**API Deprecation Policy Rule #4a:** API lifetime is determined by the API stability level
- GA API versions may be marked as deprecated, but must not be removed within a major version of Kubernetes 
- Beta API versions are deprecated no more than 9 months or 3 minor releases after introduction (whichever is longer), 
and are no longer served 9 months or 3 minor releases after deprecation (whichever is longer)
- Alpha API versions may be removed in any release without prior deprecation notice

This ensures beta API support covers the maximum supported version skew of 2 releases, and that APIs don't stagnate on 
unstable beta versions, accumulating production usage that will be disrupted when support for the beta API ends.

**API Deprecation Policy Rule #4b:** The "preferred" API version and the "storage version" for a given group may not 
advance until after a release has been made that supports both the new version and the previous version. Users must be 
able to upgrade to a new release of Kubernetes and then roll back to a previous release, without converting anything to 
the new API version or suffering breakages (unless they explicitly used features only available in the newer version). 
This is particularly evident in the stored representation of objects.

[See more on k8s docs](https://kubernetes.io/docs/reference/using-api/deprecation-policy/#:~:text=Rule%20%231%3A%20API%20elements%20may,significantly%20changed%2C%20regardless%20of%20track.)

## Kubectl Convert

As we have been seeing, when k8s clusters are upgraded, we have new apis being added and old one being deprecated and 
removed. When an old api is removed, it is important to note that we have to update our existing manifest files to new 
version (from v1beta1 to v1).

```shell
$ kubectl convert -f <old-file> --output-version <new-api>
```

Note that the kubectl convert command may not be available by default on the system, and that's because it's a separated
 plugin. So the kubectl convert is a special plugin that we have to install. The installation instructions can be found 
in the k8s documentation page, along with the instructions for installing the kubectl utility.