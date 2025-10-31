##  Generate Containerization Assets with AI

**What You'll Do:** Use AI-powered tools to generate Docker and Kubernetes manifests for your modernized Spring Boot application.

**What You'll Learn:** How to create production-ready containerization assets - including optimized Dockerfiles and Kubernetes manifests configured with health checks, secrets, and workload identity.

---

### Using Containerization Assist

In the GitHub Copilot agent chat, use the following prompt to generate production-ready Docker and Kubernetes manifests:

```prompt
/petclinic Help me containerize the application. Create me a new Dockerfile and update my ACR with @lab.CloudResourceTemplate(LAB502).Outputs[acrLoginServer]
```

!!! note
    To expedite your lab experience, you can allow the Containerization Assist MCP server to run on this Workspace. Select **Allow in this Workspace** or **Always Allow**.
    
    ![Containerization Assist MCP allow](../assets/containerization-assist-mcp-allow.png)
    
    You will also need to allow the MCP server to make LLM requests. 
    Select **Always**.
    
    ![Containerization Assist MCP LLM](../assets/containerization-assist-mcp-llm.png)

The Containerization Assist MCP Server will analyze your repository and generate:

- **Dockerfile**: Multi-stage build with optimized base image

- **Kubernetes Deployment**: With Azure workload identity, PostgreSQL secrets, health checks, and resource limits

- **Kubernetes Service**: LoadBalancer configuration for external access

**Expected Result**: Kubernetes manifests in the `k8s/` directory.

!!! tip
    You are almost there. You will deploy the AI generated files, but they might need some tuning later. Before deploying it to your cluster, double check the image location, the use of workload identity and if the service connector secret reference in the deployment file are correct to your environment.

### Build and Push Container Image to ACR

Build the containerized application and push it to your Azure Container Registry:

1. In your terminal window, login to ACR using Azure CLI

	```bash
	az acr login --name @lab.CloudResourceTemplate(LAB502).Outputs[acrName]
  
	```

1. Build the Docker image in Azure Container Registry

	```bash
	az acr build -t petclinic:0.0.1 . -r @lab.CloudResourceTemplate(LAB502).Outputs[acrName]
	```