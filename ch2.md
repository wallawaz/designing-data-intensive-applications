# Data Models and Query Languages

## Relational
Edgar Codd in 1970.
Data organized into relations (tables). Relation contains unordered set of tuples (rows).
  - RDMS; SQL
  - critisism: OOP already treats data as objects; awkard translation layer between application layer and db model of rows.
  - Modern SQL supports more complex data-types; JSON, XML, etc.
  - schema on write.
  - decarative query language (SQL) - specify pattern of data you want; conditions the results meet; and any transformations. **not how** to achieve this.

## Document
NotOnlySQL. Need for greater scalability than relation can easily achieve.
  - often lack of schema constraints of write. (schema-on-read).
  - sometimes lack ability to of joins. (all realted data is stored directly on the document anyway)
    - duplicating data
  - if application needs large part of the record at the same time; noSQL document model may be advantageous.

  - Highly interconnected data - document model is awkward. many-many relationships better handled in relational.

## Graph-like
  - many-to-many are very common in the data. Social Graphs, web graph, road/rail networks.
  - vertices (nodes, entities)
  - edges (relationships, arcs)

### Property Graphs
  - vertex
    - unique id
    - set-->(edges)
    - set<--(edges)
    - collection of properties (k/v pairs)
  - edge
    - uqique id
    - vertex at which edge starts (tail)
    - vertex at which edge ends (head)
    - label to describe the relationship between vertexes
    - collection of properties (k/v pairs).

### Triple Stores and SPARQL
 - (subject, predictate, object); e.g. (Ben, likes, sushi)
 - there was an initiative for websites to publish their info using RDFs.
   - parsable by SPARQL triple-store query lang.