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
      
      Change the names and storage size if required as below:

      ![image](https://github.com/KhuldoonIbm/stepzen/assets/108668456/4fadb58f-4a10-414a-b27f-ed71c978bf59)

