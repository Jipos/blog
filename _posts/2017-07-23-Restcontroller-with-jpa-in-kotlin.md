---
layout: post
title: Spring-boot REST controller with JPA in Kotlin
---

I want to write a spring-boot application with a REST frontend and a JPA backend in Kotlin.

The following post got me started:
> <https://moelholm.com/2017/03/19/spring-boot-a-bit-more-cool-with-kotlin/>

This seems elegant enough. Although I'm not sure the Entity would work when using hibernate as a JPA implementation, since it requires a no-args constructor for it's Entities.

The following post describes another example:
> <https://blog.codecentric.de/en/2017/06/kotlin-spring-working-jpa-data-classes/>

This one is also missing the default constructor for the Entities.

The uses a CreateDTO and UpdateDTO, to avoid having a nullable ID property when you use a single DTO for both, because the create doesn't have an id and the update does.

It proposes using a UpdateField type for all properties of the UpdateDTO, to avoid nullable properties there, but this seems to be a bit ... much.

Yet another post describing the JPA side in more detail (by a colleague of the previous post's author):
> <https://blog.codecentric.de/en/2017/04/crud-operations-spring-rest-resources-kotlin/>

It describes how to handle the Entity-to-DTO conversions (in both directions).

I don't like the fact that the code uses nullable properties, since this allows the posibility of NPE's and Kotlin was supposed to get rid of these. But I don't have a better solution yet myself.
