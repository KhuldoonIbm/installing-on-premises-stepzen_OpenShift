# API Connect Essentials (StepZen) Installation:
Installation requirements:  
1- 	Supported platforms: Red Hat OpenShift Container Platform (OCP) 4.12, or Kubernetes v.126 or v1.27.  
2- 	Cert-manager must be installed on the cluster, or (OpenShift only) you configured TLS support.  
3-	You need an IBM entitlement key for accessing IBM Container Software Library.  
4- 	A running PostgreSQL DB (version 15 or newer) must be accessible from the cluster.  
5- 	If you deploy the on-premises Introspection service, install it on the same cluster, and in the same stepzen namespace, where you install API Connect        Essentials.


# 1-  Create new namespace if required:
```yaml
      oc new-project stepzen
```     
# 2-	Create ibm-entitlement-key 
```yaml
      kubectl create secret docker-registry ibm-entitlement-key --docker-server=cp.icr.io --docker-username=cp --docker-password=<IBM entitlement key>
```
# 3-	Install Crunchy Postgres for Kubernetes or any PostgreSQL DB (version 15 or newer) :
      Go to OpenShift operator hub -> search for Postgres -> select "Crunchy Postgres for Kubernetes" -> Install -> choose Installation mode -> click "Install" -> wait till the operator installed ->
      view operatoe -> select "Postgres Cluster" -> click "Create PostgresCluster" ->
      Update the yaml file as below:
      
```yaml
      kind: PostgresCluster
      apiVersion: postgres-operator.crunchydata.com/v1beta1
      metadata:
        name: stepzen-pgclus
        namespace: stepzen
      spec:
        backups:
          pgbackrest:
            repos:
              - name: repo1
                volume:
                  volumeClaimSpec:
                    accessModes:
                      - ReadWriteOnce
                    resources:
                      requests:
                        storage: 2Gi
        instances:
          - dataVolumeClaimSpec:
              accessModes:
                - ReadWriteOnce
              resources:
                requests:
                  storage: 2Gi
            replicas: 1
        postgresVersion: 15
```
Example:      

![image](https://github.com/KhuldoonIbm/stepzen/assets/108668456/eef89088-10f1-47e7-b950-fd73bcfd515a)

Track the state of the Postgres Pod using the following command:
```yaml
kubectl -n stepzen get pods   --selector=postgres-operator.crunchydata.com/cluster=stepzen-pgclus,postgres-operator.crunchydata.com/instance
```
To inspect what services are available, you can run the following command:
```yaml
kubectl -n stepzen get svc --selector=postgres-operator.crunchydata.com/cluster=stepzen-pgclus
```
which will yield something similar to:

![image](https://github.com/KhuldoonIbm/stepzen/assets/108668456/eb67b48e-d1ef-4cb6-856c-1bb18b5085da)

After the Postgres cluster is initialized, PGO will bootstrap a database and create a Postgres user that your application can use to access the database

> This information is stored in a Secret named with the pattern: 'clusterName'-pguser-'userName'
> so in our case: "stepzen-pgclus-pguser-stepzen-pgclus"

Open this secret, then reveale the values:
![image](https://github.com/KhuldoonIbm/stepzen/assets/108668456/92e01054-b121-48c5-ba07-bd5d045994de)

# 4- Create a generic secret containing the key DSN with a DSN pointing to your PostgreSQL server (for OpenShift, replace kubectl with oc):
```yaml
kubectl create secret generic db-secret --from-literal=DSN=<dsn>
```
> where "dsn" is a string of the form postgresql://user:password@host/database.  
So, in our case:
```yaml
 kubectl create secret generic db-secret --from-literal=DSN=postgresql://stepzen-pgclus:PASSWORD@stepzen-pgclus-primary.stepzen.svc/stepzen-pgclus
```
> Note: better to get the password from "jdbc-uri" if it contains special characters.  

# 5- Download and extract the CASE bundle.
a- Navigate to the ibm-stepzen-case folder at GitHub.com:
> https://github.com/IBM/cloud-pak/tree/master/repo/case/ibm-stepzen-case  

b- Open the subfolder that represents the newest version of the CASE bundle. The subfolders are named with <version+timestamp>; for example:
> cloud-pak/repo/case/ibm-stepzen-case/1.0.0+20230925.000000  

c- Download the newest CASE bundle to the local directory.
CASE bundles are named ibm-stepzen-case-<version+timestamp>.tgz; for example:  

> ibm-stepzen-case-1.0.0+20230925.000000.tgz  

Download the CASE bundle by clicking the Download icon as shown in the following image:  
![image](https://github.com/KhuldoonIbm/stepzen/assets/108668456/03882803-4285-452b-a022-388af97f8e13)  

d- Extract the CASE bundle by running the following command:  
> tar zxvf ibm-stepzen-case-<version+timestamp>.tgz  

The contents are the CASE bundle are stored in a new directory called ibm-stepzen-case.  

# 6- Apply the operator manifest files to your cluster.  
a- In the extracted CASE, navigate to the following directory, where the installation files are stored:  
> ibm-stepzen-case/inventory/stepzenGraphOperator/files/deploy  

b- In the /deploy folder, locate the following files:  
+ crd.yaml is the operator Custom Resource Definition (CRD), a schema for the Custom Resource (CR) used to configure the operator.  
+ operator.yaml defines the operator resources, including the active service account and associated roles, webhook, and operator deployment configuration.
  
c- Apply the files by running the following commands (for OpenShift, replace kubectl with oc):  
```yaml
kubectl apply -f crd.yaml 
kubectl apply -f operator.yaml 
```

# 7- Create a custom resource (CR) that defines the configuration settings for your deployment.  
The /deploy folder also contains a sample custom resource (CR) file named cr.yaml. Modify the CR to customize your GraphQL server.

At minimum, make the following changes to enable installation:

Tip: For information on settings, see CR [configuration settings](https://www.ibm.com/docs/en/SSMNED_ESS_1.x/admin/spec.html).
+ Accept the license agreement.
> Review the API Connect Essentials License agreement, and then set the key spec.licence.accept to true in the CR file to indicate your agreement.

+ Add your registry access token.
> Set spec.imagePullSecrets to contain the name of the registry access token you created in step 2.

+ Add your PostgreSQL secret.
> Set spec.controlDatabaseSecret to the name of the secret containing the PostgreSQL DSN (that is, containing a DSN key), which you created in step 4.

The following snippet shows an example CR with minimal contents, which configures horizontal pod autoscaling (HPA) for the graph and graph subscription services:  
```yaml
apiVersion: stepzen-graph.ibm.com/v1beta1
kind: StepZenGraphServer
metadata:
  name: stepzen
spec:
  license:
    accept: true
  controlDatabaseSecret: db-secret
  imagePullSecrets: 
    - ibm-entitlement-key
  graphServer:
    hpa:
      minReplicas: 1
      maxReplicas: 10
      targetCPUUtilizationPercentage: 80
  graphServerSubscription:
    hpa:
      minReplicas: 1
      maxReplicas: 5
      targetCPUUtilizationPercentage: 80
```

# 8- Configure API Connect Essentials by applying the CR.  
Apply the CR by running the following command (for OpenShift, replace kubectl with oc):  
```yaml
kubectl apply -f cr.yaml
```
Verify that the operator is running:
```yaml
kubectl get StepZenGraphServer 
```
When the installation completes successfully, you see a message similar to the following example, which indicates that the API Connect Essentials Graph Service is now running in your cluster:  
```yaml
NAME      STATUS   SUMMARY              AGE
stepzen   Ready    Services are ready   1m
```
When you run kubectl get pods (or oc get pods), you see three types of pods running:

+ stepzen-graph-operator-xxxx: Operator pod managing the service  
+ stepzen-graph-server-xxxx: Graph Server handling single GraphQL requests (queries and mutations)  
+ stepzen-graph-server-subscription-xxxx: Graph Server handling GraphQL subscriptions (persistent queries)  
