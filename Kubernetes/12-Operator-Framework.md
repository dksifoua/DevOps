# Operator Framework

The custom resource definition and the custom controller can be package together to be deployed as a single entity using
 the operator framework.

So by deploying the flight operator, it internally creates the crd and the resources and also deploys the custom 
controller as a deployment. Now th operator does much more than just deploying these two components.

All the operator are available at the [OperatorHub](https://operatorhub.io) with all the steps to deploy them.