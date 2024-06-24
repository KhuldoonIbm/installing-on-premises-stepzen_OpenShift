# stepzen
Installation requirements
•	Supported platforms: Red Hat OpenShift Container Platform (OCP) 4.12, or Kubernetes v.126 or v1.27
•	Cert-manager must be installed on the cluster, or (OpenShift only) you configured TLS support.
•	You need an IBM entitlement key for accessing IBM Container Software Library.
•	A running PostgreSQL DB (version 15 or newer) must be accessible from the cluster.
•	If you deploy the on-premises Introspection service, install it on the same cluster, and in the same stepzen namespace, where you install API Connect Essentials.


1-  Create new namespace if required:
```yaml
      oc new-project stepzen
```     
2-	Create ibm-entitlement-key 
```yaml
      kubectl create secret docker-registry ibm-entitlement-key --docker-server=cp.icr.io --docker-username=cp --docker-password=<IBM entitlement key>
```
3-	Install Crunchy Postgres for Kubernetes:
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

NAME              TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
hippo-ha          ClusterIP   10.103.73.92   <none>        5432/TCP   3h14m
hippo-ha-config   ClusterIP   None           <none>        <none>     3h14m
hippo-pods        ClusterIP   None           <none>        <none>     3h14m
hippo-primary     ClusterIP   None           <none>        5432/TCP   3h14m
hippo-replicas    ClusterIP   10.98.110.215  <none>        5432/TCP   3h14m
