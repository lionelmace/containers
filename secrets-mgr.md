---

copyright:
  years: 2022, 2022
lastupdated: "2022-10-11"

keywords: secrets manager, secrets, certificates, secret group, CRN

subcollection: containers, openshift

---

{{site.data.keyword.attribute-definition-list}}


# Setting up {{site.data.keyword.secrets-manager_short}} in your {{site.data.keyword.containershort}} cluster
{: #secrets-mgr}

When you integrate {{site.data.keyword.secrets-manager_full_notm}} with your {{site.data.keyword.containerlong_notm}} cluster, you can centrally manage Ingress subdomain certificates and other secrets. 
{: shortdesc}

## About Secrets Manager
{: #secrets-mgr_about}

With {{site.data.keyword.secrets-manager_short}}, you can use a single service to manage your secrets and control who has access to them. A {{site.data.keyword.secrets-manager_short}} instance is not automatically provisioned in your cluster. However, you can use a single {{site.data.keyword.secrets-manager_short}} instance across multiple clusters, and a single cluster can have more than one instance.

### What functionality can I gain with {{site.data.keyword.secrets-manager_short}}?
{: #secrets-mgr_about_functionality}

With {{site.data.keyword.secrets-manager_short}}, you can:
- Create managed Kubernetes secrets with Ingress TLS certificates included.
- Create Kubernetes secrets of any type by using the CRN of any {{site.data.keyword.secrets-manager_short}} instance you own.
- Automatically update your secrets in your cluster on a regular basis.
- Track the experation dates of your certificates from the {{site.data.keyword.cloud_notm}} console.
- Control who has access to your secrets by creating secret groups for approved users.

Note that to have your secrets automatically updated, you must register at least one {{site.data.keyword.secrets-manager_short}} instance to your cluster. For more information, see [Registering your {{site.data.keyword.secrets-manager_short}} instance to your cluster](#secrets-mgr_setup_register).
{: note}

### What types of secrets are supported with {{site.data.keyword.secrets-manager_short}}?
{: #secrets-mgr_about_types}

{{site.data.keyword.secrets-manager_short}} supports IAM credentials, key-value secrets, user credentials, arbitrary secrets, and Kubernetes secrets. For Kubernetes secrets, {{site.data.keyword.secrets-manager_short}} supports both TLS and Opaque secret types. With TLS secrets, you can specify one certificate CRN. With Opaque secrets, you can specify multiple fields to pull non-certificate secrets. If you do not specify a secret type, TLS is applied by default. 

For more information on supported secrets, see [Working with secrets of different types](/docs/secrets-manager?topic=secrets-manager-what-is-secret#secret-types).

### What is the difference between the `ibmcloud ks ingress instance` CLI commands and the `ibmcloud ks ingress secret` CLI commands?
{: #secrets-mgr_about_cli}

There are two sets of CLI commands that work directly with {{site.data.keyword.secrets-manager_short}} instances in {{site.data.keyword.containerlong_notm}}: the `ibmcloud ks ingress secret` commands and the `ibmcloud ks ingress instance` commands. The `ibmcloud ks ingress instance` commands are used to manage your {{site.data.keyword.secrets-manager_short}} instances. The `ibmcloud ks ingress secret` commands are used to manage your Ingress secrets that are stored in a {{site.data.keyword.secrets-manager_short}} instance or secrets that are written directly to the cluster. 

## Setting up your Secrets Manager instance
{: #secrets-mgr_setup}

Follow the steps to set up {{site.data.keyword.secrets-manager_short}} in your cluster.
{: shortdesc}

These steps are intended for users setting up {{site.data.keyword.secrets-manager_short}} in new clusters or in clusters without {{site.data.keyword.cloudcerts_long_notm}}. If you are migrating your secrets from {{site.data.keyword.cloudcerts_long_notm}} to {{site.data.keyword.secrets-manager_short}}, follow the steps in the [{{site.data.keyword.cloudcerts_long_notm}} Migration doc instead](/docs/containers?topic=containers-certs-mgr-migration).
{: important}

### Enable service-to-service communication
{: #secrets-mgr_setup_s2s}
{: step}

Integrating {{site.data.keyword.secrets-manager_short}} with your {{site.data.keyword.containerlong_notm}} cluster requires service-to-service communication auhtorization. Follow the steps below to set up the authorization. For additional info, see [Integrations for Secrets Manager](/docs/secrets-manager?topic=secrets-manager-integrations#create-authorization).
{: shortdesc}
 
1. In the {{site.data.keyword.cloud_notm}} console, click **Manage** > **Access (IAM)**.
2. Click **Authorizations**.
3. Click **Create**.
4. In the **Source service** list, select **Kubernetes Service**.
5. Select the option to scope the access to **All resources**.
6. In the **Target service** list, select **Secrets Manager**.
7. Select the option to scope the access to **All resources**. 
8. In the **Service access** section, check the **Manager** option. 
9. Click **Authorize**. 

### Create a {{site.data.keyword.secrets-manager_short}} instance
{: #secrets-mgr_setup_create}
{: step}

To create a {{site.data.keyword.secrets-manager_short}} instance in the CLI or the UI, refer to the {{site.data.keyword.secrets-manager_short}} documentation. It may take several minutes for you {{site.data.keyword.secrets-manager_short}} instance to fully provision.
{: shortdesc}

- [Create a {{site.data.keyword.secrets-manager_short}} instance in CLI](/docs/secrets-manager?topic=secrets-manager-create-instance&interface=cli).
- [Create a {{site.data.keyword.secrets-manager_short}} instance in the UI](/docs/secrets-manager?topic=secrets-manager-create-instance).

When you create a {{site.data.keyword.secrets-manager_short}} instance, it is not provisioned directly in your cluster. You must register your new {{site.data.keyword.secrets-manager_short}} instance with your cluster in the next step. 
{: note}

### Register your {{site.data.keyword.secrets-manager_short}} instance to your cluster
{: #secrets-mgr_setup_register}
{: step}

Follow the steps to register your {{site.data.keyword.secrets-manager_short}} instance to your cluster.
{: shortdesc}

1. Get the CRN of the {{site.data.keyword.secrets-manager_short}} instance. In the output, the CRN is in the **ID** row. 

    ```sh
    ibmcloud resource service-instance <instance_name>
    ```
    {: pre}

    Example output.

    ```sh
    Name:                  my-secrets-manager-instance 
    ID:                    crn:v1:bluemix:public:secrets-manager:us-south:a/1aa111aa1a11111aaa1a1111aa1aa111:111a1111-11a1-111a-1111-1a1a1a1111a1::
    GUID:                  111a1111-11a1-111a-1111-1a1a1a1111a1 
    Location:              us-south   
    Service Name:          secrets-manager   
    Service Plan Name:     standard   
    Resource Group Name:   default   
    State:                 active   
    Type:                  service_instance   
    Sub Type:                 
    Created at:            2022-06-08T12:46:45Z   
    Created by:            user@ibm.com   
    Updated at:            2022-06-08T12:54:45Z 
    ```
    {: screen}

2. Register the instance to your cluster. Specify the instance CRN found in the previous step.

    If you want to register an instance to a cluster and also [set it as the default instance](#secrets-mgr_setup_default), include the `--is-default` option. Otherwise, you can set a default instance with the `ibmcloud ks ingress instance default set` command.
    {: tip}

    ```sh
    ibmcloud ks ingress instance register --cluster <cluster_name_or_id> --crn <instance_crn> [--is-default]
    ```
    {: pre}

3. Verify that the {{site.data.keyword.secrets-manager_short}} instance was registered to the cluster. 

    ```sh
    ibmcloud ks ingress instance ls --cluster <cluster_name_or_id>
    ```
    {: pre}

    Example output.

    ```sh
    Name                                Type              Is Default   Status    Secret Group   CRN   
    my-secrets-manager-instance         secrets-manager   false        created   default        crn:v1:bluemix:public:secrets-manager:us-south:a/1aa111aa1a11111aaa1a1111aa1aa111:111a1111-11a1-111a-1111-1a1a1a1111a1::   
    ```
    {: pre}


You can specify a {{site.data.keyword.secrets-manager_short}} instance and a secret group when you [create a cluster](/docs/containers?topic=containers-clusters&interface=cli) with the [`ibmcloud ks cluster create classic`](/docs/containers?topic=containers-kubernetes-service-cli&interface=cli#cs_cluster_create) or [`ibmcloud ks cluster create vpc-gen2`](/docs/containers?topic=containers-kubernetes-service-cli&interface=cli#cli_cluster-create-vpc-gen2) commands. Use the `--sm-instance` option to register an instance to the cluster and the `--sm-group` option to specify a secret group that can access the secrets on the cluster. See [Registering a {{site.data.keyword.secrets-manager_short}} instance when creating a cluster](#secrets-mgr_cluster_create).
{: tip} 

### Set a default {{site.data.keyword.secrets-manager_short}} instance and regenerate your secrets
{: #secrets-mgr_setup_default}
{: step}

When you set a default {{site.data.keyword.secrets-manager_short}} instance, all new Ingress subdomain certificates are stored in that instance.
{: shortdesc}

1. Run the command to set the new default instance. You can optionally specify a [secret group](/docs/secrets-manager?topic=secrets-manager-secret-groups) that is allowed access to the secrets in the instance. 

    ```sh
    ibmcloud ks ingress instance default set --cluster <cluster_name_or_id> --name <instance_name> --secret-group <secret_group_id>
    ```
    {: pre}

2. Regenerate your secrets. Any secrets that are managed by IBM, such as your default Ingress secrets, are uploaded to the new default instance. These secrets are automatically updated and the CRN is changed to reference the {{site.data.keyword.secrets-manager_short}} instance.

    1. List the nlb-dns subdomains in your cluster.

        ```sh
        ibmcloud ks nlb-dns ls --cluster <cluster_name_or_id>
        ```
        {: pre}
  
    2. For each subdomain in your cluster, run the command to regenerate your IBM-managed secrets. This updates the CRN of these secrets to reference the CRN of the new default {{site.data.keyword.secrets-manager_short}} instance.

        ```sh
        ibmcloud ks nlb-dns secret regenerate --cluster <cluster_name_or_id> --nlb-subdomain <nlb_subdomain>
        ```
        {: pre}


    3. Verify that your default Ingress secrets regenerated. In the output, the CRN of the default Ingress secrets should contain `secrets-manager`.

        It may take several minutes for your secrets to regenerate. During this process, the **Status** column in the output says `regenerating` and switches to `created` when the regeneration is complete. 
        {: note}

        ```sh
        ibmcloud ks ingress secret ls --cluster <cluster_name_or_id>
        ```
        {: pre}

        Example output.

        ```sh
        Name                               Namespace  CRN                                                                               Expires On                 Domain                                                   Status    Type   
        secret-11111aa1a1a11aa1111111-000  default    crn:v1:bluemix:public:secrets-manager:us-south:a/1aa111aa1:secret:a111aa11-11a1   2022-12-21T19:52:29+0000   secret-11111aa1a1a.us-south.containers.appdomain.cloud   created   TLS   
        secret-22222aa2a2a22aa2222222-000  default    crn:v1:bluemix:public:secrets-manager:us-south:a/2aa222aa2:secret:a222aa22-22a2   2022-12-21T19:52:29+0000   secret-22222aa2a2a.us-south.containers.appdomain.cloud   created   TLS    
        ```
        {: screen}

## Controlling access to your secrets with secret groups
{: #secrets-mgr_groups}

With {{site.data.keyword.secrets-manager_short}}, you can use secret groups to control who has access to the secrets in your cluster. A secret group can be assigned to an IAM access group so that only select users or service IDs can access the secrets within the secret group. For more information, see [Organizing your secrets](/docs/secrets-manager?topic=secrets-manager-secret-groups&interface=ui).
{: shortdesc}

## Registering a {{site.data.keyword.secrets-manager_short}} instance when creating a cluster
{: #secrets-mgr_cluster_create}

If you are [creating a new Classic or VPC cluster](/docs/containers?topic=containers-clusters&interface=cli) in the CLI, you have the option to include flags to register an existing {{site.data.keyword.secrets-manager_short}} instance and secret group to the cluster. Secrets in the cluster are stored in the {{site.data.keyword.secrets-manager_short}} instance and applied to the secret goup. 
{: shortdesc}

The {{site.data.keyword.secrets-manager_short}} instance registered during cluster create does not automatically become the default {{site.data.keyword.secrets-manager_short}} instance. You must still [set the default instance](#secrets-mgr_setup_default) manually.
{: note}

Use the following CLI options when creating a cluster:
- `--sm-instance`: Use this option to register a {{site.data.keyword.secrets-manager_short}} instance to the cluster by specifying the instance CRN. To find the CRN of a {{site.data.keyword.secrets-manager_short}} instance, run `ibmcloud resource service-instance <name_of_instance>` or navigate to your resource list in the UI and click on the instance.
- `--sm-group`: Use this option to specify the ID of the secret group. To find the secret group ID, run `ibmcloud secrets-manager secret-groups`.