# Kotlin wrapper for togglz library.

In Kotlin it's not possible to use the `Feature`-Interface for enum features as in Java, because its `name`-method clashes with the builtin `name`-method of the enum class.

Therefore, this wrapper uses a plain enum without implementing `Feature` and provides a `FeatureProvider` to wrap it into a Feature. 

# Usage (with spring)

Import dependency: 

`implementation("org.togglz:kotlin:2.7.0-SNAPSHOT")`

Create an enum for your feature toggles but don't extend the Togglz-Feature interface:

```kotlin
 enum class KotlinTestFeatures {

    @EnabledByDefault
    FOO,

    @Label("bar feature")
    BAR;

    fun isActive(): Boolean = FeatureContext.getFeatureManager().isActive { name }
   
}
```


Now, whenever you need a `Feature` you would create a Feature instance by using the name of your feature enum value as implementation: 
 `Feature { KotlinTestFeatures.BAR.name }` 


For this to work you need to create a spring configuration that creates a `FeatureManager`and a `FeatureProvider`:

```kotlin
@Configuration
class MyTogglzConfiguration {

    @Bean
    fun featureProvider() = EnumClassFeatureProvider(KotlinTestFeatures::class.java)

    @Bean
    @Primary
    fun myFeatureManager(stateRepository: StateRepository,
                              userProvider: UserProvider,
                              featureProvider: FeatureProvider): FeatureManager {

        val featureManager = FeatureManagerBuilder()
                .featureProvider(featureProvider)
                .stateRepository(stateRepository)
                .userProvider(userProvider)
                .build()

        StaticFeatureManagerProvider.setFeatureManager(featureManager)
        FeatureManagerProvider.featureMgr = featureManager
        return featureManager
    }

}
```

## Enable all toggles

for unit tests:
```kotlin
val featureManager = FeatureManagerSupport.createFeatureManagerForTest(KotlinTestFeatures::class)
FeatureManagerSupport.allEnabledFeatureConfig(featureManager)
```


for spring acceptance tests:
```kotlin
@Autowired
val featureManager: FeatureManager
//....
FeatureManagerSupport.allEnabledFeatureConfig(featureManager)
```

## Enable one toggle

```kotlin
FeatureManagerSupport.enable(Feature { KotlinTestFeatures.BAR.name })
```


# Credentials

Inspired by and copied from https://github.com/e-breuninger/spring-boot-starter-breuninger/tree/master/togglz
