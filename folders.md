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
    <tr>
        <th></th>
        <th>Folder</th>
        <th>Comments</th>
    </tr>
    <tr>
        <td align="right" valign="top">0</td>
        <td valign="top">manifests</td>
        <td valign="top">
            <ul valign="top">
                <li>Provides base yaml for all required apps, pipelines, and gitops tools.</li>
                <li>Only overlays that are needed to support environment independent variants are permitted. For example kustomizing an HA deployment versus single</li>
                <li><em>Generally</em> no buildconfigs (i.e. for s2i) in app folders, those should be in tekton or jenkins pipelines folder</li>
                <li>Controversial Any values that must be configured downstream (i.e. namespace references) should use kustomize features to avoid explicitly specifying them. In cases where it is not possible capitalized values should be used to prevent them slipping through accidentally</li>
            </ul>
        </td>
</table>
