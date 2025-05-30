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
dialect.



4. Entity Considerations
The entities themselves (like Dashboard and DashboardDTO) typically don’t require any changes because JPA will handle the SQL generation automatically, but there are some points to note when migrating from MySQL to Oracle:

Auto-increment fields: Oracle uses sequences for auto-increment fields, while MySQL uses the AUTO_INCREMENT keyword. With @GeneratedValue(strategy = GenerationType.IDENTITY) in your entities, JPA will use Oracle sequences behind the scenes to generate primary key values. If you're experiencing issues with auto-generation, you might want to define an Oracle sequence explicitly like this:

@Id
@GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "dashboard_seq")
@SequenceGenerator(name = "dashboard_seq", sequenceName = "dashboard_seq", allocationSize = 1)
private Long id;
Ensure you create a sequence in your Oracle DB if you need to use a custom sequence.


CREATE SEQUENCE dashboard_seq START WITH 1 INCREMENT BY 1;
Other than this, the model structure should remain the same as long as you use JPA annotations.



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


application.properties

________________________________________
DashboardDTO.java
package com.example.recon_connect.dto;

import com.example.recon_connect.model.Dashboard;

import java.time.LocalDateTime;

import lombok.Data;

@Data

public class DashboardDTO {

private String name;

private String description;

private String createdBy;

private LocalDateTime createdDate;

private String modifiedBy;

private LocalDateTime modifiedDate;

private boolean isPublic;

public DashboardDTO (Dashboard d) {

this.name = d.getName();

this.description = d.getDescription();

this.createdBy= d.getCreatedBy();

this.createdDate = d.getCreatedDate();

this.modifiedBy= d.getModifiedBy();

this.modifiedDate = d.getModifiedDate();

this.isPublic = d.isPublic();

}

}

DashboardController.java
package com.example.recon_connect.controller;

import com.example.recon_connect.dto.DashboardDTO;

import com.example.recon_connect.model.Dashboard;

import com.example.recon_connect.service.DashboardService;

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

public ResponseEntity<Dashboard> createDashboard (@RequestBody Dashboard dashboard) {

return ResponseEntity.ok(dashboardService.saveDashboard(dashboard));

}

@GetMapping

public ResponseEntity<List<DashboardDTO>> getAllDashboards() {

return ResponseEntity.ok(dashboardService.getAllDashboards());

}

@GetMapping("/all")

public ResponseEntity<List<Dashboard>> getAllFullDashboards() {

return ResponseEntity.ok(dashboardService.getAllFullDashboards());

}

@PutMapping("/{id}")

public ResponseEntity<Dashboard> updateDashboard (@PathVariable Long id, @RequestBody Dashboard dashboard) {

Optional<Dashboard> updated = dashboardService.updateDashboard(id, dashboard);

return updated.map(ResponseEntity::ok).orElse (ResponseEntity.notFound().build());

}

@DeleteMapping("/{id}")

public ResponseEntity<Void> deleteDashboard(@PathVariable Long id) {

boolean deleted = dashboardService.deleteDashboard(id);

if(deleted) {

return ResponseEntity.noContent().build();

}

else{

I

}

return Response Entity.notFound().build();

}



}

application.properties

server.port=8083

spring.datasource.url=jdbc:oracle:thin:@eurvlid34782.xmp.net.intra:1521/NEXTGEN1

spring.datasource.username=NEXTGEN1

spring.datasource.password=X<A67w0>BlPzw*RiqEMSwJHqU

spring.datasource.driver-class-name=oracle.jdbc.OracleDriver

spring.jpa.hibernate.ddl-auto=update

spring.jpa.show-sql=true


Dashboard.java(model)
package com.example.recon_connect.model;

import jakarta.persistence.*;

import lombok.*;

import java.time. LocalDateTime;

@Entity

@Data

@NoArgsConstructor

@AllArgsConstructor

@Builder

public class Dashboard {

@Id

@GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "dashboard_seq")

@SequenceGenerator(name = "dashboard_seq", sequenceName = "dashboard_seq", allocationSize= 1)

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

DashboardRepository.java
package com.example.recon_connect.repository;

import com.example.recon_connect.model.Dashboard;

import org.springframework.data.jpa.repository. JpaRepository;


public interface DashboardRepository extends JpaRepository<Dashboard, Long> {

}

DashboardService.java
package com.example.recon_connect.service;

import com.example.recon_connect.dto.DashboardDTO;

import com.example.recon_connect.model.Dashboard;

import com.example.recon_connect.repository. DashboardRepository;

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

public Dashboard saveDashboard (Dashboard dashboard) {

dashboard.setCreated Date (LocalDateTime.now());

dashboard.setModifiedDate (LocalDateTime.now());

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
public Optional<Dashboard> updateDashboard (Long id, Dashboard updated) {

return dashboardRepository.findById(id).map(existing -> {

existing.setName(updated.getName());

existing.setDescription (updated.getDescription());

existing.setPublic (updated.isPublic());

existing.setModel(updated.getModel());

existing.setGroupBy(updated.getGroupBy());

existing.setAggregation (updated.getAggregation());

existing.setAggregationField(updated.getAggregationField());

existing.setModifiedBy(updated.getModifiedBy());

existing.setModifiedDate(LocalDateTime.now());

return dashboardRepository.save(existing);

});

3

}

public boolean deleteDashboard (Long id) {

if (dashboardRepository.existsById(id)){

dashboardRepository.deleteById(id);

return true;

}

return false;

}

}

-------_-------
Required Changes (Frontend)
✅ 1. Fix DashboardService (Angular)
Replace in-memory array with HTTP calls.

🔁 dashboard.service.ts
ts
Copy code
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Dashboardd } from './dashboard.model';
import { Observable } from 'rxjs';

@Injectable({
  providedIn: 'root'
})
export class DashboardService {
  private apiUrl = 'http://localhost:8083/api/dashboards';

  constructor(private http: HttpClient) {}

  getDashboards(): Observable<Dashboardd[]> {
    return this.http.get<Dashboardd[]>(this.apiUrl);
  }

  addDashboard(dashboard: any): Observable<any> {
    return this.http.post(this.apiUrl, dashboard);
  }
}
✅ 2. Modify BlankDashboardComponent
Replace in-memory call with service call.

🔁 Inject and Use DashboardService in BlankDashboardComponent
Update import and constructor:

ts
Copy code
import { DashboardService } from '../dashboard.service';

constructor(private router: Router, private dashboardService: DashboardService) {}
🔁 Update submit() method:
ts
Copy code
submit() {
  const dashboard = {
    name: this.formData.name,
    description: this.formData.description,
    public: this.formData.visibility === 'Public',
    createdBy: 'User',  // Replace with real user info if available
    createdDate: new Date().toISOString(),
    modifiedBy: 'User',
    modifiedDate: new Date().toISOString(),
    model: this.model,
    groupBy: this.groupBy,
    aggregation: this.aggregation,
    aggregationField: this.aggregationField
  };

  this.dashboardService.addDashboard(dashboard).subscribe({
    next: () => {
      alert('Dashboard submitted successfully!');
      this.router.navigate(['/inventory/manage-dashboard']);
    },
    error: () => alert('Failed to save dashboard')
  });
}
✅ 3. Update ManageDashboardComponent
Fetch dashboards from backend and display in the table.

🔁 Modify imports:
ts
Copy code
import { DashboardService } from './dashboard.service';
🔁 Inject service in constructor:
ts
Copy code
constructor(private dashboardService: DashboardService) {}
🔁 Add ngOnInit:
ts
Copy code
ngOnInit() {
  this.loadDashboards();
}

loadDashboards() {
  this.dashboardService.getDashboards().subscribe({
    next: (data) => this.tableData = data,
    error: () => alert('Failed to load dashboards')
  });
}
✅ 4. Register HttpClientModule
In your app.config.ts, ensure this is added:

ts
Copy code
import { provideHttpClient } from '@angular/common/http';

export const appConfig: ApplicationConfig = {
  providers: [
    provideHttpClient(),
    ...
  ]
};
✅ Backend: No changes needed ✅
You're already exposing the correct endpoints (POST /api/dashboards and GET /api/dashboards), and they return DashboardDTO which matches your frontend structure.

🔚 Final Output
Once these changes are made:

Submitting the form in Blank Dashboard will send the data to the backend.

Upon returning to Manage Dashboard, it will fetch and display all dashboards in the table.

Would you like me to provide a downloadable .zip with fixed Angular service and component code? Or do you want me to double-check your HTML too?








Tools



ChatGPT can make mistakes. Check important info. See Cookie Preferenc