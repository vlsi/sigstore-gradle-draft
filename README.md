[![GitHub CI](https://github.com/vlsi/sigstore-gradle-draft/actions/workflows/build.yml/badge.svg?branch=main)](https://github.com/vlsi/sigstore-gradle-draft/actions/workflows/build.yml)

sigstore-gradle
---------------

A Gradle plugin for signing artifacts with Sigstore.

Requirements
------------

Java 11 (https://github.com/sigstore/sigstore-java requires Java 11)
Gradle 7.5 (Gradle 6 could be supported once https://github.com/jsonschema2dataclass/js2d-gradle/issues/401 is released)

## Minimail usage

```kotlin
plugins {
    id("dev.sigstore.sign")
}

// It would automatically sign all Maven publications
// By default, it would use GitHub Actions OIDC when available,
// and it would resort to Web Browser OIDC otherwise.
```

## Full configuration

```kotlin
plugins {
    id("dev.sigstore.sign")
}

dependencies {
    // Override sigstore-java clients
    sigstoreClient("dev.sigstore:sigstore-java:1.1")
}

sigstoreSign {
    oidcClient {
        gitHub {
            audience.set("sigstore")
        }
        web {
            clientId.set("sigstore")
            issuer.set("https://oauth2.sigstore.dev/auth")
        }
        // By default, gitHub client is used if ACTIONS_ID_TOKEN_REQUEST_URL environment variable exists
        // This setting would enforce web OIDC client
        client.set(web)
    }
}
```

## How to

### Sign a single file

```kotlin
dev.sigstore.sign.tasks.SigstoreSignFilesTask

val helloProps by tasks.register<WriteProperties> {
    outputFile = file("build/helloProps.txt")
    property("helloProps", "world")
}

val signHelloProps by tasks.register<SigstoreSignFilesTask> {
    // outputFile is File, so helloProps.map {..} is Provider<File>
    signFile(helloProps.map { it.outputFile })
    // Alternative APIs are
    // sign(File)
    // sign(Provider<RegularFile>)
}

val zip by tasks.regster<Zip> {
    from(signHelloProps.map { it.singleSignature() })
}
```

## Technical details

### dev.sigstore.sign plugin

Automatically signs all Maven publications in Sigstore.

### dev.sigstore.sign-base plugin

Provides `SigstoreSignFilesTask` task for signing files in Sigstore.
The plugin adds no tasks by default.

Extensions:
* `sigstoreSign`: `dev.sigstore.sign.SigstoreSignExtension`

  Configures signing parameters

  * `oidcClient`: `dev.sigstore.sign.OidcClientExtension`

    Configures OIDC token source.

    Supported sources: web browser, GitHub Actions.

Configurations:
* `sigstoreClient`

  A configuration to declare the version for `sigstore-java`.

* `sigstoreClientClasspath`

  A configuration that resolves `sigstore-java` dependencies.

Tasks:

* `dev.sigstore.sign.SigstoreSignFilesTask`

  Signs entries via Sigstore.


