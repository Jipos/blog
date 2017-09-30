---
layout: post
title: Create custom security expression with Spring Security
---

Spring security allows you to annotate methods with @PreAuthorize and @PostAuthorize annotations. Out of the box, this allows you to authorize calls using the authorities of the authenticated user.

> e.g. @PostAuthorize("hasAuthority('FOO_READ_PRIVILEGE')")

This creates a rather tight coupling between your application code and security rules. It might be useful to separate the two. This has already been referred to in my post on ABAC.

The following post explains how you can create a custom security expression, which allows you to have less coupling between application code and security code.

> <http://www.baeldung.com/spring-security-create-new-custom-security-expression>

This approach allows you to annotation methods with security rules like:

> @PostAuthorize("hasPermission(returnObject, 'read')")
>
> or
>
> @PreAuthorize("hasPermission(#id, 'Foo', 'read')")

This way, the rules determining what has to be checked, are no longer located in the service, but in the permission evaluator.

Another approach is to create a completely new annotation. This way you can annotate your methods like:

> @PreAuthorize("isMember(#id)")

The advantage of this approach is that it serves as a kind of documentation. Now when you read the annotated service method, you immediately get a sense of what security rules are going to be applied.

The downside of this approach is that there is again a tighter coupling between application and security code. This doesn't have to be a problem, but when you want to separate the two, using the hasPermission expression might be preferable.
