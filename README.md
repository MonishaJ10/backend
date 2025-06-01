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


@CrossOrigin(origins = "http://localhost:4200")
@RestController
@RequestMapping("/api/dashboards")
public class DashboardController {

    @Autowired
    private DashboardService dashboardService;

    @Autowired
    public DashboardController(Dashboard Service dashboardservice){
    	this.dashboardService = dashboardService;
    }

    @PostMapping
    public ResponseEntity<Dashboard> createDashboard(@RequestBody Dashboard dashboard) {
        return ResponseEntity.ok(dashboardService.saveDashboard(dashboard));
    }

    @GetMapping
    public ResponseEntity<List<DashboardDTO>> getAllDashboards() {
        return ResponseEntity.ok(dashboardService.getAllDashboards());
    }

    // extra
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
    private Long id;

    public DashboardDTO(Dashboard d){
    	this.id = d.getId();
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
    @GeneratedValue(strategy = GenerationType.SEQUENCE,generator = 'dashboard_seq')
    @SequenceGenerator(name = "dashboard_seq",sequenceName = "dashboard_seq",allocationSize = 1)
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
server.port=8083
spring.datasource.url=jdbc:oracle:thin:@eurvlid34782.xmp.net.intra:1521/NEXTGEN1
spring.datasource.username=NEXTGEN1
spring.datasource.password=X<A67w0>BlPzw*RiqEMSwJHqU
spring.datasource.driver-class-name=oracle.jdbc.OracleDriver
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql: true

Make sure you create the sequence in Oracle:
CREATE SEQUENCE dashboard_seq START WITH 1 INCREMENT BY 1;

package com.example.recon_connect.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.CorsRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class CorsConfig {
    @Bean
    public WebMvcConfigurer corsConfigurer() {
        return new WebMvcConfigurer() {
            @Override
            public void addCorsMappings(CorsRegistry registry) {
                registry.addMapping("/api/**")
                        .allowedOrigins("http://localhost:4200") // your Angular app URL
                        .allowedMethods("GET", "POST", "PUT", "DELETE", "OPTIONS")
                        .allowedHeaders("*")
                        .allowCredentials(true);
            }
        };
    }
}
