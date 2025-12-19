# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an OpenRewrite recipe project for upgrading QueryDSL dependencies from legacy `com.mysema.querydsl` to the modern `com.querydsl` namespace, with Jakarta compatibility. OpenRewrite recipes are automated code transformation tools that can be applied to large codebases.

### Current State

The repository has been stripped of its example recipes - all Java source files in `src/main/java/com/yourorg/` and corresponding tests have been deleted. Only the YAML recipe definition files remain:
- `src/main/resources/META-INF/rewrite/rewrite.yml` - Main QueryDSL upgrade recipe
- `src/main/resources/META-INF/rewrite/examples.yml` - Example transformations (for deleted recipes)

## Build & Testing Commands

**Build with Gradle:**
```bash
./gradlew build
./gradlew test
./gradlew test --tests <TestClassName>  # Run single test
```

**Build with Maven:**
```bash
./mvnw clean install
./mvnw test
./mvnw test -Dtest=<TestClassName>  # Run single test
```

**Publish locally for testing:**
```bash
./gradlew publishToMavenLocal
# or
./mvnw install
```

## Project Structure

- `build.gradle.kts` - Gradle build configuration with OpenRewrite plugin setup
- `pom.xml` - Maven build configuration
- `src/main/resources/META-INF/rewrite/` - YAML recipe definitions
- `src/main/java/` - Imperative recipe implementations (currently empty)
- `src/test/java/` - Recipe test classes (currently empty)
- `init.gradle` - Gradle init script for applying best practices recipes

## OpenRewrite Concepts

**Recipes** are the core transformation units. They come in three styles:

1. **YAML recipes** - Declarative recipes that compose other recipes (currently the only recipes in this project)
2. **Imperative Java recipes** - Custom logic using OpenRewrite's AST visitor pattern (formerly examples are deleted)
3. **Refaster recipes** - Template-based recipes using `@BeforeTemplate`/`@AfterTemplate` annotations

**Testing recipes** uses `RewriteTest` base class. Tests verify transformations with `before` and `after` code snippets.

## Key Dependencies

- `org.openrewrite:rewrite-java` - Core Java transformation engine
- `org.openrewrite.recipe:rewrite-recipe-bom` - Managed dependencies for recipe development
- `org.openrewrite:rewrite-test` - Testing framework for recipes
- `org.openrewrite:rewrite-templating` - Refaster annotation processor
- `org.openrewrite:rewrite-yaml` / `rewrite-xml` - Other language support

## Recipe Development Guidelines

When implementing new recipes, follow these principles in order of preference:

### 1. Write Tests First
Always write test cases before implementing a recipe. Tests use the `RewriteTest` base class with `before` and `after` code snippets.

### 2. Prefer Declarative YAML Recipes
Create small, purpose-built recipes and compose them together using declarative YAML. This leverages OpenRewrite's library of building-block recipes and is the most maintainable approach.

**Example approach:**
- Use existing recipes from `org.openrewrite.maven.*`, `org.openrewrite.java.*`, etc.
- Combine multiple recipes in a single YAML recipe definition
- This is preferred because it's composable and reusable

### 3. Use Refaster-Style Recipes When Needed
If composition isn't possible, prefer Refaster-style recipes using `@BeforeTemplate` and `@AfterTemplate` annotations. These are template-based and easier to understand than imperative visitors.

### 4. Write Imperative Java Recipes Only As Last Resort
Only implement custom Java recipes with imperative visitor logic when neither YAML composition nor Refaster templates can achieve the transformation.

### Reference Implementations

Study well-written recipes in:
- `/Users/matt/projects/openrewrite/recipes/` - Official OpenRewrite recipes
- `/Users/matt/projects/moderne/recipes/` - Production recipes from Moderne

These demonstrate best practices for recipe structure, testing patterns, and composition strategies.

## Writing OpenRewrite Recipes

For comprehensive guidance, refer to the [OpenRewrite documentation](https://docs.openrewrite.org/), particularly:
- [Recipes](https://docs.openrewrite.org/concepts-and-explanations/recipes)
- [Lossless Semantic Trees](https://docs.openrewrite.org/concepts-and-explanations/lossless-semantic-trees)
- [Visitors](https://docs.openrewrite.org/concepts-and-explanations/visitors)
- [Recipe Development Environment](https://docs.openrewrite.org/authoring-recipes/recipe-development-environment)

## Applying Best Practices

To apply OpenRewrite's own recipe best practices:

```bash
./gradlew --init-script init.gradle rewriteRun \
  -Drewrite.activeRecipe=org.openrewrite.recipes.rewrite.OpenRewriteRecipeBestPractices
```

or with Maven:

```bash
./mvnw -U org.openrewrite.maven:rewrite-maven-plugin:run \
  -Drewrite.recipeArtifactCoordinates=org.openrewrite.recipe:rewrite-rewrite:RELEASE \
  -Drewrite.activeRecipes=org.openrewrite.recipes.rewrite.OpenRewriteRecipeBestPractices
```

## Local Testing

To test recipes locally on another project:

**Maven:** Add to that project's pom.xml:
```xml
<plugin>
  <groupId>org.openrewrite.maven</groupId>
  <artifactId>rewrite-maven-plugin</artifactId>
  <version>RELEASE</version>
  <configuration>
    <activeRecipes>
      <recipe>io.moderne.UpgrateToQueryDsl5</recipe>
    </activeRecipes>
  </configuration>
  <dependencies>
    <dependency>
      <groupId>org.openrewrite.recipe</groupId>
      <artifactId>rewrite-querydsl</artifactId>
      <version>0.1.0-SNAPSHOT</version>
    </dependency>
  </dependencies>
</plugin>
```

**Gradle:** Add to that project's build.gradle.kts:
```kotlin
plugins {
  id("org.openrewrite.rewrite") version("latest.release")
}

repositories {
  mavenLocal()
  mavenCentral()
}

dependencies {
  rewrite("org.openrewrite.recipe:rewrite-querydsl:latest.integration")
}

rewrite {
  activeRecipe("io.moderne.UpgrateToQueryDsl5")
}
```

Then run: `mvn rewrite:run` or `./gradlew rewriteRun`

## Important Notes

- Recipes are compiled to target Java 8 (for legacy code compatibility), but tests compile to Java 21 for access to text blocks
- The project uses Gradle as the primary build tool, with Maven as an alternative
- Recipe versions follow Maven snapshot/release conventions managed by Nebula release plugin
