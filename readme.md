JSPath [![Build Status](https://secure.travis-ci.org/dfilatov/jspath.png)](http://travis-ci.org/dfilatov/jspath)
============

JSPath is a domain-specific language (DSL) that enables you to navigate and find data within your JSON documents. Using JSPath, you can select nodes of JSON in order to retrieve the data they contain.

JSPath for JSON like an XPath for XML.

Installing
----------
###In the Node.js###
You can install using Node Package Manager (npm):

    npm install jspath
    
###In the Browsers###
```html
<script type="text/javascript" src="jspath.min.js"></script>
```
Also RequireJS module format supported.

Usage
-----
```javascript
JSPath.apply(path, json);
// or
JSPath.apply(path, json, substs);
```
where:
<table>
  <tr>
    <th></th>
    <th>type</th>
    <th>description</th>
  <tr>
    <td>path</td>
    <td>String</td>
    <td>path expression</td>
  </tr>
  <tr>
    <td>json</td>
    <td>any valid JSON</td>
    <td>input JSON document</td>
  </tr>
  <tr>
    <td>substs</td>
    <td>Object</td>
    <td>substitutions (optional)</td>
  </tr>
</table>

###Quick example###
```javascript
JSPath.apply(
    '.automobiles{.maker === "Honda" && .year > 2009}.model',
    {
        "automobiles" : [
            { "maker" : "Nissan", "model" : "Teana", "year" : 2011 },
            { "maker" : "Honda", "model" : "Jazz", "year" : 2010 },
            { "maker" : "Honda", "model" : "Civic", "year" : 2007 },
            { "maker" : "Toyota", "model" : "Yaris", "year" : 2008 }            
        ],
        "motorcycles" : [{ "maker" : "Honda", "model" : "ST1300", "year" : 2012 }]
    });
```
Result will be:
```javascript
['Jazz']
```

Documentation
-------------
The result of applying JSPath is always an array.

JSPath consists of two type of top-level expressions: location path (required) and predicates (optional).

###Location path###
To select a nodes in JSPath, you use a location path.
A location path consists of one or more location steps.
Every location step starts with dot (.) or two dots (..) depending on the node you're trying to select:
  * .property &mdash; locates property nodes immediately descended from the context nodes
  * ..property &mdash; locates property nodes deeply descended from the context nodes
  * . &mdash; locates the context nodes itself

Also JSPath allows the wildcard symbol (*) instead of exact name of property.

Your location path can be absolute or relative.
If location path starts with the root (^) you are using an absolute location path &mdash; your location path begins from the root node.

Consider the following JSON:
```javascript
var doc = {
    "books" : [
        {
            "id"     : 1,
            "title"  : "Clean Code",
            "author" : { "name" : "Robert C. Martin" },
            "price"  : 17.96
        },
        {
            "id"     : 2,
            "title"  : "Maintainable JavaScript",
            "author" : { "name" : "Nicholas C. Zakas" },
            "price"  : 10
        },
        {
            "id"     : 3,
            "title"  : "Agile Software Development",
            "author" : { "name" : "Robert C. Martin" },
            "price"  : 20
        },
        {
            "id"     : 4,
            "title"  : "JavaScript: The Good Parts",
            "author" : { "name" : "Douglas Crockford" },
            "price"  : 15.67
        }
    ]
};
```

####Examples####
```javascript
// find all books authors
JSPath.apply('.books.author', doc);
/* [ { name : 'Robert C. Martin' }, { name : 'Nicholas C. Zakas' }, { name : 'Robert C. Martin' }, { name : 'Douglas Crockford' } ] */

// find all books author names
JSPath.apply('.books.author.name', doc);
/* ['Robert C. Martin', 'Nicholas C. Zakas', 'Robert C. Martin', 'Douglas Crockford' ] */

// find all names in books
JSPath.apply('.books..name', doc);
/* ['Robert C. Martin', 'Nicholas C. Zakas', 'Robert C. Martin', 'Douglas Crockford' ] */
```

###Predicates###
An JSPath predicate allows you to write very specific rules about the nodes you'd like to select when constructing your expressions.
Predicates are filters that restrict the nodes selected by an location path. There are two possible type of predicates: object and positional.

###Object predicates###
Object predicates can be used in a path expression to filter a subset of nodes according to a boolean expressions working on a properties of each node.
Object predicates are embedded in braces.

Basic expressions in object predicates:
  * numeric literals (e.g. 1.23)
  * string literals (e.g. "John Gold")
  * boolean literals (true/false)
  * subpathes (e.g. .nestedProp.deeplyNestedProp)

JSPath allows to use in predicate expressions following types of operators:
  * comparison operators
  * string comparison operators
  * logical operators
  * arithmetic operators

####Comparison operators####

<table>
  <tr>
    <td>==</td>
    <td>Returns is true if both operands are equal</td>
    <td>.books{.id == 1}</td>
  </tr>
  <tr>
    <td>===</td>
    <td>Returns true if both operands are strictly equal with no type conversion</td>
    <td>.books{.id === 1}</td>
  </tr>
  <tr>
    <td>!=</td>
    <td>Returns true if the operands are not equal</td>
    <td>.books{.id != 1}</td>
  </tr>
  <tr>
    <td>!==</td>
    <td>Returns true if the operands are not equal and/or not of the same type</td>
    <td>.books{.id !== 1}</td>
  </tr>
  <tr>
    <td>></td>
    <td>Returns true if the left operand is greater than the right operand</td>
    <td>.books{.id > 1}</td>
  </tr>
  <tr>
    <td>>=</td>
    <td>Returns true if the left operand is greater than or equal to the right operand</td>
    <td>.books{.id >= 1}</td>
  </tr>
  <tr>
    <td>&lt;</td>
    <td>Returns true if the left operand is less than the right operand</td>
    <td>.books{.id &lt; 1}</td>
  </tr>
  <tr>
    <td>&lt;=</td>
    <td>Returns true if the left operand is less than or equal to the right operand</td>
    <td>.books{.id &lt;= 1}</td>
  </tr>
</table>

Comparison rules:
  * if both operands to be compared are arrays, then the comparison will be
true if there is a element in the first array and a element in the
second array such that the result of performing the comparison of the two elements is true
  * if one operand is array and another is not, then the comparison will be true if there is element in 
array such that the result of performing the comparison of element and another operand is true
  * primitives to be compared as usual javascript primitives
 
If both operands are strings, also available additional comparison operators:
####String comparison operators####
<table>
  <tr>
    <td>==</td>
    <td>Like an usual '==' but case insensitive</td>
    <td>.books{.title == "clean code"}</td>
  </tr>
  <tr>
    <td>^==</td>
    <td>Returns true if left operand value beginning with right operand value</td>
    <td>.books{.title ^== "Javascript"}</td>
  </tr>
  <tr>
    <td>^=</td>
    <td>Like a '^==' but case insensitive</td>
    <td>.books{.title ^= "javascript"}</td>
  </tr>
  <tr>
    <td>$==</td>
    <td>Returns true if left operand value ending with right operand value</td>
    <td>.books{.title $== "Javascript"}</td>
  </tr>
  <tr>
    <td>$=</td>
    <td>Like a '$==' but case insensitive</td>
    <td>.books{.title $= "javascript"}</td>
  </tr>
  <tr>
    <td>*==</td>
    <td>Returns true if left operand value contains right operand value</td>
    <td>.books{.title *== "Javascript"}</td>
  </tr>
  <tr>
    <td>*=</td>
    <td>Like a '*==' but case insensitive</td>
    <td>.books{.title *= "javascript"}</td>
  </tr>
</table>

####Logical operators####

<table>
  <tr>
    <td>&&</td>
    <td>Returns true if both operands are true</td>
    <td>.books{.price > 19 && .author.name === "Robert C. Martin"}</td>
  </tr>
  <tr>
    <td>||</td>
    <td>Returns true if either operand is true</td>
    <td>.books{.title === "Maintainable JavaScript" || .title === "Clean Code"}</td>
  </tr>
  <tr>
    <td>!</td>
    <td>Returns true if operand is false</td>
    <td>.books{!.title}</td>
  </tr>
</table>

Logical operators convert their operands to boolean values using next rules:
  * if operand is array (as you remember result of applying subpath also array):
    * if length of array greater than zero, result will be true
    * else result will be false
  * Casting with double NOT (!!) javascript operator used in any other cases.

####Arithmetic operators####

<table>
  <tr>
    <td>+</td>
    <td>addition</td>
  </tr>
  <tr>
    <td>-</td>
    <td>subtraction</td>
  </tr>
  <tr>
    <td>*</td>
    <td>multiplication</td>
  </tr>
  <tr>
    <td>/</td>
    <td>division</td>
  </tr>
  <tr>
    <td>%</td>
    <td>modulus</td>
  </tr>  
</table>

####Operator precedence####
<table>
  <tr>
    <td>1 (top)</td>
    <td>! -<sup>unary</sup></td>
  </tr>
  <tr>
    <td>2</td>
    <td>* / %</td>
  </tr>
  <tr>
    <td>3</td>
    <td>+ -<sup>binary</sup></td>
  </tr>
  <tr>
    <td>4</td>
    <td>< <= > >=</td>
  </tr>
  <tr>
    <td>5</td>
    <td>== === != !== ^= ^== $== $= *= *==</td>
  </tr>
  <tr>
    <td>6</td>
    <td>&&</td>
  </tr>
  <tr>
    <td>7</td>
    <td>||</td>
  </tr>
</table>
  
Parentheses are used to explicitly denote precedence by grouping parts of an expression that should be evaluated first.

####Examples####
```javascript
// find all book names whose author is Robert C. Martin
JSPath.apply('.books{.author.name === "Robert C. Martin"}.name', doc);
/* ['Clean Code', 'Agile Software Development'] */

// find all book names with price less than 17
JSPath.apply('.books{.price < 17}.name', doc);
/* ['Maintainable JavaScript', 'JavaScript: The Good Parts'] */
```

###Positional predicates###
Positional predicates allows you to filter nodes by their context position.
Positional predicates are always embedded in square brackets.

There are four available forms:
  * [3] &mdash; returns fourth node in context (first node is at index 0)
  * [2:] &mdash; returns nodes whose index in context is greater or equal to 2
  * [:5] &mdash; returns first five nodes in context
  * [2:5] &mdash; returns three nodes with index 2, 3 and 4
  
Also you can use negative position numbers:
  * [-1] &mdash; returns last node in context
  * [-3:] &mdash; returns last three nodes in context

####Examples####
```javascript
// find first book name
JSPath.apply('.books[0].name', doc);
/* ['Clean Code'] */

// find last book name
JSPath.apply('.books[-1].name', doc);
/* ['JavaScript: The Good Parts'] */

// find two first book names
JSPath.apply('.books[:2].name', doc);
/* ['Clean Code', 'Maintainable JavaScript'] */

// find two last book names
JSPath.apply('.books[-2:].name', doc);
/* ['Agile Software Development', 'JavaScript: The Good Parts'] */

// find two book name from second position
JSPath.apply('.books[1:3].name', doc);
/* ['Maintainable JavaScript', 'Agile Software Development'] */
```

###Multiple predicates###
You can use more than one predicate. The result will contain only the nodes that match all the predicates.

####Examples####
```javascript
// find first book name whose price less than 15 and greater than 5 
JSPath.apply('.books{.price < 15}{.price > 5}[0].name', doc);
/* ['Maintainable JavaScript'] */
```

###Substitutions###
Substitutions allows you to use a runtime-evaluated values in predicates.

####Examples####
```javascript
var path = '.books{.author.name === $author}.name';

// find book name whose author Nicholas C. Zakas
JSPath.apply(path, doc, { author : 'Nicholas C. Zakas' });
/* ['Maintainable JavaScript'] */

// find books name whose authors Robert C. Martin or Douglas Crockford
JSPath.apply(path, doc, { author : ['Robert C. Martin', 'Douglas Crockford'] });
/* ['Clean Code', 'Agile Software Development', 'JavaScript: The Good Parts'] */
```