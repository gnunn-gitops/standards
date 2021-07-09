# GitOps Folder Structure

## Introduction

Approximately 8 months ago I began my GitOps journey with kustomize followed shortly thereafter with adding ArgoCD after being introduced to it by my colleague Andrew Pitt. When I started working with kustomize one thing I looked for was a set of standards around practices and folder layout in order to apply some consistency to the work I was doing. Unfortunately I didn't find much beyond overly simplistic suggestions that didn't really work, at least for me, once I scaled them out to more real world situations.

As a result I ended up crafting my own set of standards which I'm still using to this day and I thought I'd share it more broadly. Now I am absolutely not saying this is the end all, be all of standards. I think standards are very dependent on the nature of the applications, the organizational structure, the development methodology being used, etc. I don't think there is a standard that is right for everyone, so in this case this is something that works for me and could maybe be useful to others as a data point when crafting there own standards.

As an aside, my wife knows I have a love of laptop bags because I always feel like each bag is a little better then one before in terms of structure, layout, pockets, etc. My standards here are pretty much the same, I reserve the right to change it if I find something better. I would also love to get input from others in terms of what works for them, so feel free to comment below.

## Principles

When formulating GitOps standards, it is very important to establish a set of principles as a baseline that all other standards must follow and adhere too. The set of principles being followed here include:

* __Do__ separate code (i.e. java, python, etc) from manifests (i.e. yaml) into different repos.
* __Do__ minimize yaml duplication, no copy paste
* __Do__ support two axis of configuration: clusters and environments (prod, test, dev, etc)
* __Do__ be able to split individual cluster and environment configurations into separate repos as needed to support existing organizational practices (i.e. separate production configuration in a different repo from test and dev being the most common example)
* __Do__ prefer a multi-folder and/OR multi-repo structure over multi-branch, i.e do not use branching to hold different sets of files (i.e. dev in one branch, test in another). This does not preclude the use of branches for features, this is stating that there should not be permanent branches for clusters or environments.
* __Do__ minimize specific gitops tool (ArgoCD, ACM, etc) dependencies as much as possible
* __Do__ put dependent applications manifests in the same manifests repo when managed by the same team. A microservice or 3 tier app that is composed of multiple deployments and managed by the same team would likely be in the same repo. __Do not__ put independent applications or applications managed by different teams in the same repo.

## More On Why Not Environment Branches?

So in gitops you sometimes see organizations using *permanent* branches to represent different environments. In these cases you have a dev branch for the dev environment, a test branch for the test environment, etc.

This often seems like an ideal way to do things, promoting between environments simply comes a matter of merging from lower environment branches to higher environment branches. However in practice it can be quite challenging for the following reasons:

* There are often many files that are environment specific and should either not be merged between environments or need to be named uniquely to avoid collisions
* Typically the 1:1 branch to environment works best when the manifests are identical across all branches, tools like kustomize do not fit into this pattern
* In a microservices world, a one branch per environment will quickly lead to an explosion of branches which again becomes difficult and cumbersome to maintain
* Difficult to have a unified view of cluster state across all environments since the state is stored in separate branches.

So in short I personally much prefer a single branch style with multiple folders to represent environments and clusters as we will see below.

Obviously this does not preclude using branches for updates, PRs, etc but these branches should be short lived, temporary artifacts to support development and not permanent fixtures.

## Assumptions

* As per the [tools standards](https://github.com/gnunn-gitops/standards/blob/master/tools.md), this folder structure is heavily dependent on [kustomize](https://kustomize.io). No thought is given to Helm or other alternatives at this time.
* A lesser used feature of kustomize is its ability to leverage remote repos, i.e. specify a base or overlay from a separate repo. This can be used to isolate environmental configurations in separate repos without duplicating yaml. You can read more about this feature [here](https://github.com/kubernetes-sigs/kustomize/blob/master/examples/remoteBuild.md).
* While the initial structure was focused on Application use cases, I've found it to work well for cluster configuration use cases as well.
* My focus is on [OpenShift](https://www.openshift.com/), I have not vetted anything in this document for other kubernetes distributions however I expect this would work similarly across any distribution.

## Repository Organization

I am a fan of having or or more catalog repositories to hold common elements that will be re-used across teams and other respositories. You can see this in action with the [Red Hat Canada Catalog](https://github.com/redhat-canada-gitops/catalog) repository where colleagues and I maintain a common set of components that we re-use across our individual repositories.

This is made possible by a great but under-utilized feature of kustomize that enables it to reference remote repositories as a base or resource and then patch it as needed to meet your specific requirements. The key to making this work successfuly is to ensure that when you reference the common repository you do so via a tag or commit ID. Not doing this means any time there is an update to the common repo you will automatically get that change. Using a tag or commit ID means you control when newer versions are brought in for application.

<b>Note:</b> I realize in my repos I'm very bad at following the practice of using a tag/commit ID when referencing remote repos :)

While in Red Hat Canada we have one repository that covers everything, in many organizations it will be typical to have a few different common repositories maintained by different teams. For example, the operations team may have a common rtepository for cluster configuration whereas the application architects may maintain a common repository for application components (Nexus, Sonarqube, frameworks, etc).

For Application repositories. in general you should align your repositories along application and team boundaries. For example, if I have an application that consists of a set of microservices where team A manages one microservice and team B manages a different one then this is best done as two different repositories in my opinion. If a team is maintaining multiple applications then again this is likely different repositories, one for each application.

For cluster configuration repositories, I would lean towards having different repositories for each cluster. While for me I'm using a single repo I'm also maintaining a very small set of clusters as well. In most organizations where





## Folder Layout

<table>
    <colgroup>
        <col />
        <col nowrap />
        <col />
    </colgroup>
    <tr>
        <th></th>
        <th>Folder</th>
        <th>Comments</th>
    </tr>
    <tr>
        <td align="right" valign="top">0</td>
        <td valign="top">├bootstrap</td>
        <td valign="top">
            <ul>
                <li>The minimal yaml required to bootstrap the entity into a cluster. This entity could be an application, cluster configuration or something else.</li>
                <li><em>Generally</em> this will be an Argo CD App of Apps, an ApplicationSet or something of that nature that will in turn load the rest of the components.</li>
            </ul>
        </td>
    </tr>
    <tr>
        <td align="right" valign="top">0</td>
        <td valign="top">├components</td>
        <td valign="top">
            <ul>
                <li>Provides yaml for all required apps, pipelines, and gitops tools.</li>
                <li><em>Generally</em> I prefer to have no buildconfigs (i.e. for s2i) in app folders, those should be in tekton or jenkins pipelines folder</li>
                <li>Avoid creating namespaces in components, this should be done in <em>environments</em> or <em>clusters</em> folders.
            </ul>
        </td>
    </tr>
    <tr>
        <td align="right" valign="top">1</td>
        <td valign="top">├──apps</td>
        <td valign="top">
            <ul>
                <li>No distinction between apps (i.e. code we write) versus services (databases, messaging systems, etc). It’s an artificial distinction IMHO</li>
                <li>Having said that, if your organization feels strongly about services, have peer folder to apps called services</li>
            </ul>
        </td>
    </tr>
    <tr>
        <td align="right" valign="top">2</td>
        <td valign="top">├────{appname1}</td>
        <td valign="top">
        </td>
    </tr>
    <tr>
        <td align="right" valign="top">3</td>
        <td valign="top">├──────base</td>
        <td valign="top">
        </td>
    </tr>
    <tr>
        <td align="right" valign="top">3</td>
        <td valign="top">├──────overlays</td>
        <td valign="top">
          <ul>
          <li>Overlays related to variations (i.e. HA versus non-HA) as well as environment oriented overlays. Cluster specific overlays should not be here, instead they will be in the `/clusters` folder.</li>
        </td>
    </tr>
    <tr>
        <td align="right" valign="top">2</td>
        <td valign="top">├────{appname2}</td>
        <td valign="top">
        </td>
    </tr>
    <tr>
        <td align="right" valign="top">3</td>
        <td valign="top">├──────base</td>
        <td valign="top">
        </td>
    </tr>
    <tr>
        <td align="right" valign="top">1</td>
        <td valign="top">├──tekton</td>
        <td valign="top">
        </td>
    </tr>
    <tr>
        <td align="right" valign="top">2</td>
        <td valign="top">├────pipelines</td>
        <td valign="top">
        </td>
    </tr>
    <tr>
        <td align="right" valign="top">3</td>
        <td valign="top">├──────{pipeline1}/base</td>
        <td valign="top">
            <ul>
                <li>Includes pipeline plus all artifacts needed for pipeline not called out in separate folders. Items like PVCs for workspaces, buildconfigs used by the pipeline, etc go here</li>
                <li>Pipelines may but do not need to map 1:1 to apps</li>
            </ul>
        </td>
    </tr>
    <tr>
        <td align="right" valign="top">3</td>
        <td valign="top">├──────{pipeline2}/base</td>
        <td valign="top">
        </td>
    </tr>
    <tr>
        <td align="right" valign="top">2</td>
        <td valign="top">├────pipelineruns</td>
        <td valign="top">
        </td>
    </tr>
    <tr>
        <td align="right" valign="top">3</td>
        <td valign="top">├──────{pipelinerun1}/base</td>
        <td valign="top">
        </td>
    </tr>
    <tr>
        <td align="right" valign="top">3</td>
        <td valign="top">├──────{pipelinerun2}/base</td>
        <td valign="top">
        </td>
    </tr>
    <tr>
        <td align="right" valign="top">2</td>
        <td valign="top">├────tasks</td>
        <td valign="top">
            <ul>
                <li>All custom tasks go here, no distinction between shared tasks and pipeline specific tasks. Today’s pipeline specific task is tomorrow’s shared task</li>
            </ul>
        </td>
    </tr>
    <tr>
        <td align="right" valign="top">3</td>
        <td valign="top">├──────base</td>
        <td valign="top">
        </td>
    </tr>
    <tr>
        <td align="right" valign="top">2</td>
        <td valign="top">├────jenkins</td>
        <td valign="top">
            <ul>
                <li>If you use Jenkins, I haven't spent much time on Jenkins layout since I'm mostly using tekton these days</li>
            </ul>
        </td>
    </tr>
    <tr>
        <td align="right" valign="top">3</td>
        <td valign="top">├──────pipelines</td>
        <td valign="top">
        </td>
    </tr>
    <tr>
        <td align="right" valign="top">1</td>
        <td valign="top">├──argocd</td>
        <td valign="top">
            <ul>
                <li>An optional folder for argocd applicationsthat may be required over and above what is in the bootstrap folder. Most commonly needed when using the App of App pattern with individually defined applications rather then an applicationset.</li>
                <li>In general, put things here if you find you are duplicating argo cd manifests in the bootstrap folder</li>
            </ul>
        </td>
    </tr>
    <tr>
        <td align="right" valign="top">0</td>
        <td valign="top">├environments</td>
        <td valign="top">
            <ul>
                <li>Optional folder that provides additional environment specific kustomization that is separate and distinct from cluster configuration. While commponents can have environment specific overlays sometimes you need to aggregate different components for a complete application. I like doing this here versus coming up with something artificial in components.</li>
                <li>I also find this useful to support demos which is not a typical use case. As I tend to deploy the same environments to multiple clusters to support these demos having an environments folder makes sense.</li>
                <li><b>Must</b> Inherit from <i>components</i> only, absolutely not permitted to inherit from <i>clusters</i></li>
                <li>When aggregating multiple components namespaces should be created here not in component overlays.</li>
            </ul>
        </td>
    </tr>
    <tr>
        <td align="right" valign="top">1</td>
        <td valign="top">├──overlays</td>
        <td valign="top">
            <ul>
                <li>Note that names of overlays are arbritrary, use what works for you.</li>
        </td>
    </tr>
    <tr>
        <td align="right" valign="top">2</td>
        <td valign="top">├────dev</td>
        <td valign="top"></td>
    </tr>
    <tr>
        <td align="right" valign="top">2</td>
        <td valign="top">├────test</td>
        <td valign="top"></td>
    </tr>
    <tr>
        <td align="right" valign="top">2</td>
        <td valign="top">├────prod</td>
        <td valign="top"></td>
    </tr>
    <tr>
        <td align="right" valign="top">2</td>
        <td valign="top">├────cicd</td>
        <td valign="top"></td>
    </tr>
    <tr>
        <td align="right" valign="top">2</td>
        <td valign="top">├────tools</td>
        <td valign="top"></td>
    </tr>
    <tr>
        <td align="right" valign="top">0</td>
        <td valign="top">├clusters</td>
        <td valign="top">
            <ul>
                <li>Cluster specific overlays go here</li>
                <li>Clusters can inherit from <i>environments</i> and <i>components</i></li>
            </ul>
        </td>
    </tr>
    <tr>
        <td align="right" valign="top">1</td>
        <td valign="top">├──overlays</td>
        <td valign="top"></td>
    </tr>
    <tr>
        <td align="right" valign="top">2</td>
        <td valign="top">├────{cluster1}</td>
        <td valign="top"></td>
    </tr>
    <tr>
        <td align="right" valign="top">3</td>
        <td valign="top">├──────dev</td>
        <td valign="top"></td>
    </tr>
    <tr>
        <td align="right" valign="top">3</td>
        <td valign="top">├──────test</td>
        <td valign="top"></td>
    </tr>
    <tr>
        <td align="right" valign="top">3</td>
        <td valign="top">├──────cicd</td>
        <td valign="top"></td>
    </tr>
    <tr>
        <td align="right" valign="top">3</td>
        <td valign="top">├──────tools</td>
        <td valign="top"></td>
    </tr>
    <tr>
        <td align="right" valign="top">2</td>
        <td valign="top">├────{cluster2}</td>
        <td valign="top"></td>
    </tr>
    <tr>
        <td align="right" valign="top">3</td>
        <td valign="top">├──────prod</td>
        <td valign="top"></td>
    </tr>
    <tr>
        <td align="right" valign="top">0</td>
        <td valign="top">├tenants/{team}</td>
        <td valign="top">
            <ul>
                <li>Only applicable to cluster configuration situations, tenants represent the different teams/applications/etc in a multi-tenant cluster.</li>
                <li>This is where you define cluster level resources (namespaces, quotas, limitranges, operators to install, etc) that are needed by the teams to do their work but typically require cluster-admin rights to provision</li>
            </ul>
        </td>
    </tr>


</table>

## Promoting Manifest Changes

A key question in any gitops scenario is how to manage promotion of changes in manifests between different environments and clusters. This process is heavily dependent on the structure and processes of the organization, however it is possible to define some basic characteristics that we are looking for as follows:

* Since every overlay depends on the base manifests, every change in the manifests needs to flow through the environments in hierarchical order, i.e. (dev > test > prod). We do not want a change to a base flowing to all environments simultaneously.
* To prevent changes in manifests flowing directly to environments, the state of environments and clusters needs to be pinned in git (i.e. commit revision or tag).
* Changes to environments/clusters can flow directly to the target environment. i.e. a direct change to the prod overlay can flow directly to prod without a promotion process. However given the structure of our repo these direct changes should be rare (i.e. prod specific secrets, etc) and limited to emergencies.

So as stated above, we need to tie specific environments to specific revisions so that changes in the repo can be promoted in a controlled manner following a proper SDLC process. Both the various GitOps tools (ArgoCD, Flux, ACM, etc) and Kustomize support referencing specific commits in a repo, as a result there are various options we have for managing environment promotions.

Note that using branches is implicit in these options but not discussed directly. As per the Why Not Branches section above, the intent is for short-lived branches to be created for revisions and merged back to trunk. So managing revisions is really about tracking the appropriate revision in trunk.

#### Option 1 - Manage revisions in the GitOps Tool

In this option we deploy each environment as an independent entity in the GitOps tool and tie each environment to a specific revision. So in ArgoCD, we would tie the application object for the Dev environment to one revision, the Test environment to another revision, etc.

When we are ready to promote a change, we simply update the revision in the GitOps tool to reference the appropriate revision or tag in the repo.

This is simple to do however it does require active management of the GitOps tools entities.

#### Option 2 - Manage revisions in kustomize

In this option each environment we deploy uses kustomize to manage the git revision. By default you tie kustomize to the local directory, i.e:

```
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

namespace: product-catalog-dev

bases:
- ../../../manifests/app/database/base
- ../../../manifests/app/server/base
- ../../../manifests/app/client/base
```

However kustomize supports remote references so you can also reference the bases remotely with specific git revisions:

```
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

namespace: product-catalog-dev

bases:
- https://github.com/gnunn-gitops/product-catalog/manifests/app/database/base?ref=9769ad7
- https://github.com/gnunn-gitops/product-catalog/manifests/app/server/base?ref=9769ad7
- https://github.com/gnunn-gitops/product-catalog/manifests/app/client/base?ref=9769ad7
```

In this way we can promote changes across multiple environments by simply updating the reference accordingly.

This approach provides very explicit control over the bases at the slight cost of having kustomize perform additional clones of the repo.

#### Option 3 - Hybrid (Do Both)

In this option we combine Options #1 and #2 for maximum control.

#### Recommendation

I don't have strong feelings at this point but my personal leaning is towards Hybrid.

## Examples

Here are a couple of repositories where you can see this standard in action:

* [Product Catalog](https://github.com/gnunn-gitops/product-catalog). This is a three tier application (front-end, back-end and database) deployed using GitOps with ArgoCD (or ACM) and kustomize. It deploys three separate environments (dev, test and prod) along wth Tekton pipelines to build the front-end and back-end applications. It also deploys a grafana instance for application monitoring that ties into OpenShift's [user defined monitoring](https://docs.openshift.com/container-platform/4.6/monitoring/enabling-monitoring-for-user-defined-projects.html).
* [Cluster Configuration](https://github.com/gnunn-gitops/cluster-config). This repo shows how I configure my OpenShift clusters using GitOps with ArgoCD. It configures a number of things including certificates, authentication, default operators, console customizations, storage and more.

I also highly recommend checking out the [Red Hat Canada GitOps](https://github.com/redhat-canada-gitops) organization as well. These repos include a default installation of the excellent [ArgoCD](https://github.com/redhat-canada-gitops/argocd) operator as well as a [catalog](https://github.com/redhat-canada-gitops/catalog) of tools and applications deployed with kustomize.

## Acknowledgements

I'd like to thank Andrew Pitt who has led the way on lot of the GitOps stuff in our group, he built the ArgoCD installation above as well as a substantial portion of the catalog items.