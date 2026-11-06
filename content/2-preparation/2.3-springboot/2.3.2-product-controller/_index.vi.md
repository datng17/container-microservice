---
title: "Tạo Controller, Model và cấu hình DynamoDB, X-Ray"
date: "2026-09-30"
weight: 2
chapter: false
pre: "<b>2.3.2. </b>"
---

### 1. Tạo Model và Controller cho Product

* Tạo thư mục **products** và tạo file **Product.java**. Copy đoạn mã sau:

~~~java
import software.amazon.awssdk.enhanced.dynamodb.mapper.annotations.DynamoDbBean;
import software.amazon.awssdk.enhanced.dynamodb.mapper.annotations.DynamoDbPartitionKey;

@DynamoDbBean
public class Product {
    private String id;
    private String name;
    private Double price;

    public Product() {}

    public Product(String id, String name, Double price) {
        this.id = id;
        this.name = name;
        this.price = price;
    }

    @DynamoDbPartitionKey
    public String getId() { return id; }
    public void setId(String id) { this.id = id; }

    public String getName() { return name; }
    public void setName(String name) { this.name = name; }

    public Double getPrice() { return price; }
    public void setPrice(Double price) { this.price = price; }
}
~~~

![Architect](/images/UPDATE/02-environment/3.infraSpring/11.png)

* Tạo file **ProductController.java**. Copy đoạn mã sau:

~~~java
import java.util.ArrayList;
import java.util.List;

import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.DeleteMapping;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.PutMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import software.amazon.awssdk.enhanced.dynamodb.DynamoDbEnhancedClient;
import software.amazon.awssdk.enhanced.dynamodb.DynamoDbTable;
import software.amazon.awssdk.enhanced.dynamodb.Key;
import software.amazon.awssdk.enhanced.dynamodb.TableSchema;

@RestController
@RequestMapping("/api/products")
public class ProductController {
    private static final Logger LOG = LogManager.getLogger(ProductController.class);
    private final DynamoDbTable<Product> productTable;

    public ProductController(DynamoDbEnhancedClient enhancedClient) {
        this.productTable = enhancedClient.table("products", TableSchema.fromBean(Product.class));
    }

    @GetMapping
    public List<Product> getAllProducts() {
        LOG.info("Fetching all products from DynamoDB");
        List<Product> products = new ArrayList<>();
        productTable.scan().items().forEach(products::add);
        return products;
    }

    @GetMapping("/{id}")
    public ResponseEntity<Product> getProductById(@PathVariable String id) {
        LOG.info("Fetching product: {}", id);
        Product product = productTable.getItem(Key.builder().partitionValue(id).build());
        return product != null ? ResponseEntity.ok(product) : ResponseEntity.notFound().build();
    }

    @PostMapping
    public List<Product> createProduct(@RequestBody List<Product> products) {
        for (Product product : products) {
            LOG.info("Creating product: {}", product.getId());
            productTable.putItem(product);
        }
        return products;
    }

    @PutMapping
    public Product updateProduct(@RequestBody Product product) {
        LOG.info("Updating product: {}", product.getId());
        productTable.updateItem(product);
        return product;
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteProduct(@PathVariable String id) {
        LOG.info("Deleting product: {}", id);
        productTable.deleteItem(Key.builder().partitionValue(id).build());
        return ResponseEntity.noContent().build();
    }
}
~~~

![Architect](/images/UPDATE/02-environment/3.infraSpring/12.png)

---

### 2. Cấu hình DynamoDB và AWS X-Ray

* Tạo file **DynamoDbConfig.java** (Cấu hình Client kèm X-Ray Interceptor). Copy đoạn mã sau:

~~~java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import com.amazonaws.xray.interceptors.TracingInterceptor;

import software.amazon.awssdk.enhanced.dynamodb.DynamoDbEnhancedClient;
import software.amazon.awssdk.services.dynamodb.DynamoDbClient;

@Configuration
public class DynamoDbConfig {
    @Bean
    public DynamoDbClient dynamoDbClient() {
        return DynamoDbClient.builder()
                .overrideConfiguration(builder -> builder.addExecutionInterceptor(new TracingInterceptor()))
                .build();
    }

    @Bean
    public DynamoDbEnhancedClient dynamoDbEnhancedClient(DynamoDbClient dynamoDbClient) {
        return DynamoDbEnhancedClient.builder()
                .dynamoDbClient(dynamoDbClient)
                .build();
    }
}
~~~

![Architect](/images/UPDATE/02-environment/3.infraSpring/13.png)

* Tạo file **WebConfig.java** (X-Ray Tracing Servlet Filter chuẩn Jakarta EE). Copy đoạn mã sau:

~~~java
import com.amazonaws.xray.AWSXRay;
import com.amazonaws.xray.entities.Segment;
import jakarta.servlet.*;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import org.springframework.boot.web.servlet.FilterRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.Ordered;

import java.io.IOException;
import java.util.HashMap;
import java.util.Map;

@Configuration
public class WebConfig {

    @Bean
    public FilterRegistrationBean<Filter> tracingFilter() {
        FilterRegistrationBean<Filter> registration = new FilterRegistrationBean<>();
        registration.setFilter(new Filter() {
            @Override
            public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
                    throws IOException, ServletException {
                HttpServletRequest req = (HttpServletRequest) request;
                HttpServletResponse res = (HttpServletResponse) response;

                // Bắt đầu segment X-Ray cho service
                Segment segment = AWSXRay.beginSegment("fcaj-products-service");
                try {
                    chain.doFilter(request, response);

                    if (segment != null) {
                        int status = res.getStatus();
                        Map<String, Object> responseAttributes = new HashMap<>();
                        responseAttributes.put("status", status);
                        segment.putHttp("response", responseAttributes);

                        if (status >= 400 && status < 500) {
                            segment.setError(true);
                        } else if (status >= 500) {
                            segment.setFault(true);
                        }
                    }
                } catch (Exception e) {
                    if (segment != null) {
                        segment.addException(e);
                        segment.setFault(true);
                    }
                    throw e;
                } finally {
                    AWSXRay.endSegment();
                }
            }
        });

        registration.addUrlPatterns("/*");
        registration.setOrder(Ordered.HIGHEST_PRECEDENCE);
        return registration;
    }
}
~~~

![Architect](/images/UPDATE/02-environment/3.infraSpring/14.png)

{{% notice tip %}}
Nếu bạn gặp vấn đề trong quá trình build Spring Boot (có thể do lỗi inline hoặc xung đột thư viện), bạn hãy kiểm tra:
1. Thư viện có bị trùng lặp hay không:
   - `.\mvnw dependency:tree`: Lệnh này sẽ liệt kê cây thư viện, giúp bạn kiểm tra thư viện có bị trùng lặp hay không.
2. Do cache local: thường là do cache local hoặc xung đột thư viện. Hãy thử lệnh sau:
   - `.\mvnw clean compile`: Lệnh này sẽ xoá cache local và build lại.
3. Khởi động lại VSCode.
{{% /notice %}}

**Bạn đã hoàn thành việc tạo Spring Boot Product Service (ECR). Trong các file đã bao gồm cấu hình AWS X-Ray, DynamoDB.**
