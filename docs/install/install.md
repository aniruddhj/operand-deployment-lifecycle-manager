<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Install the operand deployment lifecycle manager On OCP 4.2+](#install-the-operand-deployment-lifecycle-manager-on-ocp-42)
  - [1. Create OperatorSource](#1-create-operatorsource)
  - [2. Create a Namespace `ibm-common-services`](#2-create-a-namespace-ibm-common-services)
  - [3. Install operand deployment lifecycle manager](#3-install-operand-deployment-lifecycle-manager)
    - [Search ODLM Package in the OperatorHub](#search-odlm-package-in-the-operatorhub)
    - [Install ODLM Operator](#install-odlm-operator)
  - [4. Manage Other Operators with ODLM](#4-manage-other-operators-with-odlm)
    - [Create Operand Request](#create-operand-request)
    - [Enable or Delete an Operator](#enable-or-delete-an-operator)
- [Post-installation](#post-installation)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# Install the operand deployment lifecycle manager On OCP 4.2+

## 1. Create OperatorSource

Before install ODLM, this operator source should be created to get operator bundles from `quay.io`.

```yaml
apiVersion: operators.coreos.com/v1
kind: OperatorSource
metadata:
  name: opencloud-operators
  namespace: openshift-marketplace
spec:
  authorizationToken: {}
  displayName: IBMCS Operators
  endpoint: https://quay.io/cnr
  publisher: IBM
  registryNamespace: opencloudio
  type: appregistry
```

Click the plus button, and then copy the above operator source into the editor.
![Create OperatorSource](../images/create-operator-source.png)

Check if all the operator packages are loaded, run command:

```bash
# oc -n openshift-marketplace get operatorsource opencloud-operators -o jsonpath="{.status.packages}"
ibm-mongodb-operator-app,ibm-management-ingress-operator-app,ibm-cert-manager-operator-app,ibm-licensing-operator-app,cp4foobar-operator-app,ibm-metering-operator-app,ibm-auditlogging-operator-app,meta-operator-app,ibm-ingress-nginx-operator-app,ibm-iam-operator-app,ibm-healthcheck-operator-app,ibm-meta-operator-bridge-app,operand-deployment-lifecycle-manager-app,ibm-catalog-operator-app,ibm-commonui-operator-app
```

During development, we need to update the csv package frequently, but the operator source need a long time to sync the new package, my experience is to re-create this operator source, the new package will be loaded immediately.

```bash
oc delete -f opencloudio-source.yaml
oc apply -f opencloudio-source.yaml
```

## 2. Create a Namespace `ibm-common-services`

Open the `OperatorHub` page in OCP console left menu, then `Create Project`, e.g., create a project named `ibm-common-services`.

![Create Project](../images/create-project.png)

## 3. Install operand deployment lifecycle manager

### Search ODLM Package in the OperatorHub

Type `operand-deployment-lifecycle-manager` in the search box
![Search ODLM Package](../images/search-odlm.png)Open `OperatorHub` and search `operand-deployment-lifecycle-manager` to find the operator, and install it.

### Install ODLM Operator

![ODLM Install Preview](../images/search-install-odlm-preview.png)

Select the namespace `ibm-common-services` that created in step [Create Project](#create-project)
![Install ODLM Operator](../images/install-odlm.png)
![Installed ODLM](../images/install-odlm-success.png)

Waiting for about 1 minute, `OperandRegistry` and `OperandConfig` operand will be ready
![ODLM All Instances](../images/odlm-all-instances.png)

So far, the ODLM operator installs completed. Next, we can start to install other common service operators.

## 4. Manage Other Operators with ODLM

### Create Operand Request

Select `OperandRequest` from `Create New` button
![Create Operand Request](../images/create-operand-request.png)

Modify the `OperandRequest` to add the operator you want to install into `spec.requests.operands`
![Modify the Operand Request](../images/operand-request-detail.png)
![Operand Request Instance](../images/operand-request-create-done.png)

The list of operators you can add:

```bash
    - name: ibm-cert-manager-operator
    - name: ibm-mongodb-operator
    - name: ibm-iam-operator
    - name: ibm-monitoring-exporters-operator
    - name: ibm-monitoring-prometheusext-operator
    - name: ibm-healthcheck-operator
    - name: ibm-management-ingress-operator
    - name: ibm-ingress-nginx-operator
    - name: ibm-metering-operator
    - name: ibm-licensing-operator
    - name: ibm-commonui-operator
    - name: ibm-auditlogging-operator
    - name: ibm-catalog-operator
    - name: ibm-platform-api-operator
    - name: ibm-helm-api-operator
    - name: ibm-helm-repo-operator
    - name: ibm-management-repo-operator
```

After the `OperandRequest` created, we can click the left navigation tree `Installed Operators` to check if our common services install successfully.
![Installed Operators](../images/operator-list.png)

### Enable or Delete an Operator

- Enable an operator, you can add the operator into the `OperandRequest`

- Delete an operator, you can remove the operator from the`OperandRequest`

# Post-installation

The operators and their custom resource would be deployed in the cluster, and thus the installation of operands will also triggered by the CR.
