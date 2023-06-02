# Custom Controllers

We have created a flight ticket object and k8s stored its information in the etcd store. Now what we need to do is to 
monitor the status of the object in etcd store and perform actions such as making calls to the flight booking api, to 
book, edit, or cancel flight tickets. And that's why we need a custom controller.

A controller is any process or code that runs in a loop and is continuously monitoring the k8s cluster and listening to 
events of specific objects being charged, in this case, the flight ticket object. We could do that in any way we can. Of
 course, we need to write some code. So say we know python well, we could write a code in python that would query the 
k8s api server for objects in the k8s api. And then, I could watch for changes and make further calls to the api to make
 changes to the system. However, developing a controller in python can be challenging as the calls made to the apis may 
become expensive, and we will to create our own queuing and caching mechanisms. Developing in go with the k8s go client 
provides support for other libraries like shared informers that provide caching and queuing mechanisms that help to 
build controllers easily and in the right way.

So how do we start building a custom controller? There's a github repo named [sample-controller](https://github.com/kubernetes/sample-controller)
we have to clone and modify the controller `controller.go` with our custom code, then we build and run it.

```shell
$ git clone https://github.com/kubernetes/sample-controller.git
$ cd sample-controller
# Customize controller.go with our custom logic
$ go build -o sample-controller .
$ ./sample-controller -kubeconfig=$HOME/.kube/config
```

Once the controller is ready, we can decide on how to distribute it. We don't want to run and build it each time, so we 
may package the custom controller in a docker image and then choose to run it inside our k8s cluster as a pod or a 
deployment.  