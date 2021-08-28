# Ejercicio Spring HATEOAS con assemblers
Spring boot, REST básico + swagger + hateoas con assemblers y relaciones con h2

Tras hacer este [tutorial guiado](https://howtodoinjava.com/spring5/hateoas/spring-hateoas-tutorial/) de HATEOAS ahora vamos a crear un CRUD de cursos y estudiantes. Usaremos lombook para facilitar el trabajo.


![Project Structure](./images/structure.png)



### /pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>es.eoi.springboot.rest.example</groupId>
    <artifactId>spring-boot-rest-service-hateoas</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>jar</packaging>

    <name>spring-boot-rest-service-hateoas</name>
    <description>Spring Boot and REST with Swagger and HATEOAS - Example Project</description>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.1.RELEASE</version>

        <relativePath/>
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
        <maven-jar-plugin.version>3.1.1</maven-jar-plugin.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-hateoas</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-boot-starter</artifactId>
            <version>3.0.0</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

    <repositories>
        <repository>
            <id>spring-snapshots</id>
            <name>Spring Snapshots</name>
            <url>https://repo.spring.io/snapshot</url>
            <snapshots>
                <enabled>true</enabled>
            </snapshots>
        </repository>
        <repository>
            <id>spring-milestones</id>
            <name>Spring Milestones</name>
            <url>https://repo.spring.io/milestone</url>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>
    </repositories>

    <pluginRepositories>
        <pluginRepository>
            <id>spring-snapshots</id>
            <name>Spring Snapshots</name>
            <url>https://repo.spring.io/snapshot</url>
            <snapshots>
                <enabled>true</enabled>
            </snapshots>
        </pluginRepository>
        <pluginRepository>
            <id>spring-milestones</id>
            <name>Spring Milestones</name>
            <url>https://repo.spring.io/milestone</url>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </pluginRepository>
    </pluginRepositories>
</project>
```
---

# Relaciones entre las clases entity
La relación entre cursos y estudiantes será también de M:M

```java
package es.eoi.springboot.rest.example.entity;

//...

@Data
@Builder
@AllArgsConstructor
@NoArgsConstructor
@ToString(exclude = "courses")
@Entity
public class Student implements Serializable {

    //...
    
    @ManyToMany(cascade=CascadeType.ALL)
    @JoinTable(
            name = "student_course",
            joinColumns = @JoinColumn(name = "student_id"),
            inverseJoinColumns = @JoinColumn(name = "course_id"))
    private List<Course> courses;

}

```
---

/src/main/java/es/eoi/springboot/rest/example/entity/Course.java
```java
package es.eoi.springboot.rest.example.entity;

//...

@Data
@Builder
@AllArgsConstructor
@NoArgsConstructor
@ToString(exclude = "students")
@Entity
public class Course {

    //...

    @ManyToMany(mappedBy = "courses", fetch = FetchType.EAGER)
    private List<Student> students;
}
```
---

# Excepciones controladas
Si no encontramos un estudiante lanzaremos una excepción `StudentNotFoundException`

# Repositorios
Extenderemos de JpaRepository al igual que en el ejemplo

# REST
Tanto para los cursos como para los estudiantes tendremos un CRUD completo.

# Initial data
Podéis utilizar estos datos como base. Podéis añadir más. 
/src/main/resources/data.sql
```

insert into student values(10001,'Jose Francisco', 'E1234567');
insert into student values(10002,'Maria Angeles', 'A1234568');

insert into course values(1001,'Programación', 'Spring');
insert into course values(1002,'Inglés I', 'Inglés básico');
insert into course values(1003,'Inglés II', 'Inglés medio');
insert into course values(1004,'Inglés II', 'Inglés avanzado');

insert into student_course values (10001, 1001);
insert into student_course values (10001, 1003);
insert into student_course values (10002, 1001);
insert into student_course values (10002, 1004);
```
---