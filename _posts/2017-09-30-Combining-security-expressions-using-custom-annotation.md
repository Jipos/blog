---
layout: post
title: Create custom security expression with Spring Security
---

When you use Spring-security's @PreAuthorize and @PostAuthorize annotations as-is, out of the box, you might end up with a lot of security logic in these annotations. E.g.

```java
@PreAuthorize("(branch.isPartOf(organization) and principal.userObject.isManagerOf(#branch.id))
                or principal.userObject.isOwnerOf(#organization.id)
                or principal.userObject.administrator")
public void updateOrganizationBranch(Organization
```

There are ways to improve this, without having to resort to custom permission evaluators or even more complex things as Spring-security's ACL feature.

The following post describes several approaches

> <http://blog.novoj.net/2012/03/27/combining-custom-annotations-for-securing-methods-with-spring-security/>

The idea is to create composite security annotations, which hide the complexities of the security rules from the service class.

The end result is something which looks like this:

```java
@AllowedForAdministrator
public class MyManager {

   public void approveOrganization(Organization organization) {
      ... content ...
   }

   @AllowedForOrganizationOwner
   public void updateOrganization(Organization organization) {
      ... content ...
   }

   @AllowedForOrganizationOwner
   @AllowedForBranchManager
   public void updateOrganizationBranch(Organization organization, Branch branch) {
      ... content ...
   }

}
```

These custom annotations are easy to create and make it much clearer what the authorization rules are.
