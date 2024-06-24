# stepzen
Installation requirements
•	Supported platforms: Red Hat OpenShift Container Platform (OCP) 4.12, or Kubernetes v.126 or v1.27
•	Cert-manager must be installed on the cluster, or (OpenShift only) you configured TLS support.
•	You need an IBM entitlement key for accessing IBM Container Software Library.
•	A running PostgreSQL DB (version 15 or newer) must be accessible from the cluster.
•	If you deploy the on-premises Introspection service, install it on the same cluster, and in the same stepzen namespace, where you install API Connect Essentials.


1-  Create new namespace if required:
      oc new-project stepzen
      
2-	Create ibm-entitlement-key 
      kubectl create secret docker-registry ibm-entitlement-key --docker-server=cp.icr.io --docker-username=cp --docker-password=<IBM entitlement key>

3-	Install Crunchy Postgres for Kubernetes:
      Go to OpenShift operator hub -> search for Postgres -> select "Crunchy Postgres for Kubernetes" -> Install -> choose Installation mode -> click "Install" -> wait till the operator installed ->
      view operatoe -> select "Postgres Cluster" -> click "Create PostgresCluster" 
      

      kind: PostgresCluster
      apiVersion: postgres-operator.crunchydata.com/v1beta1
      metadata:
        name: _stepzen_
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

Example:      

![image](https://github.com/KhuldoonIbm/stepzen/assets/108668456/eef89088-10f1-47e7-b950-fd73bcfd515a)


