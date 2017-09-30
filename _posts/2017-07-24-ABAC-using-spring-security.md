---
layout: post
title: ABAC using spring security
---

Recently I've been investigated the ways I can use spring-security to clean up the authorization code in one of our applications. Currently the access control code is spread out all over the code base. And it's a mess.

What I'm trying to do is to to annotate the service methods with spring-security's @PreAuthorize or @PostAuthorize methods, and delegate the access control decisions to "something". The idea is to ask that "something" a generic question like "Can the current user READ object X?" Only 4 questions would be asked, one for each CRUD operation.

A first idea was to use an ACL model. In this approach, each object would have an ACL and each user would have a list of granted authorities. If one of the user's granted authorities matches an item in the objects' ACL, access would be granted. One downside of this approach is that the ACL can get long (depending on how you implement it). Another is that it can become unclear what the access rules actually are.

ABAC seemed like a candidate to fix these problems.

The wikipedia article on ABAC give a concise overview on the different components which are typically needed (e.g. PEP, PDP and PIP):
> <https://en.wikipedia.org/wiki/Attribute-based_access_control>

The following video describes a framework (by Axiomatics), which integrates with spring-security, for doing ABAC:
> <https://www.youtube.com/watch?v=TZECXS1tlGk>

Unfortunately the framework and the spring-security integration is not open-source (or free).

Axiomatics wrote a short article on ABAC, which indicates how the attributes are used to specify richer access control policies:
> <https://www.axiomatics.com/attribute-based-access-control/>

Axiomatics has a lot of other articles and presentations on this topic.

* On why RBAC is not a good fit when you have complex rules, and how ABAC can help
> <https://www.axiomatics.com/resources/toxic-pairs-role-explosion-and-the-evolution-of-rbac/>
* A talk on Axiomatics' Java PEP SDK. Although most of the talk is aboud the SDK itself, it starts with an overview on the ABAC architecture. This overview is the interesting part of the presentation. (since the PEP SDK isn't free and we'll thus won't be using it).
> <https://www.axiomatics.com/resources/boot-camp-creating-a-policy-enforcement-point/>
* As the name of this talk ("ABAC 101") suggests, it's an introduction to ABAC. It contains a lot of useful information.
> <https://www.axiomatics.com/resources/axiomatics-back-to-basics-abac-101/>
* This talk lists a number of (Axiomatics) customer RFCs and describes what they came up with to accomodate them. One of these solutions is the policy auditor. It seems very useful to be able to ask questions like "What can a teacher, belonging to college X to in our application?" and get an answer to that question (in the form of a list of all the actions the teacher can take on which entities). Another interesting solution was the policy authoring tool. It provides a visual way to not only view policies, but to create new ones too. This would allow not only developers, but each member of our team to write/update/verify these policies. This would be very benificial. This reminded me of PearlChain's ShapeIt solution (which allows you to model just about anything).
> <https://www.axiomatics.com/resources/proving-compliance-throughout-the-abac-lifecycle/>
* This talk is about testing abac policies. The most interesting thing about this presentation is it's mentioning of the alpha language. Usually XACML is referenced when talking about ABAC. XACML is rather difficult to read/write though. I might be wrong about this, it seems like it should be possible to use ALFA as a replacement for XACML. The result would be more readable policies.
> <https://www.axiomatics.com/resources/axiomatics-boot-camp-testing-xacml-policies-using-junit-eclipse-and-the-abbreviated-language-for-authorization-alfa/>
