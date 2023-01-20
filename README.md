# How to use Pipelines to do GitOps with HyperShift
This repository shows an easy to follow example of how to provision a HyperShift cluster in AWS using a Pipeline tasks (supports GitOps)

## Prerequisites
1. An OpenShift cluster with at least three nodes
2. Install the Multicluster Engine operator
   a. Enable hypershift_preview
   b. Create the S3 OIDC credential (See MCE HyperShift documentation)
3. Install the OpenShift Pipelines operator
4. Install the Provision and Destroy tasks
    ```shell
    # Installs to your current namespace
    # Add -n <my-namspace> to setup the tasks elsewhere

    oc apply -f ./tasks/provision.yaml -f ./tasks/destroy.yaml
    ```

## Provisioning a cluster
1. To provision an AWS cluster you need to provide an AWS credential. In the OCP console navigate to: `All Clusters > Credentials` then `Add crential` and follow the steps. >>IMPORTANT<< Make sure you create the credential in the same namespace that you applied the tasks.
2. Now that the credential is created, the user only needs to be granted permissions to create the `taskRun` resources in the `default` or `<my-namespace>` project.  This is also why this approach works so well for GitOps. The AWS secret does not need to be exposed outside the cluster to any of the users.
3. Edit the `run/provision-taskRun.yaml` and change the `value` for the `clusterName` parameter. This should be a unique name for the cluster.
4. Apply the `provision-taskRun.yaml`
    ```
    # This command runs in the current namespace
    # Add -n <my-namespace> to apply the taskRun elsewhere

    oc apply -f ./run/provision-taskRun.yaml
    ```
5. This results in the provisioning of the HyperShift cluster. You can use the ocp pipline cli command to view the log.
    ```shell
    # Show running tasks
    tkn taskrun list

    # Show log
    tkn taskrun logs hcp1
    ```
6. Navigate to `All Clusters > Infrastructure > Clusters` If the taskrun completed successfully, you will see a hosted cluster provisioning. Click on its name to see the control plane details (monitor the provisioning) and to import the cluster

### Gitops
The only difference when you want to create a cluster with GitOps is you commit the `provision-taskRun.yaml` to Git or GitLab and it will be applied to the OCP just like the `oc apply...` command above. >>Important<< Make sure the taskRun resource is created in the same project(namespace) as the task.