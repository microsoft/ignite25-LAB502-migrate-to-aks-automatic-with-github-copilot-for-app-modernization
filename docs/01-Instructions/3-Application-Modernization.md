## Application Modernization

**What You'll Do:** Use GitHub Copilot app modernization to assess, remediate, and modernize the Spring Boot application in preparation to migrate the workload to AKS Automatic.

**What You'll Learn:** How GitHub Copilot app modernization works, demonstration of modernizing elements of legacy applications, and the modernization workflow

---

Next let's begin our modernization work. 

1. Select  `GitHub Copilot app modernization` extension
	
	![GitHub Copilot app modernization extension](../assets/copilot-appmod-extension.png)

### Execute the Assessment

Now that you have GitHub Copilot setup, you can use the assessment tool to analyze your Spring Boot PetClinic application using the configured analysis parameters.

1. Navigate the Extension Interface and click **Migrate to Azure** to begin the modernization process.

	![Extension interface](../assets/appmod-extension-interface.png)

<!-- 1. Allow the GitHub Copilot app modernization to sign in to GitHub 
	!IMAGE[ghcp-allow-signin.png](instructions310381/ghcp-allow-signin.png)

1. Authorize your user to sign in

	!IMAGE[gh-auth-user.png](instructions310381/gh-auth-user.png)

1. And finally, authorized it again on this screen

	!IMAGE[gh-auth-screen.png](instructions310381/gh-auth-screen.png)

1. The assessment will start now. Notice that GitHub will install the AppCAT CLI for Java. This might take a few minutes

	!IMAGE[appcat-install.png](instructions310381/appcat-install.png) -->

!!! tip "Hint" 
	You can follow the progress of the upgrade by looking at the Terminal in vscode

![Assessment rules in terminal](../assets/assessment-rules-terminal.png)

### Overview of the Assessment

Assessment results are consumed by GitHub Copilot App Modernization (AppCAT). AppCAT examines the scan findings and produces targeted modernization recommendations to prepare the application for containerization and migration to Azure.

- target: the desired runtime or Azure compute service you plan to move the app to.
- mode: the analysis depth AppCAT should use.

**Analysis targets**

Target values select the rule sets and guidance AppCAT will apply.

| Target | Description |
|--------|---------|
| azure-aks | Guidance and best practices for deploying to Azure Kubernetes Service (AKS). |
| azure-appservice | Guidance and best practices for deploying to Azure App Service. |
| azure-container-apps | Guidance and best practices for deploying to Azure Container Apps. |
| cloud-readiness | General recommendations to make the app "cloud-ready" for Azure. |
| linux | Recommendations to make the app Linux-ready (packaging, file paths, runtime details). |
| openjdk11 | Compatibility and runtime recommendations for running Java 8 apps on Java 11. |
| openjdk17 | Compatibility and runtime recommendations for running Java 11 apps on Java 17. |
| openjdk21 | Compatibility and runtime recommendations for running Java 17 apps on Java 21. |

**Analysis modes**

Choose how deep AppCAT should inspect the project.

| Mode | Description |
|--------|---------|
| source-only | Fast analysis that examines source code only. |
| full | Full analysis: inspects source code and scans dependencies (slower, more thorough). |

!!! info "Where to change these options"
    You can customize this report by editing the file at **.github/appmod-java/appcat/assessment-config.yaml** to change targets and modes.

    For this lab, AppCAT runs with the following configuration:

    ```yaml
    appcat:
      - target:
          - azure-aks
          - azure-appservice
          - azure-container-apps
          - cloud-readiness
        mode: source-only
    ```

    If you want a broader scan (including dependency checks) change `mode` to `full`, or add/remove entries under `target` to focus recommendations on a specific runtime or Azure compute service.

### Review the Assessment results

After the assessment completes, you'll see a success message in the GitHub Copilot chat summarizing what was accomplished:

![Assessment report overview](../assets/assessment-report-overview.png)

The assessment analyzed the Spring Boot Petclinic application for cloud migration readiness and identified the following:

Key Findings:

* 8 cloud readiness issues requiring attention (1)
* 1 Java upgrade opportunity for modernization (2)

**Resolution Approach:** More than 50% of the identified issues can be automatically resolved through code and configuration updates using GitHub Copilot's built-in app modernization capabilities (3).

**Issue Prioritization:** Issues are categorized by urgency level to guide remediation efforts:

* Mandatory (Purple) - Critical issues that must be addressed before migration.
* Potential (Blue) - Performance and optimization opportunities.
* Optional (Gray) - Nice-to-have improvements that can be addressed later.

This prioritization framework ensures teams focus on blocking issues first while identifying opportunities for optimization and future enhancements.

### Review Specific Findings

Click on individual issues in the report to see detailed recommendations. In practice, you would review all recommendations and determine the set that aligns with your migration and modernization goals for the application.

!!! note
    For this lab, we will spend our time focusing on one modernization recommendation: updating the code to use modern authentication via Azure Database for PostgreSQL Flexible Server with Entra ID authentication.

| Aspect | Details |
|--------|---------|
| **Modernization Lab Focus** | Database Migration to Azure PostgreSQL Flexible Server |
| **What was found** | PostgreSQL database configuration using basic authentication detected in Java source code files |
| **Why this matters** | External dependencies like on-premises databases with legacy authentication must be resolved before migrating to Azure |
| **Recommended solution** | Migrate to Azure Database for PostgreSQL Flexible Server |
| **Benefits** | Fully managed service with automatic backups, scaling, and high availability |