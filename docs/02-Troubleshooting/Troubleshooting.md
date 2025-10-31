
### Help

In this section you can find tips on how to troubleshoot your lab.

---

#### Troubleshooting the local deployment

**If the application fails to start:**
1. Check Docker is running: `docker ps`
2. Verify PostgreSQL container is healthy: `docker logs petclinic-postgres`
3. Check application logs: `tail -f ~/app.log`
4. Ensure port 8080 is not in use: `lsof -i :8080`

**If the database connection fails:**
1. Verify PostgreSQL container is running on port 5432: `docker port petclinic-postgres`
2. Test database connectivity: `docker exec -it petclinic-postgres psql -U petclinic -d petclinic -c "SELECT 1;"`

---
### Troubleshooting the Service Connector

### Retrieve PostgreSQL Configuration from AKS Service Connector

Before you can use **Containerization Assist**, you must first retrieve the PostgreSQL Service Connector configuration from your AKS cluster.

This information ensures that your generated Kubernetes manifests are correctly wired to the database using managed identity and secret references.

### Access AKS Service Connector and Retrieve PostgreSQL Configuration

1. Open a new tab in the Edge browser and navigate to `https://portal.azure.com/`

1. In the top search bar, type **aks-petclinic** and select the AKS Automic cluster.

	![Select AKS cluster](../assets/select-aks-cluster.png)

1. In the left-hand menu under **Settings**, select **Service Connector**.

	![Select Service Connector](../assets/select-service-connector.png)

1.  You'll see the service connection that was automatically created **PostgreSQL connection** with a name that starts with **postgresflexible_** connecting to your PostgreSQL flexible server.

1. Select the **DB for PostgreSQL flexible server** and click the **YAML snippet** button in the action bar

	![YAML snippet button](../assets/service-connector-yaml-snippet.png)

1. Expand this connection to see the variables that were created by the `sc-postgresflexiblebft3u-secret` in the cluster

	![Service Connector variables](../assets/service-connector-variables.png)

### Retrieve PostgreSQL YAML Configuration

The Azure Portal will display a YAML snippet showing how to use the Service Connector secrets for PostgreSQL connectivity.

??? example "Service Connector YAML snippet"
    ![PostgreSQL YAML config sample](../assets/postgres-yaml-config-sample.png)

!!! note
    1. The portal shows a sample deployment with workload identity configuration
    2. Key Elements:
        - Service account: `sc-account-d4157fc8-73b5-4a68-acf4-39c8f22db792`
        - Secret reference: `sc-postgresflexiblebft3u-secret`
        - Workload identity label: `azure.workload.identity/use: "true"`
    
    The Service Connector secret (`sc-postgresflexiblebft3u-secret` in this example), will contain the following variables:
    
    - AZURE_POSTGRESQL_HOST
    - AZURE_POSTGRESQL_PORT
    - AZURE_POSTGRESQL_DATABASE
    - AZURE_POSTGRESQL_CLIENTID (map to both AZURE_CLIENT_ID and AZURE_MANAGED_IDENTITY_NAME)
    - AZURE_POSTGRESQL_USERNAME

---

#### Troubleshooting the application in AKS

If for some reason you've made here and your deployment did not work, your deployment file should ressemble this example.

!!! tip "Key areas to pay close attention to are:"
    - `azure.workload.identity/use: "true"`
    - `serviceAccountName: sc-account-XXXX` this needs to reflect the service account created earlier during the PostgreSQL Service Connector
    - `image: <acr-login-server>/petclinic:0.0.1` this should point to your ACR and image created earlier.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: spring-petclinic
  labels:
    app: spring-petclinic
    version: v1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: spring-petclinic
  template:
    metadata:
      labels:
        app: spring-petclinic
        version: v1
        azure.workload.identity/use: "true"  # Enable Azure Workload Identity
    spec:
      serviceAccountName: sc-account-71b8f72b-9bed-472a-8954-9b946feee95c # change this
      containers:
      - name: spring-petclinic
        image: acrpetclinic556325.azurecr.io/petclinic:0.0.1 # change this value
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 8080
          name: http
          protocol: TCP
        
        # Environment variables from Azure Service Connector secret
        env:
        # Azure Workload Identity - automatically injected by webhook
        # AZURE_CLIENT_ID, AZURE_TENANT_ID, AZURE_FEDERATED_TOKEN_FILE are set by workload identity
        
        # Map PostgreSQL host from secret - with Azure AD authentication parameters
        - name: POSTGRES_URL
          value: "jdbc:postgresql://$(AZURE_POSTGRESQL_HOST):$(AZURE_POSTGRESQL_PORT)/$(AZURE_POSTGRESQL_DATABASE)?sslmode=require&authenticationPluginClassName=com.azure.identity.extensions.jdbc.postgresql.AzurePostgresqlAuthenticationPlugin"
        - name: POSTGRES_USER
          valueFrom:
            secretKeyRef:
              name: sc-postgresflexible4q7w6-secret # change this value
              key: AZURE_POSTGRESQL_USERNAME
        # Client ID is also needed for Spring Cloud Azure
        - name: AZURE_CLIENT_ID
          valueFrom:
            secretKeyRef:
              name: sc-postgresflexible4q7w6-secret # change this value
              key: AZURE_POSTGRESQL_CLIENTID
              optional: true
        - name: AZURE_MANAGED_IDENTITY_NAME
          valueFrom:
            secretKeyRef:
              name: sc-postgresflexible4q7w6-secret # change this value
              key: AZURE_POSTGRESQL_CLIENTID
        - name: AZURE_POSTGRESQL_HOST
          valueFrom:
            secretKeyRef:
              name: sc-postgresflexible4q7w6-secret # change this value
              key: AZURE_POSTGRESQL_HOST
        - name: AZURE_POSTGRESQL_PORT
          valueFrom:
            secretKeyRef:
              name: sc-postgresflexible4q7w6-secret # change this value
              key: AZURE_POSTGRESQL_PORT
        - name: AZURE_POSTGRESQL_DATABASE
          valueFrom:
            secretKeyRef:
              name: sc-postgresflexible4q7w6-secret # change this value
              key: AZURE_POSTGRESQL_DATABASE
        - name: SPRING_PROFILES_ACTIVE
          value: "postgres"
        # Spring Cloud Azure configuration for workload identity
        - name: SPRING_CLOUD_AZURE_CREDENTIAL_MANAGED_IDENTITY_ENABLED
          value: "true"
        - name: SPRING_DATASOURCE_AZURE_PASSWORDLESS_ENABLED
          value: "true"        
        # Make all secret keys available in the pod
        envFrom:
        - secretRef:
            name: sc-postgresflexible4q7w6-secret # change this value
        # Health check probes
        livenessProbe:
          httpGet:
            path: /actuator/health
            port: 8080
            scheme: HTTP
          initialDelaySeconds: 60
          periodSeconds: 10
          timeoutSeconds: 3
          successThreshold: 1
          failureThreshold: 3
        readinessProbe:
          httpGet:
            path: /actuator/health
            port: 8080
            scheme: HTTP
          initialDelaySeconds: 30
          periodSeconds: 5
          timeoutSeconds: 3
          successThreshold: 1
          failureThreshold: 3
        # Resource limits and requests
        resources:
          requests:
            memory: "512Mi"
            cpu: "500m"
          limits:
            memory: "1Gi"
            cpu: "1000m"
        # Security context
        securityContext:
          runAsNonRoot: true
          runAsUser: 1000
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: false
          capabilities:
            drop:
            - ALL
      # Pod security context
      securityContext:
        fsGroup: 1000
      # Restart policy
      restartPolicy: Always
```