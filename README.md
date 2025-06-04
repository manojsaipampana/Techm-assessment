# Techm-assessment
Problem Statement: : E-commerce Product Catalog
solution:
package com.example.catalog;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.*;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.context.annotation.*;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.security.core.userdetails.*;
import jakarta.persistence.*;
import java.util.List;

@SpringBootApplication
public class ProductCatalogApplication {
    public static void main(String[] args) {
        SpringApplication.run(ProductCatalogApplication.class, args);
    }
    @Entity
    public static class Product {
        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        private Long productId;
        private String name;
        private String description;
        private Double price;
        private String category;
        private Integer stockQuantity;

        public Long getProductId() { return productId; }
        public void setProductId(Long productId) { this.productId = productId; }

        public String getName() { return name; }
        public void setName(String name) { this.name = name; }

        public String getDescription() { return description; }
        public void setDescription(String description) { this.description = description; }

        public Double getPrice() { return price; }
        public void setPrice(Double price) { this.price = price; }

        public String getCategory() { return category; }
        public void setCategory(String category) { this.category = category; }

        public Integer getStockQuantity() { return stockQuantity; }
        public void setStockQuantity(Integer stockQuantity) { this.stockQuantity = stockQuantity; }
    }
     public interface ProductRepo extends JpaRepository<Product, Long> {
        List<Product> findByCategory(String category);
        List<Product> findByPriceBetween(Double min, Double max);
        List<Product> findByNameContainingIgnoreCase(String name);
    }
    
    @RestController
    @RequestMapping("/products")
    public static class ProductController {
        private final ProductRepo repo;
        public ProductController(ProductRepo repo) { this.repo = repo; }
        @GetMapping public List<Product> getAll() { return repo.findAll(); }
        
        @PostMapping public Product add(@RequestBody Product p) { return repo.save(p); }
        
        @PutMapping("/{id}") public Product update(@PathVariable Long id, @RequestBody Product p) {
            p.id = id; return repo.save(p);
        }
        @DeleteMapping("/{id}") public void delete(@PathVariable Long id) { repo.deleteById(id); }

        @GetMapping("/category/{cat}") public List<Product> byCat(@PathVariable String cat) {
            return repo.findByCategory(cat);
        }
        @GetMapping("/price") public List<Product> byPrice(@RequestParam double min, @RequestParam double max) {
            return repo.findByPriceBetween(min, max);
        }
        @GetMapping("/search") public List<Product> search(@RequestParam String name) {
            return repo.findByNameContainingIgnoreCase(name);
        }
    }
     @Configuration
    public static class SecurityConfig {

        @Bean
        public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
            http.csrf().disable()
                .authorizeHttpRequests(auth -> auth
                    .requestMatchers("/api/products").authenticated()
                    .requestMatchers("/api/products/**").authenticated()
                    .requestMatchers("/api/products").hasAnyRole("ADMIN", "USER")
                    .requestMatchers("/api/products/**").hasAnyRole("ADMIN", "USER")
                    .requestMatchers("/api/products", "/api/products/**").hasAnyRole("ADMIN", "USER")
                    .anyRequest().authenticated()
                )
                .httpBasic();
            return http.build();
        }

        @Bean
        public UserDetailsService users() {
            UserDetails admin = User.withDefaultPasswordEncoder()
                .username("admin").password("admin123").roles("ADMIN").build();
            UserDetails user = User.withDefaultPasswordEncoder()
                .username("user").password("user123").roles("USER").build();
            return new InMemoryUserDetailsManager(admin, user);
        }
    }
}
