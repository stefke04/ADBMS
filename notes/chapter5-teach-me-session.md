# Chapter 5: XML and JSON Databases (XML part), complete session notes

This file is self-contained: all code, tables and diagrams from the interactive visuals are written out here, so no other page is needed.

Sources: the lecture transcript (covers XML up to table-based mapping) and the slides. Marks used:
- **(extension)**: goes beyond what the professor said, background from the tutor.
- **(illustrative)**: made-up example values or reconstructed code. The professor taught the concepts, not this exact syntax.
- Slide numbers are the numbers printed on the slides.

---

## Orientation

XML and JSON are formats for **semi-structured data**: there is some structure, but it is more flexible than in a relational table. Their main purpose is **data interchange**. Storage is secondary. The chapter asks:

1. How do you process XML (syntax, rules for structure, parsers and APIs)?
2. How do you store it in a database (whole or in pieces, and how do you bridge the gap with relational tables)?
3. (Later, not in the recorded lecture) How do you query it, and how does JSON do the same?

XML is older and more mature (since 1997, from the W3C, which also made HTML) and has a wide enterprise ecosystem. JSON is newer, smaller, and more common in programming and NoSQL databases.

---

## 1. XML basics

**Question:** how can a program tell that a number in a document is an invoice number and not a shoe size?

- HTML has a fixed tag set that describes **layout** (`<b>` is bold). To a machine an HTML document is flat text.
- XML lets you invent your own tags, which describe **meaning and structure**, not looks. Hence *extensible* markup language. The tags are metadata, like table and column names in a relational database.
- A begin tag and an end tag with the same label (`<name>...</name>`, the end tag has a slash) surround the content.
- Tags must nest properly. Side by side is fine, overlapping is not. There is exactly **one root element**, so every document is a **tree**.
- **Attributes** sit inside the begin tag. Attributes cannot repeat, so a repeated group (two email addresses) becomes sub-elements, told apart by an attribute such as `use`.
- **Comments** exist, as in a programming language. **Processing instructions** are meant for the application that processes the document, for example the line that states the XML version.

Syntax rules, in code:

```xml
<?xml version="1.0"?>                      <!-- processing instruction: for the application -->
<!-- my wine inventory -->                  <!-- comment: for humans -->

<name><first>Jan</first><last>Bas</last></name>     allowed: nested
<first>Jan</first><last>Bas</last>                   allowed: side by side
<name><first>Jan</name></first>                      NOT allowed: overlapping tags

<author name="Bas">                         two email addresses: sub-elements, not repeated attributes
  <email use="work">bas@example.org</email>
  <email use="home">bas@example.net</email>
</author>
```

The running example, a wine cellar (values illustrative), and the same document as a tree:

```xml
<winecellar>
  <wine>
    <name>Chablis</name>
    <year>2019</year>
    <type>white</type>
    <grape percentage="100">Chardonnay</grape>
    <geo>
      <country>France</country>
      <region>Burgundy</region>
    </geo>
    <quantity>6</quantity>
  </wine>
</winecellar>
```

```
winecellar                       (root)
└─ wine
   ├─ name      "Chablis"
   ├─ year      "2019"
   ├─ type      "white"
   ├─ grape     "Chardonnay"     @percentage="100"
   ├─ geo
   │   ├─ country  "France"
   │   └─ region   "Burgundy"
   └─ quantity  "6"
```

In the professor's version, a wine can have several grapes (80% Chardonnay and 20% Pinot Blanc), and a wine can lack a price. That flexibility is what makes the data "semi-structured".

> **Takeaway:** XML tags describe what the data means, they nest strictly, and one root plus the nesting makes every document a tree.

**Self-test**
1. Why can a program interpret XML better than HTML? *The tags carry meaning (invoice number versus shoe size), HTML tags only carry layout.*
2. Which fragment breaks the nesting rule: `<name><first>Jan</first></name>`, `<name><first>Jan</name></first>` or `<first>Jan</first><last>Bas</last>`? *The middle one, tags must close in reverse order of opening.*
3. An author has two email addresses, how do you model that? *Two `email` sub-elements, told apart by an attribute such as `use`, because attributes cannot repeat.*

---

## 2. Well-formed versus valid

**Question:** before an application uses a received document, is the text proper XML at all, and does it have the structure this application expects?

- **Well-formed**: follows the XML syntax rules (proper nesting, one root, matching tags).
- **Valid**: well-formed and also satisfies the rules of a document type (for example "wine cellar"), written in a DTD or an XML Schema.
- Every valid document is well-formed, not the other way round. This mirrors relational databases: data must satisfy the schema (columns, types, foreign keys).
- A parser rejects a document that is not well-formed before validation even starts.

The rule used in the examples below (illustrative): every wine has name, year, type, at least one grape, geo and quantity, and nothing else.

```
Valid  ⊂  Well-formed  ⊂  any text
```

Four documents and their verdicts:

```xml
<!-- 1. Correct: well-formed YES, valid YES -->
<winecellar>
  <wine>
    <name>Chablis</name>
    <year>2019</year>
    <type>white</type>
    <grape>Chardonnay</grape>
    <geo>France, Burgundy</geo>
    <quantity>6</quantity>
  </wine>
</winecellar>

<!-- 2. Overlapping tags: well-formed NO, valid not checked (rejected by the parser) -->
<winecellar>
  <wine>
    <name>Chablis<year>2019</name></year>
    <type>white</type>
    <grape>Chardonnay</grape>
    <geo>France, Burgundy</geo>
    <quantity>6</quantity>
  </wine>
</winecellar>

<!-- 3. Year is missing: well-formed YES, valid NO -->
<winecellar>
  <wine>
    <name>Chablis</name>
    <type>white</type>
    <grape>Chardonnay</grape>
    <geo>France, Burgundy</geo>
    <quantity>6</quantity>
  </wine>
</winecellar>

<!-- 4. Unknown element colour: well-formed YES, valid NO -->
<winecellar>
  <wine>
    <name>Chablis</name>
    <year>2019</year>
    <type>white</type>
    <colour>gold</colour>
    <grape>Chardonnay</grape>
    <geo>France, Burgundy</geo>
    <quantity>6</quantity>
  </wine>
</winecellar>
```

> **Takeaway:** well-formed means correct XML syntax, valid means well-formed plus following a document type's rules.

**Self-test**
1. A document is valid, is it also well-formed? *Yes, always.*
2. Nesting fine, one root, but a required year is missing: verdict? *Well-formed but not valid.*
3. What does a parser do with a document that is not well-formed? *Rejects it, without validating.*

---

## 3. Describing document types: DTD and XML Schema (XSD)

**Question:** where are the document type rules written down? Two languages: the older **DTD** (document type definition) and the newer, stronger **XML Schema**, abbreviated **XSD** (XML Schema Definition).

- **DTD**: own syntax, **not an XML document**. Content is only character strings: `#PCDATA` (parsed character data). An asterisk means a repetition (grape can repeat). It also has a list of attributes.
- **XML Schema**: **is itself a well-formed XML document**, like a relational database storing its metadata in relational tables. It supports data types (string, short integer, boolean, date) and aggregated types, sequences, occurrence counts (at least zero, unbounded), complex elements (with sub-elements) and optional attributes (the currency of a price).
- Depth in the lecture: both were explained at concept level ("you can understand what is imposed"), not syntax. The code below is **illustrative**: shortened and reconstructed, not the exact slide text.

DTD (illustrative):

```dtd
<!ELEMENT winecellar (wine*)>
<!ELEMENT wine (name, year, type, grape*, geo, quantity)>
<!ELEMENT name  (#PCDATA)>
<!ELEMENT year  (#PCDATA)>
<!ELEMENT type  (#PCDATA)>
<!ELEMENT grape (#PCDATA)>
<!ELEMENT geo   (country, region)>
<!ELEMENT country (#PCDATA)>
<!ELEMENT region  (#PCDATA)>
<!ELEMENT quantity (#PCDATA)>
<!ATTLIST grape percentage CDATA #IMPLIED>
```

XML Schema (illustrative, shortened):

```xml
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema">
  <xs:element name="winecellar">
    <xs:complexType>
      <xs:sequence>
        <xs:element name="wine" minOccurs="0" maxOccurs="unbounded">
          <xs:complexType>
            <xs:sequence>
              <xs:element name="name" type="xs:string"/>
              <xs:element name="year" type="xs:short"/>
              <xs:element name="type" type="xs:string"/>
              <xs:element name="grape" minOccurs="1" maxOccurs="unbounded">
                <xs:complexType>
                  <xs:simpleContent>
                    <xs:extension base="xs:string">
                      <xs:attribute name="percentage" type="xs:short"/>
                    </xs:extension>
                  </xs:simpleContent>
                </xs:complexType>
              </xs:element>
              <xs:element name="geo">
                <xs:complexType>
                  <xs:sequence>
                    <xs:element name="country" type="xs:string"/>
                    <xs:element name="region"  type="xs:string"/>
                  </xs:sequence>
                </xs:complexType>
              </xs:element>
              <xs:element name="quantity" type="xs:short"/>
            </xs:sequence>
          </xs:complexType>
        </xs:element>
      </xs:sequence>
    </xs:complexType>
  </xs:element>
</xs:schema>
```

Note from the lecture: the DTD has `grape*` (zero or more), the schema says at least one grape. Schemas can have more, such as a price with an optional `currency` attribute.

| | DTD | XML Schema |
|---|---|---|
| Is itself XML | no, separate syntax | yes, a well-formed XML document |
| Data types | only character strings (`#PCDATA`) | string, short integer, boolean, date, aggregated types |

What a data type buys you, on the content of the `year` element:

| `<year>...</year>` | Valid against the DTD? | Valid against the schema (short integer)? |
|---|---|---|
| `2019` | yes | yes |
| `abc` | yes (it is a string) | no |
| `19.5` | yes | no |

> **Takeaway:** a DTD only knows character strings and has its own syntax, while an XML Schema adds real data types and is itself XML.

**Self-test**
1. Which is true of a DTD but not of an XML Schema? *It is not itself an XML document.*
2. DTD says `year (#PCDATA)`: is `<year>abc</year>` valid? *Yes, any string is accepted.*
3. Why is it handy that a schema is XML? *The same XML tools handle data and rules, like relational metadata stored in tables.*

---

## 4. Style sheets (XSLT)

**Question:** how do you give an XML document another structure, or a layout, without touching the source and without restructuring a database?

- A **style sheet** is a separate XML document with transformation rules. The XSL style sheet family has several components, only **XSLT** (XSL Transformations) matters for the course. The other one "has gotten out of use". It is probably XSL-FO (formatting objects, page layout for print such as PDF) **(extension: name and role)**.
- Main outputs: another **XML document** with a different structure, or an **HTML document** (for display). Other outputs exist, such as plain text **(extension)**.
- Unlike HTML, an XML document holds no layout. The layout lives in a style sheet, so one document can get different layouts (smartphone, browser, print), or be reshaped for an application that expects another format.
- Who writes the style sheet when formats do not match? Typically the developer of the application or data exchange. The lecture does not say. It is the cheaper route compared to changing source or application.
- The style sheet is itself XML. Its tags (`template`, `for-each`) are in the XSL namespace (see point 5), which keeps them apart from your own tags.
- The `select="..."` expressions inside are XPath (point 6) **(extension)**.

Source document (illustrative):

```xml
<winecellar>
  <wine><name>Barolo</name><year>2016</year>
        <quantity>3</quantity></wine>
  <wine><name>Chablis</name><year>2019</year>
        <quantity>6</quantity></wine>
  <wine><name>Rioja</name><year>2018</year>
        <quantity>12</quantity></wine>
</winecellar>
```

Style sheet 1, XML to XML (simplified template, illustrative):

```xml
<xsl:template match="/winecellar">
  <winecellarsummary>
    <xsl:for-each select="wine">
      <wine>
        <name><xsl:value-of select="name"/></name>
        <quantity><xsl:value-of select="quantity"/></quantity>
      </wine>
    </xsl:for-each>
  </winecellarsummary>
</xsl:template>
```

Output 1:

```xml
<winecellarsummary>
  <wine><name>Barolo</name><quantity>3</quantity></wine>
  <wine><name>Chablis</name><quantity>6</quantity></wine>
  <wine><name>Rioja</name><quantity>12</quantity></wine>
</winecellarsummary>
```

Style sheet 2, XML to HTML (simplified template, illustrative):

```xml
<xsl:template match="/winecellar">
  <h1>My wine cellar</h1>
  <table>
    <tr><th>Name</th><th>Year</th><th>Quantity</th></tr>
    <xsl:for-each select="wine">
      <tr>
        <td><xsl:value-of select="name"/></td>
        <td><xsl:value-of select="year"/></td>
        <td><xsl:value-of select="quantity"/></td>
      </tr>
    </xsl:for-each>
  </table>
</xsl:template>
```

Output 2 (HTML, shown in a browser as a heading and a table; this HTML carries layout metadata and no structural metadata):

```html
<h1>My wine cellar</h1>
<table>
  <tr><th>Name</th><th>Year</th><th>Quantity</th></tr>
  <tr><td>Barolo</td><td>2016</td><td>3</td></tr>
  <tr><td>Chablis</td><td>2019</td><td>6</td></tr>
  <tr><td>Rioja</td><td>2018</td><td>12</td></tr>
</table>
```

> **Takeaway:** an XSLT style sheet is an XML document with transformation rules, so one source can become a different XML structure or an HTML page without changing the source.

**Self-test**
1. What can XSLT turn an XML document into? *Another XML document with a different structure, or an HTML document (main outputs).*
2. Where do layout rules for an XML document live? *In a separate style sheet, not in the document.*
3. Why is applying a style sheet attractive? *One source can serve a phone, a browser and print, or an application expecting another format.*

---

## 5. Namespaces

**Question:** anyone can invent tags, so the same tag name can mean different things (`element` in XML Schema, in a set theory document, in a chemistry document). How does a program tell them apart?

- A **namespace prefix** points to a unique identifier, a **URI** (uniform resource identifier), in practice usually a URL. The URL does **not** have to lead to a web page, it only needs to be unique. For XSLT and XML Schema the address does show the specification, but that is not required.
- `xmlns:prefix="URI"` declares a prefix. `xmlns="URI"` declares the **default namespace**: unprefixed tags belong to it, only tags from another namespace need a prefix.
- Namespaces are mainly to avoid clashes and to tell which vocabulary a tag comes from. The default namespace is only a shortcut.
- Slide example: a first line with a prefix declaration and prefixed tags, then, as an alternative, a second `winecellar` opening tag with a default namespace. That bottom line is an opening tag of the alternative notation, not a closing tag.

Three vocabularies, same tag name:

| Document | Tag | Means | Namespace |
|---|---|---|---|
| XML Schema | `xs:element` | declares an element of a document type | `http://www.w3.org/2001/XMLSchema` |
| Set theory | `set:element` | a member of a set | `http://example.org/sets` (illustrative) |
| Chemistry | `chem:element` | a chemical element | `http://example.org/chemistry` (illustrative) |

Without namespaces all three are just `<element>`, indistinguishable.

Prefix form and default form (the URL is made up):

```xml
<w:winecellar xmlns:w="http://example.org/winecellar">
  <w:wine>
    <w:name>Chablis</w:name>
  </w:wine>
</w:winecellar>

<winecellar xmlns="http://example.org/winecellar">
  <wine>
    <name>Chablis</name>
  </wine>
</winecellar>
```

> **Takeaway:** a namespace prefix points to a unique identifier, usually a URL that needn't exist, so the same tag name from different vocabularies stays distinguishable.

**Self-test**
1. What does a namespace prefix stand for? *A unique identifier, in practice usually a URL.*
2. Must the URL lead to a web page? *No, it only has to be unique.*
3. Root declares a default namespace, what does an unprefixed child belong to? *The default namespace.*

---

## 6. XPath

**Question:** how does a program point at one particular node of the tree, like referring to a cell in a relational table?

- XPath uses **path expressions** that walk down the tree, starting at the root. The lecture stressed that it is **not a real query language**, just a way to refer to a node. Slide 24 calls it "a simple, declarative language that uses path expressions": you describe which nodes you want and the engine finds them, but it lacks the rest of a query language (joins, grouping, building new results). Proper XML querying (XQuery) comes later in the chapter.
- A number in brackets picks the n-th child, a condition in brackets filters.

Slide 24 examples:

```
doc("winecellar.xml")/winecellar/wine
doc("winecellar.xml")/winecellar/wine[2]
doc("winecellar.xml")/winecellar/wine[price > 20]/name
```

Worked example on this data (illustrative; the tree has three wines, Barolo price 18, Chablis price 25, Rioja price 32):

```
winecellar
├─ wine [1]   name Barolo   year 2016  price 18
├─ wine [2]   name Chablis  year 2019  price 25
└─ wine [3]   name Rioja    year 2018  price 32
```

| Path | Points at |
|---|---|
| `/winecellar/wine[1]` | the first wine element (Barolo) with everything inside it |
| `/winecellar/wine[2]` | the second wine element (Chablis) with everything inside it |
| `/winecellar/wine[2]/name` | the name of the second wine, Chablis |
| `/winecellar/wine[price>20][1]/name` | the name of the first wine with price above 20: Barolo (18) is skipped, answer Chablis |
| `/winecellar/wine[price>20]/name` | **(extension)** in standard XPath, without `[1]`, the names of all wines above 20: Chablis and Rioja |

> **Takeaway:** XPath is not a query language, it is a way to address one node by walking down the document tree, optionally with a number or a condition.

**Self-test**
1. How did the lecture describe XPath? *A way to refer to a particular node, not a real query language.*
2. Why can XPath point at a node by walking from the root? *Every XML document has a tree structure.*
3. What does `/winecellar/wine[2]` point at? *The second wine element, with its sub-elements.*

---

## 7. The processing pipeline (slide 25)

**Question:** what happens to a received XML document before its data is usable? A browser can show any HTML page because HTML tags are fixed. XML tags are invented per document kind, so there is no single application for all XML. What is shared is the pipeline in front of the application.

Slide 25 diagram:

```
 XML document
      │
      ▼
 XSLT processor  ◄──── XSL stylesheet
      │
      ▼
  XML parser     ◄──── DTD or XSD
      ▲
      ║   DOM or SAX
      ▼
  Application
```

- **XSLT processor**: optional, only if the application expects another format than the document has.
- **XML parser**: generic component. Checks well-formed, and valid if the document refers to a DTD or XSD. Rejected documents never reach the application.
- **API** (DOM or SAX): how the parsed document is presented to the application.
- **Application**: your own logic, specific to the document type (wine cellar, purchase order). The parser is conceptually a separate stage, in practice usually a library your program calls **(extension)**.

Three scenarios:

| Document | Where it ends |
|---|---|
| Good document | XSLT (if needed), well-formed pass, valid pass, API, application |
| Not well-formed | rejected at the parser, application never sees it |
| Well-formed, not valid | rejected at the parser during validation |

> **Takeaway:** every XML document runs through a pipeline of an optional XSLT processor, a parser that checks well-formedness and validity, and an API, so the application only sees safe input.

**Self-test**
1. Why is there no single application for all XML? *Tags are self-defined, so each document type needs tailored application logic. The generic part is the parser.*
2. Where is a document that is not well-formed stopped? *At the parser, before the application.*
3. When is the XSLT processor in the pipeline? *When the application expects another format than the document has.*

---

## 8. DOM, SAX and StAX (and their use cases)

**Question:** the parser has read the document, in what form does the application get at the content? Two extremes: the whole document as a tree of objects, or a stream of events as the parser reads.

The fragment used (slide 27 and 28):

```xml
<wine>
  <name>Meneghetti White</name>
  <year>2010</year>
</wine>
```

**DOM** (tree-based API, developed by W3C; Document Object Model is my expansion **(extension)**). The document becomes interrelated objects, with methods to navigate and change them:

```
[wine]
   [name]
      text: Meneghetti White
   [year]
      text: 2010
```

**SAX** (Simple API for XML, event-based). Events raised as the parser meets each part (slide 28):

```
start document
start element: wine
start element: name
text: Meneghetti
end element: name
start element: year
text: 2010
end element: year
end element: wine
end document
```

**StAX** (Streaming API for XML): a compromise, the application **pulls** data from the stream with a **cursor** (like scrolling through the rows of a SQL query result, one at a time). More control than SAX, still a stream.

| | DOM | SAX | StAX |
|---|---|---|---|
| Form | tree of objects | stream of events pushed to a handler | stream, pulled with a cursor |
| Memory | whole document in memory, can be memory intensive | small footprint, scalable | small footprint |
| Access | direct access to any part | sequential only, cannot return | sequential, application controls pace |
| Many manipulations | well suited (prune, graft, move, delete sub-trees, add elements) | less suited | n/a |

Use cases (saved with `!`):
- **DOM**: direct access to specific parts and many manipulations, on a document that fits in memory.
- **SAX**: a big XML dump from a database that does not fit in memory, or a real-time stream such as sensors sending XML. A handler reacts only to relevant events, for example every wine from before 2010.
- **StAX**: when you want more control than SAX gives, while still streaming.

> **Takeaway:** DOM hands over the whole document as a tree of objects, which is flexible but memory hungry, while SAX streams events, which is light but sequential, and StAX lets the application pull with a cursor.

**Self-test**
1. Endless stream of sensor XML, react to certain readings: which API? *SAX, with event handlers.*
2. Direct access and many manipulations on a modest document? *DOM.*
3. What is StAX? *A compromise where the application pulls data from the stream with a cursor.*

---

## 9. Storing XML in a database: whole or in pieces

**Question:** when you store an XML document, do you care about the document itself or only about the data in it? Three approaches: **document-oriented**, **data-oriented** and **combined** (point 10).

- **Document-oriented**: the document is stored as a whole, a round trip gives the same document. In a relational database it is a **CLOB** (character large object, a user-defined type beyond the basic types), or a dedicated XML data type from SQL/XML. For plain SQL the cell is the smallest value. Looking inside needs user-defined functions (searching for a term, a full-text style search) or an XML type with functions such as XPath. Typical when the document is important and mostly static. Dedicated document stores (next chapter) also store documents whole.
- **Data-oriented**: **shredding**: the document is cut into pieces that go to individual cells (rows, columns, even different tables) via a mapping. SQL can query single values. A round trip gives the same data, not necessarily the same document. Typical when the data matters and you want fine-grained retrieval.

Staff table with a resume column (lecture idea; values illustrative):

Document-oriented, one CLOB cell per resume:

| id | name | resume (CLOB) |
|---|---|---|
| 1 | An Peeters | `<resume><degree>MSc</degree><employer>Acme</employer></resume>` |
| 2 | Jan Smets | `<resume><degree>BSc</degree><employer>Globex</employer></resume>` |

Data-oriented, shredded:

| id | name | degree | employer |
|---|---|---|---|
| 1 | An Peeters | MSc | Acme |
| 2 | Jan Smets | BSc | Globex |

| | Document-oriented | Data-oriented |
|---|---|---|
| Get the resume back unchanged | yes | not guaranteed (same data, maybe other structure) |
| Read the degree with plain SQL | no (only whole cell) | yes |
| Search a term inside | with user-defined functions or XML type | with SQL on cells |

> **Takeaway:** store the document as a whole (CLOB or XML type) when the document itself matters, and shred it into cells when the data matters and you want fine-grained SQL access.

**Self-test**
1. Which approach guarantees the same document back? *Document-oriented.*
2. What can plain SQL do with a resume in a CLOB? *Only retrieve the whole cell.*
3. Data matters and you want fine-grained SQL retrieval: which approach? *Data-oriented (shredding).*

---

## 10. The combined approach and shredding depth

**Question:** must every part of a document get the same treatment? No: shred what you query, keep free-form parts as XML fragments.

Lecture examples: an **insurance claim** (date, claim number, contract number, address shredded, the prose explanation with a few tags stored as an XML fragment) and **employees** (name and number shredded, the resume as XML).

Reason for limiting depth: if you shred a whole document and later want it back as a whole, you need many joins between tables, which is expensive. So shred up to a certain level only. The DBMS itself may pick a good shredding level from past usage, or store **both** a shredded and a whole version and use whichever suits the query. Storing a full copy next to the shredded cells avoids joins but costs extra storage and the effort to keep both in sync **(extension: trade-offs)**.

Claim document (illustrative):

```xml
<claim>
  <date>2025-03-14</date>
  <claimNumber>C-1042</claimNumber>
  <contractNumber>P-778</contractNumber>
  <address><street>Main St 5</street>
           <city>Leuven</city></address>
  <explanation>
    <p>A tree fell on my car...</p>
    <p>The other driver saw it.</p>
  </explanation>
</claim>
```

Three storage depths:

Whole document, one table:

| id | document |
|---|---|
| 1 | `<claim>...</claim>` |

Combined, one table (SQL queries date, numbers, address, explanation comes back as XML fragment, rebuild without a join):

| id | date | claim_nr | contract_nr | street | city | explanation |
|---|---|---|---|---|---|---|
| 1 | 2025-03-14 | C-1042 | P-778 | Main St 5 | Leuven | `<explanation>...</explanation>` |

Fully shredded, two tables (everything queryable, rebuilding needs a join):

| claim: id | date | claim_nr | contract_nr | street | city |
|---|---|---|---|---|---|
| 1 | 2025-03-14 | C-1042 | P-778 | Main St 5 | Leuven |

| paragraph: claim_id | nr | text |
|---|---|---|
| 1 | 1 | A tree fell on my car... |
| 1 | 2 | The other driver saw it. |

> **Takeaway:** shred the parts you query with SQL and keep the free-form parts as XML, because shredding everything makes rebuilding the whole document expensive.

**Self-test**
1. Claim with a date, a claim number and a free prose explanation: best approach? *Combined: shred the structured parts, keep the explanation as XML.*
2. Why not always shred completely? *Rebuilding the whole document then needs many joins, which is expensive.*
3. What may the DBMS do itself? *Choose a shredding level from past usage, or store both a shredded and a whole version.*

---

## 11. Why mapping XML to tables is hard (1/2): shape

**Question:** both models are structured, so a mapping exists, but their building blocks differ (like the object versus table gap in an object-relational mapping, only larger).

| XML | Relational problem | Relational fix | Object-relational fix |
|---|---|---|---|
| Ordered elements | tuples have no logical order | extra position column | list collection type (an ordered set) |
| Nesting (hierarchy) | first normal form forbids nested relations (composite attributes) | separate table with a foreign key: a relationship, not necessarily part-of | nested / composite types |
| Repeated children | first normal form forbids multivalued attributes | additional table | collection types, so shredding is less deep |

In tables (illustrative):

Order:

```xml
<winecellar>
  <wine>Chablis</wine>      the first wine and the second wine cannot be switched
  <wine>Barolo</wine>
</winecellar>
```

| wine: name | position |
|---|---|
| Chablis | 1 |
| Barolo | 2 |

Nesting:

```xml
<wine>
  <name>Chablis</name>
  <geo><country>France</country><region>Burgundy</region></geo>
</wine>
```

| wine: id | name |
|---|---|
| 1 | Chablis |

| geo: wine_id (FK) | country | region |
|---|---|---|
| 1 | France | Burgundy |

Repeated children:

```xml
<wine>
  <name>Chablis</name>
  <grape>Chardonnay</grape>
  <grape>Pinot Blanc</grape>
</wine>
```

| wine: id | name |
|---|---|
| 1 | Chablis |

| grape: wine_id (FK) | grape |
|---|---|
| 1 | Chardonnay |
| 1 | Pinot Blanc |

If the order of the grapes matters too, that table needs its own position column as well (the lecture's ordering example was the wines in the cellar).

How a list helps **(extension: example)**: an object-relational column of type "list of wines" holds Chablis, Barolo in that order as part of the value itself. A list with Chablis, Barolo differs from a list with Barolo, Chablis, so no position column is needed.

Composite versus collection **(extension: contrast)**: a composite attribute is one value made of named parts of different kinds (country, region). A collection is several values of the same kind. Both put more than one atomic value in a cell, which first normal form forbids in a plain relational database.

> **Takeaway:** XML is ordered, nested and allows repeated children, which tables forbid under first normal form, so a mapping needs position columns and extra tables, unless the object-relational features model them directly.

**Self-test**
1. Why does a relational mapping of a wine cellar need a position column? *XML elements are ordered, tuples of a relation are not.*
2. Why can a wine with two grapes not hold both in one grape column? *First normal form forbids multivalued attributes, use an extra table.*
3. How does an object-relational database help? *Collection and nested types model order, nesting and repeats directly, so shredding is less deep.*

---

## 12. Why mapping XML to tables is hard (2/2): types and rigidity

**Question:** does the model know that a year is a number and not just characters, and what happens when a new kind of data appears?

| Model | Which types exist | Note |
|---|---|---|
| Relational (plain) | atomic built-in types only: integer, string, date | no composite types |
| XML without a schema | none, everything is characters | |
| DTD | only parsed character data, #PCDATA | no difference between a number and a string |
| XML Schema | atomic and aggregated types | the tree is an aggregation of different elements |

Aggregated types: modeled with **user-defined types** in an object-relational database, with multiple tables in a plain relational database.

Rigidity (slide 36: "XML data is semi-structured, can include certain anomalies, a change to the DTD or XSD necessitates re-generation of tables"). Cases (illustrative):

| Case | XML (semi-structured) | Relational (rigid) |
|---|---|---|
| A wine has no price | leave the price element out (if the schema marks it optional) | the table has a price column for every row, the structure is fixed |
| A wine has two prices | add a second price element | one column cannot hold two values, extra table (point 11) |
| New field colour | add a colour element, adjust the schema if you validate | change the table structure, regenerate and populate again: much more expensive |

Rigidity is not simply bad: it means stronger enforcement of constraints and so good data quality, at the cost of flexibility. What is best depends on the use.

> **Takeaway:** relational databases have only atomic types and a fixed structure, while XML Schema has richer types and XML documents may vary, which gives flexibility but weaker enforcement of data quality.

**Self-test**
1. Which XML specification can say year is a short integer? *XML Schema (a DTD only has #PCDATA).*
2. Why is a new field more expensive in a relational database? *The table structure has to be regenerated and the table populated again.*
3. Is rigidity just a disadvantage? *No, it enforces constraints more strongly and guarantees data quality, at the cost of flexibility.*

---

## 13. Mapping techniques and table-based mapping

**Question:** given all those differences, how do you actually map? The lecture sorts the techniques by where the mapping starts (slide 37):

| Technique | Starts from |
|---|---|
| Table-based mapping | the relational structure, gives a deterministic XML format |
| Schema-oblivious mapping | the XML structure, imposes a rigid relational structure |
| Schema-aware mapping | the conceptual (domain) model behind the schema, so mapping at the conceptual level |
| SQL/XML | an extension of SQL to cope with XML in a relational database, several purposes |

The lecture only covered **table-based** mapping, the rest continues next week.

Table-based structure (slide 38): tables, rows, columns and cells are preserved:

```xml
<database>
  <table>
    <row>
      <column1> data </column1>
      …
    </row>
    <row>
      <column1> data </column1>
      …
    </row>
    …
  </table>
  <table>
    …
  </table>
  …
</database>
```

Example on a wine table (values illustrative). Table:

| name | year |
|---|---|
| Chablis | 2019 |
| Barolo | 2016 |

As XML with generic tags, and with renamed column tags (the names may change, the structure still reflects tables, rows and cells):

```xml
<database>                             <database>
  <table>                                <table>
    <row>                                  <row>
      <column1>Chablis</column1>             <name>Chablis</name>
      <column2>2019</column2>                <year>2019</year>
    </row>                                 </row>
    <row>                                  <row>
      <column1>Barolo</column1>              <name>Barolo</name>
      <column2>2016</column2>                <year>2016</year>
    </row>                                 </row>
  </table>                               </table>
</database>                            </database>
```

- Advantage: simplicity, a perfect one-to-one mapping, very deterministic. The actual data is stored as the content of column elements.
- Disadvantage: rigid XML structure. If an application expects something else, transform it afterwards, for example with XSLT (slide 39).
- Convenient to **dump** a relational database or to **migrate** data from one relational database to another: it stays in the format of columns, tables and rows.
- Slide 39 adds that the document structure can be implemented with an updatable SQL view (not mentioned by the professor).

> **Takeaway:** table-based mapping starts from the relational structure and writes it out as database, table, row and column elements, which is simple and one-to-one but rigid.

**Self-test**
1. Where does table-based mapping start? *From the relational structure, which fixes the XML format.*
2. Advantage and disadvantage? *Simple one-to-one mapping, but a rigid XML structure.*
3. A typical use? *Dumping a relational database or migrating data to another relational database.*

---

## Questions asked during the session, and the short answers

- **Did the professor go deep on DTD and schema syntax?** No. DTD: about three sentences (multiple wine elements, the asterisk for repetition, #PCDATA is a string, a list of attributes, not XML itself). Schema: walked through in words (complex type, sequence, zero to unbounded wine elements, name string, year short, type string, grape at least once, price with optional currency attribute, geo with country and region). He called the attribute details "nitty-gritty" to look at later. The exact keywords in my code are reconstructions.
- **Only these two XSLT outputs?** The lecture says multiple outputs exist, XML and HTML are the most important. Plain text is my extension.
- **XSLT versus XSL-FO, and is XSLT an application of XSL?** XSL is the style sheet family, XSLT is one component, the other component is out of use (probably XSL-FO, page layout for print). XSLT is not an "application" of XSL, it is one of its parts.
- **Did he talk about the HTML code?** Briefly: the style sheet starts from the wine cellar, wraps in HTML tags (header, table, header row), a for-each over wine elements selects name, year and quantity per row, output is an HTML document viewed as a table in a browser. He did not explain HTML syntax.
- **Are other namespaces for duplicate tags or a cleaner structure?** Mainly to avoid clashes and tell which vocabulary a tag comes from, the default namespace is the tidy shortcut.
- **Is the bottom line of the namespace slide a closing tag?** No, it is the opening tag of the default-namespace notation.
- **Is XPath not declarative like SQL?** Slide 24 says declarative. The professor's remark was that it is not a real query language. Both hold: you describe the nodes, but it lacks joins, grouping and result construction.
- **Is the parser part of the application?** No, it is a generic component in front of it. In practice usually a library the application calls **(extension)**.
- **Is the XML always tailored to the application, maybe via a style sheet?** Not always. If document and application match it goes straight through. A style sheet reshapes it when they do not. Someone has to write the style sheet.
- **Abbreviations:** SAX = Simple API for XML, StAX = Streaming API for XML, DOM = Document Object Model (my expansion), API = application programming interface, XSD = XML Schema Definition, DTD = document type definition.
- **What is a cursor mechanism?** A pointer to the current position in a sequence of results that you move forward one item at a time. For a SQL query with a big result you scroll row by row, StAX does the same on the event stream.
- **Can you not query attributes inside a document-oriented resume?** Not with plain SQL on a CLOB. User-defined functions (term search) or an XML data type with XPath-like functions can, and if you query one value often, shred it.
- **Is this about storage in a relational database?** The examples here, yes (also object-relational because of CLOB and collection types). Document stores (next chapter) store whole documents too.
- **Could you shred the queried parts and also keep the full document in a separate cell, so no joins are needed?** Yes, the lecture says the system may store both versions and use whichever suits the query. Trade-offs (extra storage, keeping them in sync) are my extension.
- **If the order matters, should the grape table also have a position column?** Yes, for any repeated element whose order matters. I left it out to show one difference at a time.
- **Is a list a composite attribute?** Close, but not the same. Composite is named parts of different kinds (nesting), a collection is several values of the same kind (multivalued, ordered with a list).

## Not covered by the recorded lecture

Slides 40 to 79: schema-oblivious mapping (a generic table with columns ID, PARENT_ID, TYPE, LABEL, VALUE), schema-aware mapping, SQL/XML, searching XML (full-text, keyword-based, structured search with XQuery) and the whole JSON part (key-value pairs, objects, arrays, JSON Schema, JSONPath, SQL/JSON). Also the sample exam questions on physical database organization discussed after the break (earlier chapters).

## Glossary

- **CLOB**: character large object, a text stored in a single cell.
- **DTD**: document type definition. **XSD**: XML Schema Definition.
- **XSLT**: XSL Transformations. **XPath**: path expressions to refer to nodes of the tree.
- **URI**: uniform resource identifier, in practice usually a URL, used as a unique namespace identifier.
- **PCDATA**: parsed character data, plain strings.
- **DOM**: tree-based API. **SAX**: event-based API. **StAX**: pull-based streaming API with a cursor.
- **Shredding**: cutting an XML document into pieces stored in individual cells.
- **Well-formed**: follows the XML syntax. **Valid**: well-formed and follows its document type.
- **Semi-structured**: some structure, but more flexible than a relational schema.

## Tutoring corrections made during the session

- Point 1 first skipped comments and processing instructions, added later in point 4.
- XPath: the slide says declarative, the professor said not a real query language. Both are noted in point 6.
- The quiz wording on XSLT outputs was loosened: the two outputs are the main ones, not the only ones.
