# kotlin-spring-feign-client-generator

Templates for generating feign client interfaces for kotlin.

At the moment this repo was created there was no feign client generator in OpenAPI Generator (5.4.0) but a very capable
kotlin-spring version with spring-boot library which I used to generate server stubs. The API interfaces generated for
delegate pattern looked already close to perfect for feign, so I decided to adapt the template a bit instead of going
for a complete new generator.

Most files in the `template` directory simply empty the generated file with a comment which file was responsible for
doing so. Only `apiInterface.mustache` required some small changes to be suitable for feign.

The original files can be found
under https://github.com/OpenAPITools/openapi-generator/tree/master/modules/openapi-generator/src/main/resources/kotlin-spring
.

Usage:
 - Copy the templates folder into your project.
 - Configure your openApiGenerate task to use the templateDir.
 - Make sure to use the delegate pattern.

Here is a sample taken from a working project. All the `openAPI*` variables need to be replaced with your own values.

```kotlin
plugins {
    id("org.openapi.generator") version "5.4.0"
}

openApiGenerate {
    // https://openapi-generator.tech/docs/generators/kotlin-spring
    generatorName.set("kotlin-spring")
    library.set("spring-boot")
    templateDir.set("$projectDir/$openAPISpecFilePath/templates")

    inputSpec.set("$projectDir/$openAPISpecFilePath/$openAPISpecFileName.$openAPISpecFileExt")
    outputDir.set("$buildDir/$openAPIGenOutBase")

    packageName.set(openAPIBasePackage)
    apiPackage.set("$openAPIBasePackage.api")
    invokerPackage.set("$openAPIBasePackage.client")
    modelPackage.set("$openAPIBasePackage.model")

    modelNameSuffix.set("Dto")
    typeMappings.set(
        mapOf(
            "DateTime" to "Instant"
        )
    )
    importMappings.set(
        mapOf(
            "Instant" to "java.time.Instant"
        )
    )
    configOptions.set(
        mapOf(
            "useBeanValidation" to "true",
            "delegatePattern" to "true",
            "useTags" to "true",
            "exceptionHandler" to "false",
            "gradleBuildFile" to "false",
            "sortModelPropertiesByRequiredFlag" to "true",
            "sortParamsByRequiredFlag" to "true",
        )
    )
}

val openApiClean = tasks.create<Delete>("openApiClean") {
    group = "openapi tools"
    delete("$openApiSrcDir/${openAPIBasePackage.replace(".", "/")}")
}

val openApiGenerateTask = tasks.getByName("openApiGenerate")
val compileKotlinTask = tasks.getByName("compileKotlin")

openApiGenerateTask.dependsOn(openApiClean)
compileKotlinTask.dependsOn(openApiGenerateTask)
```
