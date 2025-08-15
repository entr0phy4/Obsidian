# Neo4j

## Overview
Neo4j is a native graph database management system that stores data in nodes and relationships rather than tables. It uses the Cypher query language and is designed for applications that require complex relationship modeling and traversal.

## Graph Database Concepts

### Core Components
- **Nodes**: Entities in the graph (people, products, locations)
- **Relationships**: Connections between nodes with direction and type
- **Properties**: Key-value pairs attached to nodes and relationships
- **Labels**: Tags that group nodes into sets

### Graph Model
```
(Person:User {name: "Alice", age: 30})-[:FRIENDS_WITH {since: 2020}]->(Person:User {name: "Bob", age: 25})
```

## Installation

### Docker Installation
```bash
# Run Neo4j in Docker
docker run -d \
    --name neo4j \
    -p 7474:7474 -p 7687:7687 \
    -e NEO4J_AUTH=neo4j/password \
    -v neo4j_data:/data \
    -v neo4j_logs:/logs \
    neo4j:latest

# Access browser interface at http://localhost:7474
```

### Ubuntu/Debian Installation
```bash
# Add Neo4j repository
wget -O - https://debian.neo4j.com/neotechnology.gpg.key | sudo apt-key add -
echo 'deb https://debian.neo4j.com stable 4.4' | sudo tee /etc/apt/sources.list.d/neo4j.list

# Install Neo4j
sudo apt update
sudo apt install neo4j

# Start Neo4j service
sudo systemctl enable neo4j
sudo systemctl start neo4j
```

### Configuration
```bash
# Configuration file location
/etc/neo4j/neo4j.conf

# Key configuration options
dbms.default_listen_address=0.0.0.0
dbms.connector.bolt.listen_address=:7687
dbms.connector.http.listen_address=:7474
dbms.memory.heap.initial_size=1G
dbms.memory.heap.max_size=2G
```

## Cypher Query Language

### Basic Syntax
```cypher
// Comments start with //

// Node syntax
(variable:Label {property: value})

// Relationship syntax
-[:RELATIONSHIP_TYPE {property: value}]->
<-[:RELATIONSHIP_TYPE]-
-[:RELATIONSHIP_TYPE]-

// Complete pattern
(a:Person {name: "Alice"})-[:KNOWS]->(b:Person {name: "Bob"})
```

### CREATE Operations
```cypher
// Create nodes
CREATE (p:Person {name: "Alice", age: 30, city: "New York"})

// Create multiple nodes
CREATE 
  (a:Person {name: "Alice", age: 30}),
  (b:Person {name: "Bob", age: 25}),
  (c:Company {name: "TechCorp", founded: 2010})

// Create relationships
MATCH (a:Person {name: "Alice"}), (b:Person {name: "Bob"})
CREATE (a)-[:FRIENDS_WITH {since: 2020}]->(b)

// Create nodes and relationships together
CREATE (a:Person {name: "Charlie", age: 28})
-[:WORKS_FOR {position: "Developer", since: 2022}]->
(c:Company {name: "StartupInc"})
```

### MATCH and WHERE
```cypher
// Find all persons
MATCH (p:Person) RETURN p

// Find person by name
MATCH (p:Person {name: "Alice"}) RETURN p

// Using WHERE clause
MATCH (p:Person) 
WHERE p.age > 25 
RETURN p.name, p.age

// Pattern matching with relationships
MATCH (p:Person)-[:WORKS_FOR]->(c:Company)
RETURN p.name, c.name

// Complex patterns
MATCH (p:Person)-[:FRIENDS_WITH]-(friend)-[:WORKS_FOR]->(c:Company)
WHERE p.name = "Alice"
RETURN friend.name, c.name
```

### UPDATE Operations
```cypher
// Set properties
MATCH (p:Person {name: "Alice"})
SET p.age = 31, p.email = "alice@example.com"

// Add labels
MATCH (p:Person {name: "Alice"})
SET p:VIP:Customer

// Update with conditions
MATCH (p:Person)
WHERE p.age > 30
SET p.category = "Senior"

// Remove properties
MATCH (p:Person {name: "Alice"})
REMOVE p.email

// Remove labels
MATCH (p:Person {name: "Alice"})
REMOVE p:VIP
```

### DELETE Operations
```cypher
// Delete nodes (must delete relationships first)
MATCH (p:Person {name: "Alice"})-[r]-()
DELETE r, p

// Delete relationships only
MATCH (a:Person {name: "Alice"})-[r:FRIENDS_WITH]->(b:Person {name: "Bob"})
DELETE r

// Detach delete (removes node and all relationships)
MATCH (p:Person {name: "Alice"})
DETACH DELETE p

// Conditional delete
MATCH (p:Person)
WHERE p.age < 18
DETACH DELETE p
```

## Advanced Queries

### Aggregation
```cypher
// Count nodes
MATCH (p:Person) RETURN count(p)

// Group by and aggregate
MATCH (p:Person)-[:WORKS_FOR]->(c:Company)
RETURN c.name, count(p) as employee_count

// Statistical functions
MATCH (p:Person)
RETURN 
  avg(p.age) as average_age,
  min(p.age) as youngest,
  max(p.age) as oldest

// Collect values
MATCH (p:Person)-[:FRIENDS_WITH]->(friend)
WHERE p.name = "Alice"
RETURN p.name, collect(friend.name) as friends
```

### Path Finding
```cypher
// Shortest path
MATCH path = shortestPath((alice:Person {name: "Alice"})-[*]-(target:Person {name: "Charlie"}))
RETURN path

// All shortest paths
MATCH paths = allShortestPaths((alice:Person {name: "Alice"})-[*]-(target:Person {name: "Charlie"}))
RETURN paths

// Variable length relationships
MATCH (alice:Person {name: "Alice"})-[:FRIENDS_WITH*1..3]-(connected)
RETURN DISTINCT connected.name

// Path with conditions
MATCH path = (alice:Person {name: "Alice"})-[:FRIENDS_WITH*]-(target)
WHERE ALL(node in nodes(path) WHERE node.age > 20)
RETURN path
```

### Complex Patterns
```cypher
// Optional match
MATCH (p:Person)
OPTIONAL MATCH (p)-[:HAS_PHONE]->(phone:Phone)
RETURN p.name, phone.number

// Union queries
MATCH (p:Person) RETURN p.name as name, "Person" as type
UNION
MATCH (c:Company) RETURN c.name as name, "Company" as type

// Case expressions
MATCH (p:Person)
RETURN p.name,
  CASE 
    WHEN p.age < 25 THEN "Young"
    WHEN p.age < 50 THEN "Middle-aged"
    ELSE "Senior"
  END as age_group
```

## Indexing and Performance

### Create Indexes
```cypher
// Create index on property
CREATE INDEX person_name_index FOR (p:Person) ON (p.name)

// Create composite index
CREATE INDEX person_name_age_index FOR (p:Person) ON (p.name, p.age)

// Create full-text index
CREATE FULLTEXT INDEX person_fulltext FOR (p:Person) ON EACH [p.name, p.description]

// Show indexes
SHOW INDEXES
```

### Constraints
```cypher
// Unique constraint
CREATE CONSTRAINT person_email_unique FOR (p:Person) REQUIRE p.email IS UNIQUE

// Node key constraint (multiple properties must be unique together)
CREATE CONSTRAINT person_name_birth_key FOR (p:Person) REQUIRE (p.name, p.birth_date) IS NODE KEY

// Existence constraint
CREATE CONSTRAINT person_name_exists FOR (p:Person) REQUIRE p.name IS NOT NULL

// Show constraints
SHOW CONSTRAINTS
```

### Query Optimization
```cypher
// Use EXPLAIN to see query plan
EXPLAIN MATCH (p:Person {name: "Alice"})-[:FRIENDS_WITH]->(friend) RETURN friend

// Use PROFILE for detailed execution statistics
PROFILE MATCH (p:Person {name: "Alice"})-[:FRIENDS_WITH]->(friend) RETURN friend

// Optimize with indexes
CREATE INDEX person_name FOR (p:Person) ON (p.name)
MATCH (p:Person {name: "Alice"}) RETURN p  // Now uses index
```

## Data Modeling Examples

### Social Network
```cypher
// Users and relationships
CREATE 
  (alice:User {name: "Alice", email: "alice@example.com"}),
  (bob:User {name: "Bob", email: "bob@example.com"}),
  (charlie:User {name: "Charlie", email: "charlie@example.com"}),
  (post1:Post {title: "Hello World", content: "My first post"}),
  (post2:Post {title: "Neo4j rocks", content: "Learning graph databases"})

CREATE 
  (alice)-[:FRIENDS_WITH {since: "2020-01-01"}]->(bob),
  (alice)-[:FRIENDS_WITH {since: "2021-06-15"}]->(charlie),
  (alice)-[:AUTHORED]->(post1),
  (bob)-[:AUTHORED]->(post2),
  (charlie)-[:LIKED {timestamp: datetime()}]->(post1)
```

### Recommendation Engine
```cypher
// Find friends of friends who aren't already friends
MATCH (user:User {name: "Alice"})-[:FRIENDS_WITH]->(friend)-[:FRIENDS_WITH]->(fof:User)
WHERE NOT (user)-[:FRIENDS_WITH]-(fof) AND user <> fof
RETURN fof.name, count(*) as mutual_friends
ORDER BY mutual_friends DESC

// Product recommendations based on similar users
MATCH (user:User {name: "Alice"})-[:PURCHASED]->(product:Product)
<-[:PURCHASED]-(similar:User)-[:PURCHASED]->(recommendation:Product)
WHERE NOT (user)-[:PURCHASED]->(recommendation)
RETURN recommendation.name, count(*) as score
ORDER BY score DESC
```

### Organizational Hierarchy
```cypher
// Company structure
CREATE 
  (ceo:Employee {name: "John CEO", position: "CEO"}),
  (cto:Employee {name: "Jane CTO", position: "CTO"}),
  (dev1:Employee {name: "Dev One", position: "Senior Developer"}),
  (dev2:Employee {name: "Dev Two", position: "Junior Developer"})

CREATE 
  (cto)-[:REPORTS_TO]->(ceo),
  (dev1)-[:REPORTS_TO]->(cto),
  (dev2)-[:REPORTS_TO]->(dev1)

// Find all subordinates
MATCH (manager:Employee {name: "Jane CTO"})<-[:REPORTS_TO*]-(subordinate:Employee)
RETURN subordinate.name, subordinate.position
```

## Administration

### User Management
```cypher
// Create user
CREATE USER alice SET PASSWORD 'password123'

// Grant privileges
GRANT ROLE reader TO alice
GRANT ACCESS ON DATABASE neo4j TO alice

// Show users
SHOW USERS

// Change password
ALTER USER alice SET PASSWORD 'newpassword456'
```

### Database Management
```cypher
// Show databases
SHOW DATABASES

// Create database
CREATE DATABASE myapp

// Use database
:USE myapp

// Drop database
DROP DATABASE myapp
```

### Backup and Restore
```bash
# Dump database
neo4j-admin dump --database=neo4j --to=backup.dump

# Load database
neo4j-admin load --database=neo4j --from=backup.dump --force

# Export to CSV
MATCH (p:Person) 
RETURN p.name, p.age, p.email
INTO OUTFILE '/tmp/persons.csv'
```

## Monitoring and Maintenance

### Query Monitoring
```cypher
// Show running queries
SHOW TRANSACTIONS

// Kill long-running query
TERMINATE TRANSACTIONS 'transaction-id'

// Query statistics
CALL db.stats.retrieve('GRAPH COUNTS')
```

### Performance Monitoring
```cypher
// Database info
CALL dbms.components()

// Memory usage
CALL dbms.queryJmx('org.neo4j:instance=kernel#0,name=Memory Manager')

// Node and relationship counts
MATCH (n) RETURN count(n) as node_count
MATCH ()-[r]->() RETURN count(r) as relationship_count
```

## Python Integration

### Using neo4j Driver
```python
from neo4j import GraphDatabase

class Neo4jConnection:
    def __init__(self, uri, user, password):
        self.driver = GraphDatabase.driver(uri, auth=(user, password))
    
    def close(self):
        self.driver.close()
    
    def create_person(self, name, age):
        with self.driver.session() as session:
            result = session.run(
                "CREATE (p:Person {name: $name, age: $age}) RETURN p",
                name=name, age=age
            )
            return result.single()
    
    def find_friends(self, name):
        with self.driver.session() as session:
            result = session.run(
                "MATCH (p:Person {name: $name})-[:FRIENDS_WITH]->(friend) "
                "RETURN friend.name as name",
                name=name
            )
            return [record["name"] for record in result]

# Usage
conn = Neo4jConnection("bolt://localhost:7687", "neo4j", "password")
conn.create_person("Alice", 30)
friends = conn.find_friends("Alice")
conn.close()
```

## Use Cases

### Graph Analytics
- **Social networks**: Friend recommendations, influence analysis
- **Fraud detection**: Pattern recognition in financial transactions
- **Network topology**: IT infrastructure mapping
- **Supply chain**: Traceability and optimization

### Knowledge Graphs
- **Semantic search**: Content recommendation systems
- **Data integration**: Connecting disparate data sources
- **AI/ML**: Feature extraction for machine learning

## Related Tools
- [[Cypher]] - Graph query language
- [[APOC]] - Neo4j procedures library
- [[Graph Data Science Library]] - Analytics algorithms
- [[Neo4j Browser]] - Web interface
- [[Bloom]] - Graph visualization

## Related Topics
- [[Graph databases]]
- [[Graph algorithms]]
- [[NoSQL databases]]
- [[Data modeling]]
- [[Graph theory]]
