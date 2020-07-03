# Gitops Folder Structure

## Introduction

## Principles

* __Do__ separate code (i.e. java, python, etc) from manifests (i.e. yaml) into different repos.
* __Do__ minimize yaml duplication, no copy paste
* __Do__ support two axis of configuration: clusters and environments (prod, test, dev, etc)
* __Do__ be able to split individual cluster and environment configurations into separate repos as needed to support existing organizational practices (i.e. separate production configuration in a different repo from test and dev being the most common example)
* __Do__ prefer a multi-folder and/OR multi-repo structure over multi-branch, i.e do not use branching to hold different sets of files (i.e. dev in one branch, test in another). This does not preclude the use of branches for features, this is stating that there should not be permanent branches for clusters or environments.
* __Do__ minimize specific gitops tool (ArgoCD, ACM, etc) dependencies as much as possible
* __Do__ put dependent applications manifests in the same manifests repo when managed by the same team. A microservice or 3 tier app that is composed of multiple deployments and managed by the same team would likely be in the same repo. __Do not__ put independent applications or applications managed by different teams in the same repo.

## Assumptions

* As per the tools standards, this folder structure is dependent on kustomize. No thought is given to Helm or other alternatives at this time.
* A lesser used feature of kustomize is its ability to leverage remote repos, i.e. specify a base or overlay from a separate repo. This can be used to isolate environmental configurations in separate repos without duplicating yaml

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
        <td valign="top">├manifests</td>
        <td valign="top">
            <ul>
                <li>Provides base yaml for all required apps, pipelines, and gitops tools.</li>
                <li>Only overlays that are needed to support environment independent variants are permitted. For example kustomizing an HA deployment versus single</li>
                <li><em>Generally</em> no buildconfigs (i.e. for s2i) in app folders, those should be in tekton or jenkins pipelines folder</li>
                <li>Controversial Any values that must be configured downstream (i.e. namespace references) should use kustomize features to avoid explicitly specifying them. In cases where it is not possible capitalized values should be used to prevent them slipping through accidentally</li>
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
          <li>In general overlays in manifests should be the exception not the norm. Variants of deployments are acceptable as overlays but more often then not overlays should be confined to environments.</li>
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
                <li>Includes pipeline plus all artifacts needed for pipeline not called out in separate folders. Things like PVCs for workspaces, buildconfigs used by the pipeline, etc go here</li>
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
        <td align="right" valign="top">2</td>
        <td valign="top">├────tools</td>
        <td valign="top">
            <ul>
                <li>Define gitops artifacts here (i.e. ArgoCD projects and applications, ACM subscriptions, channels, etc)</li>
                <li>Having separate named folders for tools is optional, if your org is committed to one tool feel free to drop yaml directly into tools</li>
            </ul>
        </td>
    </tr>
    <tr>
        <td align="right" valign="top">3</td>
        <td valign="top">├──────argocd/base</td>
        <td valign="top">
        </td>
    </tr>
    <tr>
        <td align="right" valign="top">3</td>
        <td valign="top">├──────acm/base</td>
        <td valign="top">
        </td>
    </tr>
    <tr>
        <td align="right" valign="top">0</td>
        <td valign="top">├environments</td>
        <td valign="top">
            <ul>
                <li><b>Must</b> Inherit from <i>manifests</i> only, absolutely not permitted to inherit from <i>clusters</i></li>
                <li>Provides environment specific kustomization that is separate and distinct from cluster configuration</li>
                <li>Can aggregate multiple bases from <i>manifests</i> to create complete application. For example, aggregate several microservices or an application and it's database.</li>
                <li>Namespaces should be created here</li>
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
                <li>Clusters can inherit from <i>environments</i> and <i>manifests</i></li>
                <li>No namespace creation or specification goes here</li>
                <li>Cluster specific rolebindings go here, since we do not allow namespace specification in kustomize here this avoids the whole issue with kustomize replacing namespaces in roles issue</li>
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

</table>
