# Custom Resource Definitions - CRD

When we create a deployment, k8s stores its information in the etcd data store but also creates pods equal to the number
of replicas defined in the deployment definition file. So who or what is responsible for that? That's the job of the 
controller; in this case the deployment controller. We don't have to create the controller because a deployment 
controller is built-in in k8s and is already available. So, the controller is a process that runs in the background and 
its job is to continuously monitor the status of resources that it's supposed to manage; in this case the deployment 
that we created. And when we create, update, or delete the deployment, it makes the necessary changes on the k8s cluster
 to match what we have done. In this case, when we create a deployment, the deployment controller creates a replica set 
which in turn creates as many as pods as we have specified in the deployment definition file. So that's the job of the a
controller.

For each type of resources in k8s (pod, replica set, deployment, service, volume, etc.), there's a controller to manage 
the resources created by the user.

K8s offers us the possibility to create custom resources (they can literally be anything). Let's say we want to create a
flight ticket object.

We create the definition file.

```yaml
# flight-ticket.yaml
apiVersion: flights.com/v1
kind: FlightTicket
metadata:
  name: my-flight-ticket
spec:
  from: Toronto
  to: Vancouver
  number: 2
```

We can create, retrieve, or delete the flight ticket object.

```shell
$ kubectl create -f flight-ticket.yaml
$ kubectl get flighttickets
$ kubectl delete -f flight-ticket.yaml
```

These are going to create or delete the flight ticket object in the etcd data store. But it's not actually going to book
 a flight ticket, is it? We want this to actually goes out and book a flight ticket for real. Say for instance, there's 
an api available at `bookflight.com/api` that we can call to book a flight ticket. So how do we call this api whenever 
we create a flight ticket object? For that, we are going to need a flight ticket controller.

It might look something like this in code.

```golang
// flightticket_controller.go
package flightticket

var controllerKind = apps.SchemaGroupVersion.WithKind("FightTicket")

// Code hidden

// Run begin watching and syncing
func (dc *FlightTicketController) Run(Workers int, stopCh <-chan struct {})

// Code hidden

// Call BookFlightAPI
func (dc *FlightTicketController) CallBookFlightAPI(obj interface {})

// A lot of code hidden
```

That's how it works at a high level. The flight ticket that we created is a **custom resource** and the controller that we 
wrote to book the actual flight ticket by calling the api is called a **custom controller**. Now we have a custom 
resource and a custom controller.

If we try to create the flight ticket resource now, it will fail with an error message `no matches for kind 
"FightTicket" in version "flights.com/v1"`. This is because we can't simply create any resource that we want without 
configuring it in the k8s api, without first telling k8s that it should allow us to create a flight ticket object. So we
 have to first define what that resource is that we want to create. For that, we need what is known as a **Custom 
Resource Definition - CRD**. We are going to use crd to tell k8s that we would like to create objects of kind 
FightTicket.

```yaml
# flightticket-custom-definition.yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: flighttickets.flights.com
spec:
  scope: Namespaced
  group: flights.com
  names:
    kind: FlightTicket
    singular: flightticket
    plural: flighttickets
    shortNames:
     - ft
    versions:
      - name: v1
        served: true
        storage: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          from:
            type: string
          to:
            type: string
          number:
            type: integer
            minimum: 1
            maximum: 10
```

We can now create the custom resource definition with `$ kubectl create -f flightticket-custom-definition.yaml` and once
 it is created, we can now create the flight ticket object with `$ kubectl create -f flight-ticket.yaml` without any 
error if the schema specified in the crd has been followed.

That all the first part of our problem: being able to create any kind of object type that we want on k8s by creating the
 corresponding crd. But it's only going to allow us to create these resources and store them in etcd data store. It's 
not actually going to do anything about these resources because we don't yet have a controller for it. 
