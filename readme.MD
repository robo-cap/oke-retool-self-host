# Prerequisites

1. Provision a [PostgreSQL database](https://docs.oracle.com/en-us/iaas/Content/postgresql/create-db.htm#top) and ensure the OKE worker nodes have connectivity to the database.

    You can use the smallest configuration available:
    - 2 OCPUs
    - 75k IOPS
    - Single Node

2. Deploy Nginx ingress controller to OKE using the [nginx helm chart](https://github.com/kubernetes/ingress-nginx/tree/main/charts/ingress-nginx).
   
- Add the nginx repo
    
    ```
    helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
    helm repo update
    ```

- Prepare the `nginx-ingress-override.yaml` file
    
    Uncomment the `oci.oraclecloud.com/oci-network-security-groups: "nsg-ocid"` line in the `nginx-ingress-override.yaml` file and replace `nsg-ocid` with the OCID of the NSG you would like to use to secure the LB. (This step is optional if you don't want to use NSGs to secure the LB traffic).

- Install nginx
    
    ```
    helm upgrade --install ingress-nginx ingress-nginx/ingress-nginx -n ingress-nginx --create-namespace -f nginx-ingress-override.yaml
    ```
    
- Determine the public IP address of the Load Balancer

    ```
    k get svc -A | grep ingress-nginx | grep LoadBalancer
    ```

3. Make sure you meet the prerequisite for retool self hosting:

    https://docs.retool.com/self-hosted/quickstarts/kubernetes/helm#cluster-size

# Setup Retool in OKE

The purpose of this document is to document the steps required to deploy retool to OKE using helm charts.

Use this documentation as a reference: https://docs.retool.com/self-hosted/quickstarts/kubernetes/helm

1. Register the domain you intend to use with your application and create an A-record for the Public IP address of the LoadBalancer (e.g.: retool.domain). If you don't have access to manage a DNS server you can use `nip.io`.

2. Update the values specific to your deployment in the `retool-override.yaml` file.

3. Setup Google OAUTH using [these instructions](https://docs.retool.com/sso/quickstarts/google/google-sign-in) and below data:

- Authorized JavaScript origins

    ```
    http://retool.domain
    ```

- Authorized redirect URIs
    
    ```
    http://retool.domain/oauthcallback
    http://retool.domain/oauth
    ```

4. Prepare the helm charts for retool

- Add retool repo

    ```
    helm repo add retool https://charts.retool.com
    helm repo update
    ```

- Install the helm chart

    ```
    helm install my-retool retool/retool -f retool-override.yaml
    ```

5. Setup the database

- Your pods may not start from the first time.

    ```
    kubectl get pods | grep my-retool
    ```

- If you check the logs on the job-runner, you may see an log entry saying it's waiting for migrations each 1 second.
    
    ```
    kubectl logs my-retool-jobs-runner-7648596757-zwl9n
    ```

- Connect to the job runner pod and execute `yarn sequelize db:migrate` command

    ```
    kubectl exec -it my-retool-jobs-runner-7648596757-zwl9n -- bash
    $ yarn sequelize db:migrate
    ```

6. Open a new tab in your browser and navigate to the page: http://retool.domain






