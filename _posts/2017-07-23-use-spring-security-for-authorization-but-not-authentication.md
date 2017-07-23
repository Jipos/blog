---
layout: post
title: Use Spring-security for authorization but not authentication
---

We have an application, which uses Shibboleth for authentication. We want to use spring-security for authorizaion within the application. The default spring-security assumes it it responsible for handling the authentication too.

Spring-security provides _pre-authentication scenarios_, for use cases like this.
> https://docs.spring.io/spring-security/site/docs/current/reference/html/preauth.html

In this scenario, we register a specific 'preauthentication' filter, which assumes that the user has been authenticated and the user's credentials are available somehow (e.g. as a request header or a request attribute), and uses this information to populate the SecurityContext.

## Proof of concept

Create a new, empty, spring-boot project (http://start.spring.io/).
For our example, it needs to include the spring-boot-starter-web and spring-boot-starter-security as dependencies.

By default, spring-boot secures itself using basic auth. This means that it uses basic auth for authentication. Because we don't want our application to handle authentication, we disable this feature
```
security.basic.enabled=false
```

In our case, when the user is authenticated, a 'kulmoreunifieduid' header gets added to the request. The Shibboleth daemon make sure that this header only gets set when the user is authenticated. When the user is not authenticated and he adds this header himself, this header gets removed from the request.

For the pre-authentication scenario to work we need to add a RequestHeaderAuthenticationFilter to our application context. The RequestHeaderAuthenticationFilter needs to retrieve the user details from a UserDetailsService. We need to create some intermediate objects (UserDetailsByNameServiceWrapper, PreAuthenticatedAuthenticationProvider and ProviderManager) to link the two together.

**NOTE:*
Don't add the intermediate objects as beans to your application context. This would override the default authentication manager, created by spring-boot, and break some of the default security setup.

The code below uses a dummy UserDetailsService, which doesn't set a password and uses a default "USER" GrantedAuthority, but it is good enough for a POC.

```kotlin
@SpringBootApplication
class TolBizzV2Application {

    @Bean
    fun kulmoreunifieduidPreauthFilter(): RequestHeaderAuthenticationFilter {
        val userDetailsService: AuthenticationUserDetailsService<PreAuthenticatedAuthenticationToken> =
                UserDetailsByNameServiceWrapper(UsernameOnlyUserDetailsService())
        val authenticationProvider = PreAuthenticatedAuthenticationProvider()
        authenticationProvider.setPreAuthenticatedUserDetailsService(userDetailsService)
        val authenticationManager = ProviderManager(listOf(authenticationProvider))
        val filter = RequestHeaderAuthenticationFilter()
        filter.setAuthenticationManager(authenticationManager)
        filter.setPrincipalRequestHeader("kulmoreunifieduid")
        return filter
    }

}

fun main(args: Array<String>) {
    SpringApplication.run(TolBizzV2Application::class.java, *args)
}

class UsernameOnlyUserDetailsService : UserDetailsService {

    override fun loadUserByUsername(username: String?) =
            User("$username", "", listOf(MyGrantedAuthority("USER")))

}

class MyGrantedAuthority(@JvmField val authority: String): GrantedAuthority {
    override fun getAuthority() = authority
    override fun toString() = authority
}
```

With this setup, the securityContext gets populated correctly when the 'kulmoreunifieduid' header is found. This can be tested using the following controller

```kotlin
@RestController
class MyController {

    @GetMapping("/user")
    fun getUser(@AuthenticationPrincipal principal: User): String {
        return principal.name
    }

}
```
