## Deploy to AKS

**What You'll Do:** Deploy the modernized application to AKS Automatic using Service Connector secrets for passwordless authentication with PostgreSQL.

**What You'll Learn:** Kubernetes deployment with workload identity, Service Connector integration, and testing deployed applications with Entra ID authentication.

---

!!! info "About AKS Automatic"
    AKS Automatic is a new mode for Azure Kubernetes Service that provides an optimized and simplified Kubernetes experience. It offers automated cluster management, built-in security best practices, intelligent scaling, and pre-configured monitoring - making it ideal for teams who want to focus on applications rather than infrastructure management.

### Deploy the application to AKS Automatic

Using Containerization Assist we have built a Kubernetes manifest for the Petclini application. In the next steps we will deploy it to the AKS Automatic cluster and verify that it is working:

1. Deploy the application:

	```bash
	kubectl apply -f k8s/petclinic.yaml
	```

1.  Monitor deployment status

	```bash
	kubectl get pods,services,deployments
	```

	It might take a minute for the AKS Automatic cluster to provision new nodes for the workload so it is normal to see your pods in a `Pending` state until the new nodes are available. You can verify is there are nodes available with the `kubectl get nodes` command.

	```bash
	NAME                                    READY   STATUS              RESTARTS   AGE
	petclinic-deployment-5f9db48c65-qpb8l   0/1     Pending             0          2m2s
	```

### Verify Deployment and Connectivity

Test the deployed application and verify Entra ID authentication:

1. Port forward to access the application

	```bash
  	kubectl port-forward svc/spring-petclinic-service 9090:8080
	```
1. To test the application, open a new tab in Microsoft Edge and go to `http://localhost:9090`


### Validate Entra ID Authentication

Verify that the application is using passwordless authentication:

1. Check environment variables in the pod (get first pod with label)
	```bash
	POD_NAME=$(kubectl get pods -l app=spring-petclinic -o jsonpath='{.items[0].metadata.name}')
	kubectl exec $POD_NAME -- env | grep POSTGRES
	```

	Expected output:

	```bash
	AZURE_POSTGRESQL_PORT=5432
	AZURE_POSTGRESQL_DATABASE=petclinic
	AZURE_POSTGRESQL_USERNAME=aad_pg
	AZURE_POSTGRESQL_CLIENTID=1094a914-1837-406a-ad58-b9dcc499177a
	AZURE_POSTGRESQL_HOST=db-petclinic55954159.postgres.database.azure.com
	AZURE_POSTGRESQL_SSL=true
	POSTGRES_USER=aad_pg
	```

1. Verify no password environment variables are present

	```bash
	kubectl exec $POD_NAME -- env | grep -i pass
	```

	Expected output:
  
	```bash
	SPRING_DATASOURCE_AZURE_PASSWORDLESS_ENABLED=true
	```

1. Check application logs for successful authentication

	```bash
	kubectl logs -l app=spring-petclinic --tail=100 | grep -i "hibernate"
	```

	Expected outcome:

	```bash
	[...]
	Hibernate: drop table if exists pets cascade
	Hibernate: drop table if exists specialties cascade
	Hibernate: drop table if exists types cascade
	Hibernate: drop table if exists vet_specialties cascade
	[...]
	```

**Expected Outcome:** The application is successfully deployed to AKS with passwordless authentication to PostgreSQL using Entra ID and workload identity.