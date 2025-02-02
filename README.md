## Contents:
1. [Overview](#1-overview)
2. [Pre-requisites](#2-pre-requisites)
3. [Upload images to IBM Cloud Container Registry](#3-upload-images-to-ibm-cloud-container-registry)
4. [Provide registry READ permission and create API Key](#4-provide-registry-read-permission-and-create-api-key)
5. [Review the Vulnerability Advisor report for the images](#5-review-the-vulnerability-advisor-report-for-the-images)
6. [Update the operator and bundle artifacts as per best practices guidelines](#6-update-the-operator-and-bundle-artifacts-as-per-best-practices-guidelines)
7. [Push the operator bundle to a GIT repository](#7-push-the-operator-bundle-to-a-git-repository)
8. Onboard to IBM Cloud Catalog
9. [Upgrade to new operator version](#9-upgrade-to-a-new-version)

****
## 1. Overview

This git repository contains a sample Node-Red Red Hat Certified Operator that helps you understand how to prepare your Operator and Operator bundle directory structure to be [Onboarded to IBM Cloud](https://cloud.ibm.com/docs/third-party?topic=third-party-operator-onboard-tutorial)

This guide will help you learn the best practices to be followed for preparing your operator bundle artifacts which is required while onboarding an operator to **IBM Cloud Catalog**.

Before you start, please check out the *Pre-requisites* section.

## 2. Pre-requisites

Prior to continuing, kindly ensure that you have the below accompanying stages ready.

1. You have an active IBM Cloud account.
2. Your Operator must be created using [*latest supported*](https://docs.openshift.com/container-platform/4.5/operators/operator_sdk/osdk-getting-started.html) Operator-SDK version . This will let you perform smooth onboarding of operator to **IBM Cloud Catalog**. 

Refer the sample [*node-red-operator*](https://github.com/IBM-Cloud/isv-operator-product-deploy-sample) for folder structure and other details.

If the above  you can proceed to the following sections to get information on the best practices for onboarding your operator.

****

## 3. Upload images to IBM Cloud Container Registry

A certified Operator would have followig images:

* Operator Image
* Application Container Image(s)
* Operator Bundle Image

To onboard your operator onto **IBM Cloud Catalog**, your **Application Image(s)** and **Operator Image** have to be uploaded to the ibmcloud container registry.

**NOTE**: It is required that the Operator Image is uploaded to a container Registry Namespace for which a **READ** API key can be created and shared during Operator Onboarding process. Please note that this requirement is ONLY for Operator Image.

Please follow the steps in this [Quick start](https://cloud.ibm.com/registry/start) guide to get started.


## 4. Provide registry READ permission and create API Key

For operator onboarding we will require a provider to set an **IAM serviceID** with _Container Registry_ access with **READ** permission for a namespace that contains the **Operator Image**, and create an API key for the same. 

Please follow the steps listed [here](https://cloud.ibm.com/docs/Registry?topic=Registry-iam).

**Quick Links**:
* [Getting Started with Registry](https://cloud.ibm.com/docs/Registry?topic=Registry-getting-started)
* [Create serviceid](https://cloud.ibm.com/docs/account?topic=account-serviceids#serviceids)
* [Give access to service id](https://cloud.ibm.com/docs/account?topic=account-assign-access-resources)
* [Create API keys](https://cloud.ibm.com/docs/account?topic=account-serviceidapikeys#create_service_key)

## 5. Review the Vulnerability Advisor report for the images

The images used by your operator should be free from all the vulnerabilities and security issues. 

You can view the **Security Status** of the uploaded Images on the IBM Cloud Registry. 

Vulnerability Advisor inspects your images to detect common deficiencies in certain security settings. [*Learn more*](https://cloud.ibm.com/docs/Registry?topic=va-va_index#app_configurations).

If the security status of your images show any issues, complete the corrective action for each issue shown in the report, rebuild the image and push it to the IBM Cloud container registry.


## 6. Update the operator and bundle artifacts as per best practices guidelines 

Before you begin with the best practices, make sure that you have generated your operator bundle using [*latest supported*](https://docs.openshift.com/container-platform/4.5/operators/operator_sdk/osdk-getting-started.html) operator sdk.

The folder structure of your operator bundle should look like the example given below for node-red-operator.

```
.
├── bundle
|   ├── manifests
|   |   ├── nodered.com_noderedbackups.yaml
|   │   ├── nodered.com_noderedrestores.yaml
|   │   │── nodered.com_nodereds.yaml
|   │   └── node-red-operator.clusterserviceversion.yaml
|   ├── metadata
|	    └── annotations.yaml
├── bundle.Dockerfile
```

Also , validate your operator bundle by executing the following command on the root directory of your project:

```execute
operator-sdk bundle validate ./bundle
```

Once you have validated your operator bundle, you're good to start with the Operator Bundle Artifact Check process.

**a) Check CRD version**

By default the operator-sdk creates the latest version of CustomResourceDefinition(CRD) `v1`. The older versions of OpenShift (4.5 and earlier) only support `v1beta1`. For backward support its recommended to change the version of CRD to `v1beta1`. 

See the [example CRD](https://github.com/IBM-Cloud/isv-operator-product-deploy-sample/blob/main/bundle/1.0.0/manifests/nodered.com_nodereds_crd.yaml) from Node Red Operator

**b) Check CSV fields**

There are a few important fields in your bundle CSV file that don't get generated by default. So, it is recommended that you make the required changes to your CSV file manually.

You need to add personalized data to the following required fields of the CSV.

| field                              | sub-field                                                    | Description                                                  |
| ---------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| metadata                           | name                                                         | This is the name of your CSV file. In general, the naming convention includes the name of the operator followed by the semantic version number, separated by a period (e.g. node-red-operator.v2.1.0). |
| metadata                           | namespace                                                    | This defines the Namespace in which the operator will be installed. It should be set as ```placeholder``` |
| annotations                        | alm-examples                                                 | This is an array containing CRD templates. The users of your Operator should be aware of mandatory and optional choices. You can upload the CRD templates containing a minimum set of configuration as an annotation in JSON format. |
| annotations                        | capabilities:                                                | Based on the operator maturity model, the capability can be classified as Basic Install or Seamless Upgrades or Full Lifecycle or Deep Insights or Auto Pilot |
| annotations                        | categories                                                   | A category is a comma separated string of applicable category names. |
| annotations                        | description                                                  | This defines a short description of the operator                  |
| annotations                        | createdAt                                                    | This indicates a rough (to the day) timestamp describing when the operator image was created |
| annotations                        | support                                                      | This contains the  name of the supporting vendor (eg: ExampleCo) |
| annotations                        | certified                                                    | (deprecated) This string has no effect, but can be set to "true" or "false" |
| annotations                        | repository                                                   | This is an optional URL of the operator's source code repository. If you don't want to specify , set it to NA. |
| spec                               | version                                                      | The value you enter in this field will depict the current version of the operator. |
| spec                               | replaces                                                     | This interprets as “the previous version to be replaced with a particular version". This field is very important for version upgrades to take place. |
| spec                               | customresourcedefinitions.owned                              | List all the CRDs for your operator in this section.         |
| spec                               | icon                                                         | This is typically a ```base64data``` encoded icon                              |
| spec                               | install                                                      | Contains details of Operator Deployment definition which is used to deploy the operator pod. |
| spec.install.spec.containers[].env | RELATED_IMAGE_ Environment variable in container definition of Operator Deployment | Ensure your Operator is updated to support Air-Gap feature. [*Learn more*](https://redhat-connect.gitbook.io/certified-operator-guide/appendix/offline-enabled-operators) |
| spec                               | serviceAccountName                                           | The name of the service account to which the RBAC is assigned. Operator gets the required permissions for its deployments and execution from this serviceaccount. |
| spec                               | clusterPermissions                                           | Define all the required cluster permissions for your operator's serviceAccount |
| spec                               | Permissions                                                  | Define all the required namespace level permissions for your operator's serviceAccount |
| spec                               | description                                                  | This  is mandatory field and shows the details about the Operator Description, its feature , how to use it, pre-requisites etc. A user of your operator will see this description and this is required. |
| spec                               | maintainers                                                  | It is the comma-separated list of maintainers and their emails (e.g. 'name1:email1, name2:email2') |
| spec                               | keywords                                                     | It contains comma-separated list of keywords for your operator |
| spec                               | provider                                                     | It is the provider's name for the operator                   |



**NOTE**: Ensure that the CSV references the Application Image(s) and Operator Image from IBM Cloud Registry.

Check the CSV for [*node-red-operator*](https://github.com/IBM-Cloud/isv-operator-product-deploy-sample/blob/main/bundle/1.0.0/manifests/node-red-operator.v1.0.0.clusterserviceversion.yaml) for more details.


**c) Check bundle.Dockerfile**

The `bundle.Dockerfile` in the root directory of your operator project is the Dockerfile for your metadata bundle image. 

Make sure the following 3 labels are present in the Dockerfile:

* `LABEL com.redhat.openshift.versions="v4.5,v4.6"` : This lists the OpenShift versions, starting with 4.5, that your operator will support. The value should with the letter 'v', followed by the number with no space in between.
* `LABEL com.redhat.delivery.operator.bundle=true` : This just needs to be there
* `LABEL com.redhat.deliver.backport=true` : This indicates the support provision for OpenShift versions before 4.5. If you don't specify this flag, your operator won't be listed in 4.4 or earlier.

As a best practice you may want to add the default channel and package name by adding the following labels for smooth onboarding to IBM cloud.

`LABEL operators.operatorframework.io.bundle.channel.default.v1=alpha`

`LABEL operators.operatorframework.io.bundle.package.v1=node-red-operator-certified`

For more details, refer the example   [*node-red-operator*](https://github.com/IBM-Cloud/isv-operator-product-deploy-sample/blob/main/bundle/bundle-v1.0.0.Dockerfile) 

**d) Check annotations.yaml**

For a smooth onboarding you need to have all the labels present in your `bundle.Dockerfile` contained in your `annotations.yaml`.

Make sure you add the package name properly.

`operators.operatorframework.io.bundle.package.v1: node-red-operator-certified`

See [*node-red-operator*](https://github.com/IBM-Cloud/isv-operator-product-deploy-sample/blob/main/bundle/1.0.0/metadata/annotations.yaml) example for more details.


## 7. Push the operator bundle to a GIT repository ##

If you have updated your Operator Bundle Artifacts as per the above instructions , make sure you validate your bundle once again using the following command. 

```execute
operator-sdk bundle validate ./bundle
```

If it produces any error or warning please ensure that you resolve all of them.

Upload to Git repository your operator bundle artifacts in one of the below required formats based on the release type.

* First Operator Bundle version Release (example 1.0.0):

```
.
├── bundle
|   ├── manifests
|   |   ├── nodered.com_noderedbackups.yaml
|   │   ├── nodered.com_noderedrestores.yaml
|   │   │── nodered.com_nodereds.yaml
|   │   └── node-red-operator.clusterserviceversion.yaml
|   ├── metadata
|	    └── annotations.yaml
├── bundle.Dockerfile
```

* Subsequent Operator Bundle version upgrades (example 2.0.0):

```
bundle
├── 1.0.0
│   ├── manifests
│   │   ├── nodered.com_noderedbackups_crd.yaml
│   │   ├── nodered.com_noderedrestores_crd.yaml
│   │   ├── nodered.com_nodereds_crd.yaml
│   │   └── node-red-operator.v1.0.0.clusterserviceversion.yaml
│   └── metadata
│       └── annotations.yaml
├── 2.0.0
│   ├── manifests
│   │   ├── nodered.com_noderedbackups_crd.yaml
│   │   ├── nodered.com_noderedrestores_crd.yaml
│   │   ├── nodered.com_nodereds_crd.yaml
│   │   └── node-red-operator.v2.0.0.clusterserviceversion.yaml
│   └── metadata
│       └── annotations.yaml
├── bundle-1.0.0.Dockerfile
└── bundle-2.0.0.Dockerfile

```

**NOTE:** While onboarding your operator to IBM Cloud Catalog , you would need to provide the URL of the ClusterServiceVersion(CSV) file residing in the git repository.

_Example CSV URL for Node Red Operator Release Version 1.0.0 would be_: https://github.com/IBM-Cloud/isv-operator-product-deploy-sample/blob/main/bundle/manifests/node-red-operator.v1.0.0.clusterserviceversion.yaml 


## 8. Onboard to IBM Cloud Catalog and validate

****
## 9. Upgrade to a new version ##

A new version of an operator can be released in any of the following scenarios:
a) Update the Product version used in the operator

b) Update in Operator code 

If you need to release a new version of the operator (say 2.0.0), while your present version is 1.0.0), follow the below procedure:

**a) Copy the bundle and modify the version**

Copy the folder contents of the previous version(example:1.0.0) to a new folder called 2.0.0, which will be the new release version.

```
├── 1.0.0
│   ├── manifests
│   │   ├── nodered.com_nodereds_crd.yaml
│   │   └── node-red-operator.v1.0.0.clusterserviceversion.yaml
│   ├── metadata
│       └── annotations.yaml   
├── bundle-1.0.0.Dockerfile
```

 Change the version number in the CSV file name. See the sample below

```
├── 2.0.0
│   ├── manifests
│   │   ├── nodered.com_nodereds_crd.yaml
│   │   └── node-red-operator.v2.0.0.clusterserviceversion.yaml
│   ├── metadata
│       └── annotations.yaml   
├── bundle-2.0.0.Dockerfile
```

Now you should have two folders representing two versions as shown below

```
├── 1.0.0
│   ├── manifests
│   │   ├── nodered.com_nodereds_crd.yaml
│   │   └── node-red-operator.v1.0.0.clusterserviceversion.yaml
│   ├── metadata
│       └── annotations.yaml  
├── 2.0.0
│   ├── manifests
│   │   ├── nodered.com_nodereds_crd.yaml
│   │   └── node-red-operator.v2.0.0.clusterserviceversion.yaml
│   ├── metadata
│       └── annotations.yaml  
├── bundle-1.0.0.Dockerfile
├── bundle-2.0.0.Dockerfile
```

**b) Make changes in the CSV file**

It is the most important step for any version upgrade.

To ensure that the upgraded contents are there inside  `clusterserviceversion.yaml`, the following modifications are required for the CSV file inside the newly created folder, 2.0.0.

  1) Update the `metadata.name` field of the CSV file with the new version(2.0.0). 

     ```
     name: node-red-operator.v2.0.0
     ```

  2) Update the `spec.version` field with the new version. 

     ```
     version: 2.0.0
     ```

  3) Update the `spec.replaces` field to indicate the previous version(1.0.0) of the CSV that is being upgraded to the new version(2.0.0). 

     ```
     replaces: node-red-operator.v1.0.0
     ```

  **NOTE:** Along with the above metadata updates to the CSV file, you will also need to update the new version of Operator Image and/or Application Image(s) based on the updates associated with the new release.



**c) Validate the operator bundle**

Once you have updated your operator bundle, validate your bundle using the following command. 

```execute
operator-sdk bundle validate ./2.0.0
```
If it shows any error or warning, please ensure that you resolve all of them. 

****

Back to [Onboarding Tutorial](https://test.cloud.ibm.com/docs/third-party?topic=third-party-operator-onboard-tutorial)

****

