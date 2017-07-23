---
layout: post
title: Immutable persistence
---

The following blog post describes a way to use immutable objects for persistence in java:
> http://vlkan.com/blog/post/2015/03/21/immutable-persistence/

It seems like an interesting idea, to differentiate between saved and unsaved (or draft) objects by using different classes.

The post uses an Map as it's datastore. I'm not sure yet how easy it would be to change this with e.g. a JPA repository. I think it might be necessary to link the Entity objects to the entityManager somehow. Maybe you need to load the current entity from the DB first, and update it accordingly based on the draft entity. This needs to be investigated further.
