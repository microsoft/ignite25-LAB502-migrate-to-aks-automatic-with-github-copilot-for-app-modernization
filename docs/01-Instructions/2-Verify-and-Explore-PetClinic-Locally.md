
## Verify and Explore PetClinic Locally

**What You'll Do:** Confirm that the locally deployed PetClinic application is running with PostgreSQL, and explore its main features.

**What You'll Learn:** How to verify a local Spring Boot application connected to a Docker-based PostgreSQL database and navigate its core functionality.

---

### Verify the Application

1. In VS Code, open a new terminal by pressing ``Ctrl+` `` (backstick) or go to **Terminal** → **New Terminal** in the menu.

1. In the new terminal, run the petclinic

	```bash
	 mvn clean compile && mvn spring-boot:run \
    -Dspring-boot.run.arguments="--spring.messages.basename=messages/messages --spring.datasource.url=jdbc:postgresql://localhost/petclinic --spring.sql.init.mode=always --spring.sql.init.schema-locations=classpath:db/postgres/schema.sql --spring.sql.init.data-locations=classpath:db/postgres/data.sql --spring.jpa.hibernate.ddl-auto=none"
	```

1. Open your browser and go to `http://localhost:8080` to confirm the PetClinic application is running.

![PetClinic application running](../assets/petclinic-app-running.png)

**Explore the PetClinic Application:**

Once it's running, try out the key features:

* **Find Owners:** Select **"FIND OWNERS"**, leave the Last Name field blank, and click "Find Owner" to list all 10 owners.

* **View Owner Details:** Click an owner (e.g., Betty Davis) to see their information and pets.

* **Edit Pet Information:** From an owner's page, click **"Edit Pet"** to view or modify pet details.

* **Review Veterinarians:** Go to **"VETERINARIANS"** to see the 6 vets and their specialties (radiology, surgery, dentistry).

After exploring the PetClinic application, you can stop it by pressing `CTRL+C`