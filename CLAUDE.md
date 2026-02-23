# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build and Test Commands

This is a Spring Boot 3.2 application using Java 17+. Both Maven and Gradle are supported.

### Maven (Primary)
```bash
# Build the application
./mvnw package

# Run all tests
./mvnw test

# Run a single test class
./mvnw test -Dtest=OwnerControllerTests

# Run a single test method
./mvnw test -Dtest=OwnerControllerTests#testProcessUpdateOwnerFormSuccess

# Run the application
./mvnw spring-boot:run

# Build Docker image (no Dockerfile needed)
./mvnw spring-boot:build-image

# Compile CSS from SCSS (required after modifying petclinic.scss)
./mvnw package -P css
```

### Gradle
```bash
# Build the application
./gradlew build

# Run tests
./gradlew test

# Run the application
./gradlew bootRun
```

## Running the Application

### Quick Development Mode
The fastest way to run the application during development is to use the test application classes with main() methods:

1. **PetClinicIntegrationTests.main()** - Uses H2 in-memory database with Spring Boot DevTools for hot reload
2. **MysqlTestApplication.main()** - Uses Testcontainers to start MySQL in Docker
3. **PostgresIntegrationTests.main()** - Uses Docker Compose to start PostgreSQL

These can be run directly in your IDE or as tests. They provide fast feedback during development.

### Database Profiles
Default database is H2 (in-memory). To use other databases:

```bash
# MySQL
./mvnw spring-boot:run -Dspring-boot.run.profiles=mysql
docker run -e MYSQL_USER=petclinic -e MYSQL_PASSWORD=petclinic -e MYSQL_ROOT_PASSWORD=root -e MYSQL_DATABASE=petclinic -p 3306:3306 mysql:8.2

# PostgreSQL
./mvnw spring-boot:run -Dspring-boot.run.profiles=postgres
docker run -e POSTGRES_USER=petclinic -e POSTGRES_PASSWORD=petclinic -e POSTGRES_DB=petclinic -p 5432:5432 postgres:16.1

# Or use docker-compose
docker-compose --profile mysql up
docker-compose --profile postgres up
```

Access the application at http://localhost:8080 and H2 console at http://localhost:8080/h2-console.

## Architecture Overview

### Package Structure (Feature-Based)
The codebase uses **feature-based packaging** where each domain feature is self-contained:

- **owner/** - Owner management (owners, pets, visits)
  - Controllers, repositories, entities, and validators for owner-related functionality
- **vet/** - Veterinarian management
  - Controllers and repositories for vets and specialties
- **model/** - Base entity classes shared across features
  - BaseEntity, NamedEntity, Person - common domain superclasses
- **system/** - System-level concerns
  - CacheConfiguration, error handling, welcome page

Each feature package contains:
- **Controller** - Spring MVC controller (handles HTTP requests)
- **Repository** - Spring Data JPA repository interface (data access)
- **Entity classes** - JPA entities (Pet, Owner, Visit, etc.)
- **Validators** - Custom validation logic when needed

### Key Patterns

**Repositories**: Use Spring Data JPA Repository interface (not JpaRepository or CrudRepository). Custom JPQL queries are defined with `@Query` annotations. Repositories are interfaces with no implementation - Spring Data generates proxies at runtime.

**Controllers**: Standard Spring MVC pattern. Package-private visibility is preferred (no `public` modifier on controllers). Use Thymeleaf templates in `src/main/resources/templates/`.

**Validation**: Mix of JSR-303 Bean Validation annotations and custom validators. Custom validators implement Spring's `Validator` interface and are registered via `@InitBinder`.

**Caching**: Configured in CacheConfiguration using Caffeine. Example: VetRepository.findAll() is cached.

**Templates**: Thymeleaf templates in `templates/` directory with corresponding static resources (CSS, JS) in `static/resources/`.

### Database Schema
SQL schema is version-controlled in `src/main/resources/db/{database}/` with separate directories for h2, mysql, and postgres. Schema initialization is controlled by `spring.sql.init.schema-locations` and `spring.sql.init.data-locations` in application.properties.

### Testing Strategy
- **Integration tests** extend test classes that start the full Spring context
- **Testcontainers** used for MySQL integration tests (automatic Docker container management)
- **Docker Compose** used for Postgres integration tests
- Test classes with main() methods double as both integration tests and runnable applications for development

## Key Files
- **PetClinicApplication.java** - Spring Boot main class
- **application.properties** - Default H2 configuration
- **application-{profile}.properties** - Profile-specific database configuration
- **pom.xml** - Maven build configuration (primary)
- **build.gradle** - Gradle build configuration (alternative)
