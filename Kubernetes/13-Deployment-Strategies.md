# Deployment Strategies

There are two deployment strategies. The recreate strategy where a newer version is deployed by first destroying all the
 existing versions and then creating newer versions of the application instances. The problem with this approach is that
 during the period after the older versions are down and before any newer version is up, the application is down and 
inaccessible to users. The second strategy is the rolling update strategy where we do not destroy all of them at once. 
Instead, we take down the older version and bring up a newer version one by one. This way, the application never goes 
down and the upgrade is seamless. The rolling update is the default deployment strategy.

There are a couple of other strategies as well, which are not really strategies that can be specified as option in the 
deployment, but they can be implemented in a different way.

## Blue-Green Deployment Strategy

The blue-green strategy is a deployment strategy where we have the new version deployed alongside the old version. So 
the old version is called blue and the new version is called green. And 100% of the traffic is still routed to the old 
version. So at this point in time, tests are run on the new version, and once all tests passed, we switch traffic to
 the new version all at once.

## Canary Deployment Strategy

In this strategy, we deploy the new version and route only a small version percentage of traffic to it. So the majority 
of the traffic is still routed to the older version, but we have a small percentage routed to the new version. At this 
point, we run tests, and if everything looks good, we upgrade the original deployment with the newer version of the app.