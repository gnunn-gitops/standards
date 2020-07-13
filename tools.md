# GitOps Tools

## Introduction

There are a variety of tools that can be used as part of a GitOps process. In addition to the GitOps tool itself (ArgoCD, Flux, RHACM, etc) there are a variety of ways to manage the manifest yaml for our applications including Ansible, OpenShift Templates, Helm and Kustomize.

In this document we will discuss them briefly and then provide some recommendations at the end of the document.

## Manifest Tools

There are a variety of tools to manage the yaml that will get deployed into your cluster. At a high level, here are the tools I typically see being used at my customers:

__Ansible__. Great tool for managing infrastructure as code across a wide variety of infrastructure. Includes a *k8s* module that enables Ansible to easily integrate with kubernetes and uses jinja2 templates for manifests.

While Ansible is great automation tool, I personally prefer to use native *k8s* tools where possible. Also like Helm it relies on templating which means you cannot directly apply yaml in your git repos when doing iterative development.

Finally none of the common GitOps tools support Ansible so it is really only an option if you using Ansible for your kubernetes GitOps which is generally not recommended. Absolutely use Ansible for infrastructure gitops (i.e. VMs, cluster provisioning, etc) but when dealing with kubernetes manifests there are better options.

__OpenShift Templates__. OpenShift templates pre-dates a lot of the tools on this list, it was released in OpenShift 3.0 when very few tools for managing managing manifests existed. OpenShift Templates are very easy to develop and work with, however like other templating solutions it's challenging to use in an iterative development process.

Similar to Ansible, none of the existing DevOps as far as I know support OpenShift Templates making it a non-starter.

__Helm__. Helm is a package manager for Kubernetes and OpenShift 4 now supports it natively. Helm provides charts which can be deployed into OpenShift using the Helm CLI or via the OpenShift web console. These charts contain templates along with values that can be substituted when the chart is deployed to control the yaml that is applied. A neat feature of the charts is that you can have a chart that is uses other charts as dependencies.

Personally I think Helm's sweet spot is as a package manager, i.e. similar to RPMs, and is overkill for most enterprise teams. Additionally charts can quickly become very complex and difficult to maintain for anyone other then the original author. Lastly it uses templates which again is challenging for iterative development but easier then other templating solutions with the capabilities that Helm provides.

Finally from an application/package manager perspective I feel like I'm getting that functionality out of my gitops tool and Helm is providing a lot of value add here.

All GitOps tools support working with Helm charts.

__jsonnet/ksonnet__. No experience with this but I do know some folks quite like this approach, however personally I've never seen it used at any of my customers.

Most GitOps tools support jsonnet/ksonnet.

__kustomize__. Kustomize is built into Kubernetes as of 1.14 which means there is full support for it in your kubernetes or OpenShift environment out of the box. Kustomize is not a templating framework, it is a patching framework that works through the concept of inheritance. _Bases_ define the base level yaml for an application and _overlays_ inherit from those bases or other overlays with patches to differentiate as needed.

Since kustomize doesn't use templating it's very easy to follow and understand what it is doing keeping initial complexity low. However because it relies on inheritance getting a structure that works for your team can be an involved process.

All GitOps tools support kustomize since it is built into kubernetes.

__Recommendation__. My personal recommendation is kustomize, it has worked extremely well for me so far and does what I need. All documents in this repo assume that kustomize is being used.

## GitOps Tools

There are a variety of GitOps tools available and I will not provide information or make a recommendation here simply because I do not have sufficient experience with them. I personally use ArgoCD and Red Hat's Advanced Cluster Management (ACM) and am satisfied with both. Disclaimer I am a Red Hat employee.

I do not have experience with Flux but it is a popular tool as well.