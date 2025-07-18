# Kotlin Pom Gradle Plugin

[![Kotlin](https://img.shields.io/badge/Kotlin-2.0.21-blue.svg)](https://kotlinlang.org/)
[![Gradle](https://img.shields.io/badge/Gradle-8.0%2B-blue.svg)](https://gradle.org/)
[![License](https://img.shields.io/badge/License-Apache--2.0-green.svg)](https://opensource.org/licenses/Apache-2.0)

A Gradle plugin that makes managing your Maven POM metadata simple and effortless.

## Features

- Define organization information once at the root project and automatically propagate to all submodules
- Selectively override specific information in submodules as needed
- Maven POM-compatible structure (groupId, artifactId, version, licenses, etc.)
- Artifact signing verification before publishing
  Add a `checkProjectArtifact` task to verify that all artifacts staged for publishing are properly signed, ensuring compliance with Maven Central requirements.

## 1. Organization-wide POM Management Options

### Option 1: Using settings.gradle.kts

Apply the plugin in your `settings.gradle.kts`:

```kotlin
plugins {
    id("io.github.yonggoose.kotlin-pom-gradle-setting") version "0.1.0"
}

rootProjectSetting {
    groupId = "io.github.yonggoose"
    artifactId = "my-project"
    version = "1.0.0"
    
    name = "My Project"
    description = "A sample project"
    url = "https://github.com/YongGoose/my-project"
    
    licenses {
        license {
            licenseType = "Apache-2.0"
        }
    }
    
    organization {
        name = "YongGoose"
        url = "https://github.com/YongGoose"
    }
    
    developers {
        developer {
            id = "yonggoose"
            name = "Yongjun Hong"
            email = "yongjunh@apache.org"
        }
    }
    
    inceptionYear = "2025"
    
    mailingLists {
        mailingList {
            name = "Developers"
            subscribe = "dev-subscribe@example.org"
            unsubscribe = "dev-unsubscribe@example.org"
            post = "dev@example.org"
            archive = "https://example.org/archive"
        }
    }
    
    issueManagement {
        system = "GitHub"
        url = "https://github.com/YongGoose/my-project/issues"
    }
    
    scm {
        url = "https://github.com/YongGoose/my-project"
        connection = "scm:git:git@github.com:YongGoose/my-project.git"
        developerConnection = "scm:git:git@github.com:YongGoose/my-project.git"
    }
}
```

In your submodule's `build.gradle.kts`, you can access these settings:

```kotlin
// Access the settings in any module
tasks.register("printProjectInfo") {
    doLast {
        val service = gradle.sharedServices
            .registrations
            .named("rootProjectSetting")
            .get()
            .service
            .get() as OrganizationDefaultsService
            
        val defaults = service.getDefaults()
        println("Project name: ${defaults.name}")
        println("Project group: ${defaults.groupId}")
        println("Project version: ${defaults.version}")
    }
}
```

### Option 2: Using build.gradle.kts

Apply the plugin in your root `build.gradle.kts`:

```kotlin
plugins {
    id("io.github.yonggoose.kotlin-pom-gradle-project") version "0.1.0"
}

rootProjectPom {
    groupId = "io.github.yonggoose"
    artifactId = "my-project"
    version = "1.0.0"
    
    name = "My Project"
    description = "A sample project"
    url = "https://github.com/YongGoose/my-project"
    
    licenses {
        license {
            licenseType = "Apache-2.0"
        }
    }
    
    organization {
        name = "YongGoose"
        url = "https://github.com/YongGoose"
    }

    developers {
        developer {
            id = "yonggoose"
            name = "Yongjun Hong"
            email = "yongjunh@apache.org"
        }
    }
    
    inceptionYear = "2025"
    
    mailingLists {
        mailingList {
            name = "Developers"
            subscribe = "dev-subscribe@example.org"
            unsubscribe = "dev-unsubscribe@example.org"
            post = "dev@example.org"
            archive = "https://example.org/archive"
        }
    }
    
    issueManagement {
        system = "GitHub"
        url = "https://github.com/YongGoose/my-project/issues"
    }
    
    scm {
        url = "https://github.com/YongGoose/my-project"
        connection = "scm:git:git@github.com:YongGoose/my-project.git"
        developerConnection = "scm:git:git@github.com:YongGoose/my-project.git"
    }
}
```

In your submodule's `build.gradle.kts`, apply the plugin and override settings as needed:

```kotlin
plugins {
    id("io.github.yonggoose.kotlin-pom-gradle-project") version "0.1.0"
}

// Override only what's needed
projectPom {
    artifactId = "my-submodule"
    description = "A submodule of my project"
    // Any other properties you need to override
}

// Access the merged values
tasks.register("printProjectInfo") {
    doLast {
        val pom = project.extensions.extraProperties.get("mergedDefaults") as OrganizationDefaults
        println("Project name: ${pom.name}")
        println("Project group: ${pom.groupId}")
        println("Project version: ${pom.version}")
    }
}
```
<details>
<summary>Data Structure</summary>

- Basic information: groupId, artifactId, version, name, description, url, inceptionYear  
- Organization information: name, url  
- License details: licenseType  
- Developers: id, name, email, url, organization, organizationUrl, timezone  
- Mailing lists: name, subscribe, unsubscribe, post, archive  
- Issue management: system, url  
- SCM: connection, developerConnection, url

</details>

## 2. Artifact Signing & POM Validation

### Artifact Signing Verification

```kotlin
plugins {
    id("io.github.yonggoose.kotlin-pom-gradle-artifact-check-project")
    id("io.github.yonggoose.kotlin-pom-gradle-project")
}

rootProjectPom {
    groupId = "io.github.yonggoose"
    artifactId = "organization-defaults"
    version = "1.0.0"

    name = "Test Organization"
    description = "Organization defaults plugin test"
    url = "https://example.org"

    licenses {
        license {
            licenseType = "MIT"
        }
        license {
            licenseType = "Apache-2.0"
        }
    }

    developers {
        developer {
            id = "dev1"
            name = "Developer1"
            email = "dev1@example.com"
            timezone = "UTC"
            organization = "YongGoose"
            organizationUrl = "https://yonggoose.github.io"
        }
    }

    scm {
        url = "https://github.com/YongGoose/organization-defaults"
        connection = "scm:git:git@github.com:YongGoose/organization-defaults.git"
        developerConnection = "scm:git:git@github.com:YongGoose/organization-defaults.git"
    }
}
```

After configuring your `build.gradle.kts` as above, simply run the following task to verify your artifacts:

```shell
./gradlew checkProjectArtifact
```

This will validate that all required POM fields and artifact signatures meet Maven Central requirements.  
If any issues are found, the build will fail with detailed error messages.

## Requirements

- Gradle 8.0 or higher
- Kotlin DSL support

## License

Apache License, Version 2.0