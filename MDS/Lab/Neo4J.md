# Neo4j Cypher Quick Syntax Reference

A rapid, syntax-first study guide covering node and relationship creation, graph pattern matching, filtering, updating properties and labels, and deletion operations in Neo4j.

---

## 1. Creating Nodes (`CREATE`)

Nodes are represented using parentheses `()`.

### 1.1 Empty Nodes

````cypher
// Create a single anonymous empty node and return it
CREATE (n)
RETURN n;

// Create multiple empty nodes at once
CREATE (n), (m);
````

### 1.2 Nodes with Labels and Properties

````cypher
// Create a node with a label
CREATE (n:Person);

// Create a node with a label and properties
CREATE (n:Person {name: 'Andy', title: 'Developer'});
````

---

## 2. Creating Relationships

Relationships are represented using arrows `-->`, `<--`, or `--` with square brackets `[]`.

### 2.1 Create Nodes and Relationship Together

````cypher
// Create two nodes and the directed relationship between them in one step
CREATE (a:Person {name: 'A'})-[:friendof]->(b:Person {name: 'B'});

// Create relationship with attributes/properties
CREATE (a:Person {name: 'A'})-[r:friendof {since: 2021}]->(b:Person {name: 'B'});
````

### 2.2 Create Relationship Between Existing Nodes (`MATCH` + `CREATE`)

````cypher
// Match existing nodes first, then link them
MATCH (a:Person), (b:Person)
WHERE a.name = 'A' AND b.name = 'B'
CREATE (a)-[r:follows]->(b);
````

---

## 3. Querying & Pattern Matching (`MATCH` ... `RETURN`)

### 3.1 Fetching Nodes

````cypher
// Return all nodes in the database
MATCH (n)
RETURN n;

// Return all nodes belonging to a specific label
MATCH (n:Movie)
RETURN n;

// Return specific properties (projection)
MATCH (n:Movie)
RETURN n.title;
````

### 3.2 Finding Related Nodes & Filtering Patterns

````cypher
// Filter by node properties inside pattern
MATCH (p:Person {name: 'Ron Howard'})-[r:DIRECTED]->(m:Movie)
RETURN m.title;

// Label match without specifying relationship type/direction
MATCH (:Person {name: 'Tom Hanks'})--(movie:Movie)
RETURN movie.title;

// Directional traversal (outgoing from Person to Movie)
MATCH (:Person {name: 'Tom Hanks'})-->(movie:Movie)
RETURN movie.title;

// Incoming relationship with a specific type
MATCH (m:Movie {title: 'The Matrix'})<-[:ACTED_IN]-(actor:Person)
RETURN actor.name;
````

### 3.3 Multiple Relationship Types & Storing Edge Variables

````cypher
// Match multiple relationship types using pipe (|)
MATCH (m:Movie {title: 'The Matrix'})<-[:ACTED_IN | DIRECTED]-(p:Person)
RETURN p.name;

// Bind relationship to variable 'r' to access edge properties
MATCH (m:Movie {title: 'The Matrix'})<-[r:ACTED_IN]-(actor:Person)
RETURN r.roles;

// Multi-hop path: Find movies Tom Hanks acted in AND their directors
MATCH (p:Person {name: 'Tom Hanks'})-[:ACTED_IN]->(m:Movie)<-[:DIRECTED]-(director:Person)
RETURN m.title, director.name;
````

---

## 4. Modifying Nodes & Properties (`SET`)

### 4.1 Add or Update Properties

````cypher
// Add a new property (or update existing)
MATCH (n:Person {name: 'Andy'})
SET n.surname = 'Taylor'
RETURN n.name, n.surname;

// Update data type of an existing property
MATCH (n:Person {name: 'Andy'})
SET n.age = toString(n.age)
RETURN n.name, n.age;
````

### 4.2 Clear Property via `SET`

````cypher
// Setting a property to null deletes it from the node
MATCH (n {name: 'Andy'})
SET n.name = null
RETURN n.name, n.age;
````

---

## 5. Removing Properties & Labels (`REMOVE`)

`REMOVE` unbinds labels and properties without deleting the underlying node.

````cypher
// Remove a single property
MATCH (a {name: 'Andy'})
REMOVE a.age
RETURN a.name, a.age;

// Remove a single label
MATCH (n:German {name: 'Andy'})
REMOVE n:German
RETURN n.name, labels(n);

// Remove multiple labels at once
MATCH (n:French {name: 'Andy'})
REMOVE n:German:Swedish
RETURN n.name, labels(n);
````

---

## 6. Deleting Nodes & Relationships (`DELETE` / `DETACH DELETE`)

> **Note:** A node cannot be deleted if it still has incoming or outgoing relationships attached to it.

### 6.1 Delete Relationships Only

````cypher
// Delete only the relationship between nodes
MATCH (n:Person {name: 'A'})-[r:friendof]->()
DELETE r;
````

### 6.2 Delete a Standalone Node

````cypher
// Delete a node that has no active relationships
MATCH (n:Person {name: 'Andy'})
DELETE n;
````

### 6.3 Cascading Deletion (`DETACH DELETE`)

````cypher
// Delete a specific node along with all its attached relationships
MATCH (n:Person {name: 'A'})
DETACH DELETE n;

// Delete EVERY node and relationship in the entire database (Reset DB)
MATCH (n)
DETACH DELETE n;
````

---

## 7. Syntax Pattern Summary Cheat Sheet

| Task | Pattern Syntax |
|---|---|
| **Node pattern** | `(variable:Label {prop: 'val'})` |
| **Directed edge** | `(a)-[:REL_TYPE]->(b)` |
| **Undirected / Bidirectional edge** | `(a)--(b)` |
| **Multi-type relationship** | `(a)-[:TYPE_A | TYPE_B]->(b)` |
| **Inspect labels** | `RETURN labels(nodeVariable)` |
| **Safe node deletion** | `MATCH (n:Label) DETACH DELETE n` |
| **Drop property** | `REMOVE n.prop` or `SET n.prop = null` |
`