---
layout: page
title: Transformations and the art of preserving source data
---

# General

## Transformations and the art of preserving source data

You probably don't really want to modify source data, there are a lot of reasons why:

- Source data can be used for something else
- If there is a problem you prefer to know who modified data and why
- Responsability
- You never really know how your code is going to evolve
- You never really know how your data usage is going to evolve
- And so on...

Grafter philosophy is to **give you the freedom to choose not to alter source data!**

Let's look at this dataset:

![transformations](/assets/101_tranformation_1.png)

Imagine that we need to get the App uri. One obvious (but bad) idea would be to modify the first column:

![transformations](/assets/101_tranformation_2.png)

Whereas Grafter's philosophy is to create a new column with the uri:

![transformations](/assets/101_tranformation_3.png)
