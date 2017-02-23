---
layout: post
title:  "Test DSL for C#"
date:   2017-01-10 12:55:31 +0200
categories: .NET fsharp
---

Approach for building test factories

1. Adhoc. No any specific infrastructure is created. All setup is done in test class methods.
This is OK for relatively simple test data setup. 

In some cases one might need some more sophisticated setup. Let's consider some workflow, for
example -- order in online shop. One test verifies how adding items to cart works. While other 
need to test checkout. To test checkout order should have some items added to it. Being good 
developers we tend to use DRY principle and reuse code adding items to cart. 

1 Tests calling Tests
2. Tests calling methods 

