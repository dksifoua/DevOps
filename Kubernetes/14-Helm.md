# Helm

## Introduction

K8s is awesome at managing complex infrastructures. We, humans tend to struggle with complexity. Though applications we 
deploy into our k8s cluster can become very complicated. A typical app is usually made up of a collection of objects 
that need to interconnect to make everything work. For example, even a relatively simple wordpress site might need the 
following: a deployment, a persistent volume, a persistent volume claim, a service, and a secret. And may be even more 
if we want extra stuff like periodic backups and so on. And for every object, we might need a separate yaml file. Then 
we to kubectl apply to every yaml file. Now, this can be a tedious task. But it's not the end of the world. Imagine we 
download these yaml files from the internet, we are happy with the default so start changing stuff. Whe have to open 
every yaml file and edit each one according to our needs. Well, not bad enough yet. Now imagine two months go by. We now
 have to upgrade some components in our app, and back to editing multiple yaml files with great care, so we don't change 
the wrong thing in the wrong place. We need to delete the app, we need to remember each object that belongs to our app 
and delete them all one by one.

Now, we might be thinking, it's not a big deal, we can just write all the object declarations in a single yaml file. 
Well,  that's true, but  it might make it even harder to find the stuff that we are looking for. We'd have to 
continuously search for stuff we need to edit in something that could be 25 pages of text. At least in multiple files, 
it might be somewhat organized.

Enter Helm.

Helm changes the paradigm. K8s doesn't really care about ou app as a whole. All that it knows is that we declared 
various objects, and it proceeds to make each of them exist in our cluster. It doesn't really know that pv, pvc, svc, 
etc. are all part of a big application called wordpress. It looks at all the little pieces that the admin wanted to have
in the cluster and takes care of each one individually.

Helm however, is built from the ground up to know about such stuff. That's why it's sometimes called a **package manager
** for k8s. It looks at those objects as part of a big package as a group, whenever we need to perform an action, we 
don't tell helm the objects it should touch. We just tell it what package we want it to act on like our wordpress 
package. And based on package name, it then knows what objects it should change and how, and even if there are hundreds 
of objects that belong to that package.


## Helm Concepts

**Templates**

**Values**

**Chart**
