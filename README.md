DashboardController.java
package com.project.v1.controller;

import com.project.v1.dto.DashboardDTO;
import com.project.v1.model.Dashboard;
import com.project.v1.service.DashboardService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;
import java.util.Optional;

@RestController
@RequestMapping("/api/dashboards")
public class DashboardController {

    @Autowired
    private DashboardService dashboardService;

    @PostMapping
    public ResponseEntity<Dashboard> createDashboard(@RequestBody Dashboard dashboard) {
        return ResponseEntity.ok(dashboardService.saveDashboard(dashboard));
    }

    @GetMapping
    public ResponseEntity<List<DashboardDTO>> getAllDashboards() {
        return ResponseEntity.ok(dashboardService.getAllDashboards());
    }

//    extra
    @GetMapping("/all")
    public ResponseEntity<List<Dashboard>> getAllFullDashboards() {
        return ResponseEntity.ok(dashboardService.getAllFullDashboards());
    }

    @PutMapping("/{id}")
    public ResponseEntity<Dashboard> updateDashboard(@PathVariable Long id, @RequestBody Dashboard dashboard) {
        Optional<Dashboard> updated = dashboardService.updateDashboard(id, dashboard);
        return updated.map(ResponseEntity::ok).orElse(ResponseEntity.notFound().build());
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteDashboard(@PathVariable Long id) {
        boolean deleted = dashboardService.deleteDashboard(id);
        if(deleted){
            return ResponseEntity.noContent().build();
        }
        else{
            return ResponseEntity.notFound().build();
        }

    }

}

DashboardDTO
package com.project.v1.dto;

import com.project.v1.model.Dashboard;
import lombok.Data;

import java.time.LocalDateTime;

@Data
public class DashboardDTO {
    private String name;
    private String description;
    private String createdBy;
    private LocalDateTime createdDate;
    private String modifiedBy;
    private LocalDateTime modifiedDate;
    private boolean isPublic;

    public DashboardDTO(Dashboard d){
        this.name = d.getName();
        this.description = d.getDescription();
        this.createdBy = d.getCreatedBy();
        this.createdDate = d.getCreatedDate();
        this.modifiedBy = d.getModifiedBy();
        this.modifiedDate =d.getModifiedDate();
        this.isPublic = d.isPublic();
    }
}

Dashboard
package com.project.v1.model;

import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import lombok.*;

import java.time.LocalDateTime;

@Entity
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class Dashboard {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private String description;
    private String createdBy;
    private LocalDateTime createdDate;
    private String modifiedBy;
    private LocalDateTime modifiedDate;
    private boolean isPublic;

    private String model;
    private String groupBy;
    private String aggregation;
    private String aggregationField;

}

DashboardRepository
package com.project.v1.repository;

import com.project.v1.model.Dashboard;
import org.springframework.data.jpa.repository.JpaRepository;

public interface DashboardRepository  extends JpaRepository<Dashboard,Long> {
}

DashboardService
package com.project.v1.service;

import com.project.v1.dto.DashboardDTO;
import com.project.v1.model.Dashboard;
import com.project.v1.repository.DashboardRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.time.LocalDateTime;
import java.util.List;
import java.util.Optional;
import java.util.stream.Collectors;

@Service
public class DashboardService {

    @Autowired
    private DashboardRepository dashboardRepository;

    public Dashboard saveDashboard(Dashboard dashboard){
        dashboard.setCreatedDate(LocalDateTime.now());
        dashboard.setModifiedDate(LocalDateTime.now());
        return dashboardRepository.save(dashboard);
    }

    public List<DashboardDTO> getAllDashboards(){
        return dashboardRepository.findAll()
                .stream()
                .map(DashboardDTO::new)
                .collect(Collectors.toList());
    }

    public List<Dashboard> getAllFullDashboards(){
        return dashboardRepository.findAll();
    }

    public Optional<Dashboard> updateDashboard(Long id, Dashboard updated) {
        return dashboardRepository.findById(id).map(existing -> {
            existing.setName(updated.getName());
            existing.setDescription(updated.getDescription());
            existing.setPublic(updated.isPublic());
            existing.setModel(updated.getModel());
            existing.setGroupBy(updated.getGroupBy());
            existing.setAggregation(updated.getAggregation());
            existing.setAggregationField(updated.getAggregationField());
            existing.setModifiedBy(updated.getModifiedBy());
            existing.setModifiedDate(LocalDateTime.now());
            return dashboardRepository.save(existing);
        });
    }

    public boolean deleteDashboard(Long id) {
        if(dashboardRepository.existsById(id)){
            dashboardRepository.deleteById(id);
            return true;
        }
        return  false;
    }
}

V1Application
package com.project.v1;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class V1Application {

	public static void main(String[] args) {
		SpringApplication.run(V1Application.class, args);
	}

}

application.properties
spring.application.name=v1
spring.jpa.hibernate.ddl-auto=update
spring.datasource.url=jdbc:mysql://localhost:3306/bnp
spring.datasource.username=root
spring.datasource.password=2004
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.jpa.show-sql: true
_____________________________________________________________________________________________________________
To modify the existing Spring Boot application to work with Oracle instead of MySQL, you'll need to update your application.properties and make a few small adjustments, particularly related to the database connection and Hibernate dialect configuration.

1. Update application.properties
You will change the database connection properties to point to Oracle instead of MySQL. Oracle uses a different JDBC URL format and requires a specific driver and dialect.

Updated application.properties:

spring.application.name=v1

# Hibernate Dialect for Oracle
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.Oracle12cDialect

# JPA settings
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true

# DataSource configuration for Oracle
spring.datasource.url=jdbc:oracle:thin:@//localhost:1521/xe   # Oracle URL (replace 'xe' with your Oracle SID or service name)
spring.datasource.username=your_oracle_username
spring.datasource.password=your_oracle_password
spring.datasource.driver-class-name=oracle.jdbc.OracleDriver

Make sure to replace your_oracle_username, your_oracle_password, and localhost:1521/xe with your actual Oracle DB username, password, and connection URL.

Explanation of the Oracle JDBC URL:
jdbc:oracle:thin:@//localhost:1521/xe: This is the typical JDBC connection string for Oracle.

Replace localhost with the hostname or IP address of your Oracle DB.

Replace 1521 with the port number your Oracle DB is using (default is 1521).

Replace xe with the Oracle SID or Service Name you are using. For example, if your Oracle SID is orcl, you would use @//localhost:1521/orcl.

2. Update Hibernate Dialect
In Oracle, we use the Oracle12cDialect for Hibernate to communicate with the Oracle database properly. This is set in spring.jpa.properties.hibernate.dialect.

3. Oracle Driver Dependency
If you haven't already included the Oracle JDBC driver in your pom.xml, you'll need to add it.

Maven Dependency for Oracle JDBC Driver:
If you are using Maven, add the following dependency for Oracle JDBC:

<dependency>
    <groupId>com.oracle.database.jdbc</groupId>
    <artifactId>ojdbc8</artifactId>
    <version>19.8.0.0</version> <!-- Ensure to use the correct version -->
</dependency>
Make sure the version matches the version of your Oracle database and JDBC driver.

4. Entity Considerations
The entities themselves (like Dashboard and DashboardDTO) typically don’t require any changes because JPA will handle the SQL generation automatically, but there are some points to note when migrating from MySQL to Oracle:

Auto-increment fields: Oracle uses sequences for auto-increment fields, while MySQL uses the AUTO_INCREMENT keyword. With @GeneratedValue(strategy = GenerationType.IDENTITY) in your entities, JPA will use Oracle sequences behind the scenes to generate primary key values. If you're experiencing issues with auto-generation, you might want to define an Oracle sequence explicitly like this:

@Id
@GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "dashboard_seq")
@SequenceGenerator(name = "dashboard_seq", sequenceName = "dashboard_seq", allocationSize = 1)
private Long id;
Ensure you create a sequence in your Oracle DB if you need to use a custom sequence.

sql
Copy
Edit
CREATE SEQUENCE dashboard_seq START WITH 1 INCREMENT BY 1;
Other than this, the model structure should remain the same as long as you use JPA annotations.

5. Oracle SQL Syntax Adjustments
Oracle SQL has some syntax differences from MySQL. For example, in Oracle:

VARCHAR has a maximum length of 4000 characters.

The TEXT type in MySQL would be represented by CLOB in Oracle.

Date handling in Oracle is different, but JPA will automatically map LocalDateTime to Oracle's TIMESTAMP type.

Your entity class Dashboard with LocalDateTime fields (like createdDate and modifiedDate) will work fine, as JPA will map it to Oracle's TIMESTAMP.

Final Code Changes Recap:
application.properties for Oracle DB:
properties
Copy
Edit
spring.application.name=v1
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.Oracle12cDialect
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.datasource.url=jdbc:oracle:thin:@//localhost:1521/xe
spring.datasource.username=your_oracle_username
spring.datasource.password=your_oracle_password
spring.datasource.driver-class-name=oracle.jdbc.OracleDriver
Add Oracle JDBC Dependency in pom.xml:
xml
Copy
Edit
<dependency>
    <groupId>com.oracle.database.jdbc</groupId>
    <artifactId>ojdbc8</artifactId>
    <version>19.8.0.0</version> <!-- Ensure to use the correct version -->
</dependency>
Entity (optional adjustments for auto-increment fields in Oracle):

@Id
@GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "dashboard_seq")
@SequenceGenerator(name = "dashboard_seq", sequenceName = "dashboard_seq", allocationSize = 1)
private Long id;
Make sure you create the sequence in Oracle:
CREATE SEQUENCE dashboard_seq START WITH 1 INCREMENT BY 1;
Conclusion
This configuration should enable your Spring Boot application to work with an Oracle database while retaining the same functionality as the original MySQL-based application. The majority of changes are related to the database connection and dialect configuration in application.properties. The actual code for entities, services, and controllers should not require significant changes.

_________________________________________________________________________________________________________________________________________________________________________________

application.properties (Updated for Oracle)
properties
Copy
Edit
spring.application.name=v1

# Hibernate Dialect for Oracle
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.Oracle12cDialect

# JPA settings
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true

# DataSource configuration for Oracle
spring.datasource.url=jdbc:oracle:thin:@//localhost:1521/xe
spring.datasource.username=your_oracle_username
spring.datasource.password=your_oracle_password
spring.datasource.driver-class-name=oracle.jdbc.OracleDriver
🔁 Replace your_oracle_username, your_oracle_password, and localhost:1521/xe with actual values.

✅ pom.xml (Add Oracle JDBC dependency)
Add this inside the <dependencies> section:

xml
Copy
Edit
<dependency>
    <groupId>com.oracle.database.jdbc</groupId>
    <artifactId>ojdbc8</artifactId>
    <version>19.8.0.0</version>
</dependency>
✅ Dashboard.java (Updated for Oracle SEQUENCE)
java
Copy
Edit
package com.project.v1.model;

import jakarta.persistence.*;
import lombok.*;

import java.time.LocalDateTime;

@Entity
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class Dashboard {

    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "dashboard_seq")
    @SequenceGenerator(name = "dashboard_seq", sequenceName = "dashboard_seq", allocationSize = 1)
    private Long id;

    private String name;
    private String description;
    private String createdBy;
    private LocalDateTime createdDate;
    private String modifiedBy;
    private LocalDateTime modifiedDate;
    private boolean isPublic;

    private String model;
    private String groupBy;
    private String aggregation;
    private String aggregationField;
}
✅ SQL to Create Oracle Sequence
Run this in your Oracle database:

sql
Copy
Edit
CREATE SEQUENCE dashboard_seq START WITH 1 INCREMENT BY 1;
✅ Other Files (No Change Needed)
All the following remain the same as they’re already JPA-compatible:

DashboardController.java

DashboardDTO.java

DashboardRepository.java

DashboardService.java

V1Application.java

✅ Summary of Changes
Area	Action
application.properties	Switched from MySQL to Oracle connection and dialect
pom.xml	Added ojdbc8 dependency
Entity class	Replaced IDENTITY with SEQUENCE and added @SequenceGenerator
Oracle SQL	Manually create sequence dashboard_seq







