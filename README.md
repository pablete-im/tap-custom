# tap-custom
Customizations for Tap OOB Supply Chains and Deliveries

This project currently contains 2 customizations

 - A ClusterDelivery customization to send emails at the end of the Deployment process
 - A ClusterSupplyChain customization to run SonarQube analysis, evaluate the Quality Gate and send an email with the result of the scan. This customization creates a Cartographer Deliverable with a label to be selected by the previously described ClusterDelivery.
 - A ClusterSupplyChain customization to run SonarQube analysis, evaluate the Quality Gate, send an email with the result of the scan and create a ConfigMap with the Deliverable instead creating the Deliverable itself. The Deliverable inside the ConfigMap is then writen to a GitRepository by a git-writer resource to enable GitOps.

**NOTE:** When inspecting the source files, you'll need to replace the "<DEVELOPER NAMESPACE>" placeholders in the different yaml files and the placeholders in the secrets.

## Summary email sending at the end of the Delivery creation process

//TODO

While all the files in this repo _should_ work if you want to use them, the full ClusterDelivery file _MAY NOT_. This is because it was copied out of an existing TAP cluster and, thus, has pointers to some values specific to that install. 

Files that properly modify the ClusterDelivery:

 - **sendmail-secret.yaml**: Creates the secret that specifies the connection details to the SMTP server
 - **mail-sender-tekton-task.yaml**: Creates the Tekton task that sends the email and uses the previously described secret.
 - **mail-sender-tekton-pipeline.yaml**: Creates the Tekton pipeline that runs the previously described task.
 - **mail-sender-clusterDeploymentTemplate.yaml**: Creates the ClusterDeploymentTemplate template that creates the PipelineRun referencing the tekton pipeline.
 - **delivery-basic-mail-cd.yaml**: Creates the modified ClusterDelivery that contains the new resource to send the email and the new selector for the Deliverable CRD created by the SupplyChain.

## Sonarqube and Quality Gate Supply Chain Example

This repo is an example of how to add a new task, a Sonarqube scan, to a TAP SupplyChain. 

It Creates:
- a modified deliverable-template ClusterTemplate for the last step of the Supply Chain to create the Deliverable object with the proper tag to be selected by the new ClusterDelivery that sends an email.
- a (templated) Tekton PipelineRun in a new ClusterSourceTemplate
- a Tekton Pipeline
- a Tekton Task
- a secret for the SonarQube server connection details that is directly used by the Tekton Task.

And it wires those objects to a new Resource in a modified ClusterSupplyChain.

While all the files in this repo _should_ work if you want to use them, the full ClusterSupplyChain and eliverable-template-mail ClusterTemplate files _WILL NOT_. This is because it was copied out of an existing TAP cluster and, thus, has pointers to some values specific to that install (e.g. the URL t the container registry). 

Put more plainly, this repo exists for educational purposes and to provide a starting point for people new to Cartographer.

**NOTE:** If you want to use this example on your cluster, you will need an existing SonarQube server installed somewhere. The Helm charts are an easy place to start.

If you'd like to implement this on your cluster, you'll just need to extract the ClusterSupplyChain and deliverable-template ClusterTemplate from your cluster and make edits. As an example, here is how to export the scanning-testing SupplyChain, which is what this repo started with.

```
kubectl get csc source-test-scan-to-url -o yaml > testing-scanning-sonar-csc.yml

k get ct deliverable-template -o yaml > deliverable-template-mail-ct.yaml
```

### How to Read

//TODO

If you're new to Cartographer and supply chains, it is not immediately obvious what files run in what order. To help, consider reading in the following order.

### [testing-scanning-sonar-csc.yml](testing-scanning-sonar-csc.yml)

This is the top-level file and where the adventure begins. There are minor comments to show what we changed from the original.

### [sonar-clusterSourceTemplate.yml](sonar-clusterSourceTemplate.yml)

In the SupplyChain, we added a new resource, `sonarqube-scan`. That resource has to reference a ClusterTemplate of some sort. Because we need to  emit a source URL, we chose a `ClusterSourceTemplate`. This file is that ClusterSourceTemplate. It contains a templated `PipelineRun`, which is how your app flows through a Tekton Pipeline.

### [tekton-pipeline.yml](tekton-pipeline.yml)

This is the Tekton Pipeline. Largely stolen from this [Cartographer tutorial](https://cartographer.sh/docs/v0.7.0/tutorials/lifecycle/).

### [tekton-task.yml](tekton-task.yml)

This is _THE ACTUAL TASK_. It was borrowed from the Tekton Task Hub and modified to fit our needs (link in the file). Don't worry about the details. Just know that it runs a Sonarqube scan, connecting to an existing Sonarqube server.

## Sonarqube, Quality Gate Supply Chain and Git Writer Example

//TODO