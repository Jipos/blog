---
layout: post
title: Authorization using ACL with spring security
---

A previous post mentioned that we could separate application code from security code by using the hasPermission security expression.

We can implement our own permissionEvaluator. But Spring-security also provides it's own implementation: AclPermissionEvaluator. This evaluator allows you to "do" access control using an ACL.

The following post contains a comprehensive explanation of how to setup ACL based security using spring security.

> <https://www.denksoft.com/articles/acl-spring-security-tutorial/>

The post contains a diagram showing how all ACL DB tables are related. This will help when trying to understand how the ACL classes (which correspond to the DB tables) are linked.

Concerning ACL management, it mentions that the process of assigning permissions to users, can be automated. This might be preferable, since doing it by hand, might lead to mistakes. Although the "automated" process they mention still seems to involve a custom class for creating the permissions.

In the end, the setup which is detailed, allows you to secure your service methods like this:

```java
@Secured({"ROLE_CUSTOMER","AFTER_ACL_READ"})
public Customer getCustomer(long id);
```

Because the post is more than 4 years old, part (or a lot) of the setup might currently be handled for you by the spring-boot-starter-security.

Maybe the way this post describes it, is not quite the way I would like to do it, but it's still worth reading since it mentions a lot of ACL related classes and methods.

Starting from the spring-security reference documentation might be a better approach. Though it is rather tough to get your head around at first.

> <https://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/#domain-acls>

The spring-security/samples directory contains some examples, but they are rather basic.

> <https://github.com/spring-projects/spring-security/tree/master/samples/xml/contacts>
> <https://github.com/spring-projects/spring-security/tree/master/samples/xml/dms>

Browsing the ACL related source code is also helpfull. This helps you see connections between the different components.
