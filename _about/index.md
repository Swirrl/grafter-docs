---
layout: page
title: The Grafter Vision
---

# The Grafter DSL

At the heart of Grafter is a Domain Specific Langauge (DSL) which
allows the specification of transformation pipelines that convert
tabular data into either more tabular data or linked data graphs.

The DSL is designed to support a good separation of concerns, and
acknowledges the reality that even within a single dataset many people
with different skills and responsibilities may be involved.

# The Grafter Vision

Grafter has been designed to form the basis of a suite of tools for
creating robust ETL processes, for both tabular and graph data.

These tools are concerned not only with transformation, but also data
cleaning, and robust loading with excellent facilities for error
handling and reporting.

Implemented in Clojure, Grafter's core is targeted at software
developers, however we are planning on building graphical tools
targeted at spreadsheet users to allow non-programmers to develop
robust, repeatable transformations.  Our vision is to help enable a
seamless collaboration between a broad spectrum of users, enabling
division of labour between transformations built by spreadsheet users
and developers.

Additionally we are developing RESTFUL service wrappers for data
loading that allow transformations to be packaged as import services.

# The Grafter Roadmap

Grafter is still in the early stages, but is already being used to
develop large, commercial grade ETL processes for large quantities of
government data.

In spite of this Grafter's DSL itself is under rapid development and
is expected to undergo radical changes in the run up to v1.0.0.

In particular we plan on investigating and potentially formalising the
following features, either in Grafter's core or in external libraries
in coming releases:

- 1st class error handling features
- Decomplecting the selection of data from the application of transformation
- DSL redesign to ensure full coverage of typical ETL operations
- Ontology loading
- Review to ensure laziness in all operations that can support it, and
  providing explicit guarantees of laziness
- Database adapters for reading/writing to SQL
- Adapters for non RDF, Graph Databases, such as Datomic and neo4j
- Integration with core.matrix
- Transducer and core.async support
- Creating a library of common, composable transformation functions

# Previous Grafter Versions

Earlier releases of Grafter should still be left available in the
public clojars repositories, and we hope to maintain their online API
documentation.

- [Latest Grafter API](http://api.grafter.org/master/)
- [Grafter v0.4.x API](http://api.grafter.org/0.4/)
- [Grafter v0.3.x API](http://api.grafter.org/0.3/)
- [Grafter v0.2.x API](http://api.grafter.org/0.2/)
