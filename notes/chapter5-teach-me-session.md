# Chapter 5: XML and JSON Databases (XML part), session notes

Source: lecture transcript (covers XML up to table-based mapping) plus the slides. Items marked **(extension)** go beyond the lecture. Sample values in examples are illustrative.

Artifact pages (interactive visuals and quizzes, private links):
1 https://claude.ai/artifact/WnwoQxf1JjufvYMgw1rqmj
2 https://claude.ai/artifact/XE1QnFSosdsCrPBjj1X7ur
3 https://claude.ai/artifact/HE3uv62qX3UBbJfkFG6V4V
4 https://claude.ai/artifact/FpC3fezifF7htWQt4g47E6
5 https://claude.ai/artifact/QAccMpr3j8EEt8Cu9XctkA
6 https://claude.ai/artifact/ByukRuCEDa3dVpS4YAm3wd
7 https://claude.ai/artifact/Hf7eMgGoR77fpLAMv3F4h4
8 https://claude.ai/artifact/MFms8S723LYStMZ5B1Wy3S
9 https://claude.ai/artifact/CSxXUgHqmSGKrxGhGPNQSk
10 https://claude.ai/artifact/8Sbp8CYVranZoQhdZfnYgy
11 https://claude.ai/artifact/8ErmRDbUb1xVYyqKw2vhKA
12 https://claude.ai/artifact/CS7QnZpoCMS62YeTBouVJu
13 https://claude.ai/artifact/H19aqCZ5ry8zsxAQrXzFcz

## Orientation

XML and JSON are formats for **semi-structured data**: some structure, but more flexible than a relational table. Their main purpose is data interchange, storage is secondary. The chapter asks how to process XML (syntax, rules, parsers), how to store it (whole or in pieces, and how to bridge the gap with relational tables), and later how to query it and how JSON does the same. XML is older and more mature (since 1997, W3C), JSON is newer, smaller and more common in programming and NoSQL.

## 1. XML basics

- HTML has a fixed tag set that describes **layout** (`<b>` bold). XML lets you invent tags that describe **meaning and structure**, so a machine can tell an invoice number from a shoe size. Hence "extensible markup language".
- Begin tag and end tag with the same label (`<name>...</name>`). Tags must nest properly, side by side is fine, overlapping is not. Exactly one root element, so every document is a **tree**.
- Attributes sit in the begin tag. Attributes cannot repeat, so repeated groups (two email addresses) become sub-elements, told apart by an attribute such as `use`.
- Comments exist, as in programming languages. **Processing instructions** are for the application that processes the document, for example the XML version line `<?xml version="1.0"?>`.
- Tags are metadata, like table and column names in a relational database.

Example (values illustrative) and its tree:

```
<winecellar>                          winecellar            (root)
  <wine>                                 └─ wine
    <name>Chablis</name>                      ├─ name      "Chablis"
    <year>2019</year>                         ├─ year      "2019"
    <type>white</type>                        ├─ type      "white"
    <grape percentage="100">                  ├─ grape     "Chardonnay"   @percentage="100"
      Chardonnay</grape>                      ├─ geo
    <geo>                                     │    ├─ country  "France"
      <country>France</country>               │    └─ region   "Burgundy"
      <region>Burgundy</region>               └─ quantity  "6"
    </geo>
    <quantity>6</quantity>
  </wine>
</winecellar>
```

## 2. Well-formed versus valid

- **Well-formed**: follows the XML syntax rules (nesting, one root, matching tags).
- **Valid**: well-formed and also satisfies the rules of a document type (for example wine cellar), defined in a DTD or an XML Schema.
- Every valid document is well-formed, not the other way round. A parser rejects a non-well-formed document before validation even starts.

| Document | Well-formed? | Valid? |
|---|---|---|
| Correct wine cellar | yes | yes |
| Overlapping tags | no | not checked |
| Required year missing | yes | no |
| Unknown element (colour) | yes | no |

## 3. Describing document types: DTD and XML Schema (XSD)

- **DTD** (document type definition): oldest. Own syntax, **not XML itself**. Only character strings (`#PCDATA`). Asterisk means repetition (grape can repeat). It also has a list of attributes. The lecture covered it in a few sentences.
- **XML Schema** (**XSD**, XML Schema Definition): newer and stronger. **Is itself a well-formed XML document**, like a relational database storing its metadata in relational tables. Supports data types (string, short integer, boolean, date) and aggregated types, element sequences, occurrence counts (at least zero, unbounded), optional attributes (currency of price), complex elements with sub-elements.
- The lecture explained both at concept level, not syntax. The exact keywords in my excerpts were reconstructions **(extension)**.

| | DTD | XML Schema |
|---|---|---|
| Is itself XML | no | yes |
| Data types | only strings (#PCDATA) | string, number, boolean, date, aggregated types |

Demo: `<year>abc</year>` is valid against the DTD (it is a string) but not against the schema (year is a short integer).

## 4. Style sheets (XSLT)

- Question: how to give an XML document another structure or a layout without touching the source? A style sheet is a separate XML document with transformation rules. XSLT (XSL Transformations) is the relevant part of XSL. The other component is out of use (probably XSL-FO, formatting objects, for print layout) **(extension: the name and role)**.
- Outputs the lecture names as most important: another **XML document** with a different structure, or an **HTML document**. Other outputs exist, such as plain text **(extension)**.
- Example (illustrative): for each `wine`, select `name` and `quantity` and wrap them in new tags gives a summary document. The HTML version outputs a header "my wine cellar" and a table with a row per wine (name, year, quantity). The output then carries layout metadata and no structural metadata.
- Advantage: write the document once, apply different style sheets for smartphone, browser and print, or reshape it for an application that expects another format. Much easier than restructuring a relational database.
- The style sheet is itself XML, its tags (`template`, `for-each`) are in the XSL namespace. Someone (usually the developer) has to write the style sheet when formats do not match.

## 5. Namespaces

- Anyone can create tags, so the same tag name can mean different things (`element` in XML Schema, in set theory, in chemistry). A namespace prefix points to a unique identifier, a **URI**, in practice usually a URL. The URL does **not** have to lead to a web page, it only needs to be unique. For XSLT and XML Schema the address does show the specification, but that is not required.
- `xmlns:prefix="URI"` declares a prefix. `xmlns="URI"` declares the **default namespace**: unprefixed tags belong to it, only tags from another namespace need a prefix.
- Namespaces are mainly to avoid clashes and tell which vocabulary a tag comes from. The default namespace is merely a shortcut.
- On the slide, the line with `xmlns="..."` after the prefixed example is the opening tag of the alternative notation, not a closing tag.

```
<w:winecellar xmlns:w="http://example.org/winecellar">   prefix form
  <w:wine><w:name>Chablis</w:name></w:wine>
</w:winecellar>

<winecellar xmlns="http://example.org/winecellar">       default namespace form
  <wine><name>Chablis</name></wine>
</winecellar>
```

## 6. XPath

- Path expressions that refer to one particular node by walking down the tree. The lecture stressed it is **not a real query language**, just a way to refer to a node. Slide 24 calls it "a simple, declarative language that uses path expressions", so both statements hold: you describe the nodes you want, the engine finds them, but it lacks joins, grouping and result construction.
- Slide examples: `doc("winecellar.xml")/winecellar/wine`, `.../wine[2]` (second wine), `.../wine[price > 20]/name`. In standard XPath, a path without an index returns all matching nodes **(extension)**.

## 7. The processing pipeline (slide 25)

```
XML document ──► XSLT processor ◄── XSL stylesheet
                      │
                      ▼
                 XML parser ◄── DTD or XSD
                      ▲
                      ║  DOM or SAX
                      ▼
                 Application
```

- Unlike a browser with HTML, there is no single application for all XML, because tags are self-defined. What is generic is the **parser**. What happens with the data is application logic, specific per document kind.
- XSLT processor: optional, only if the application expects another format. Parser: checks well-formed, and valid if a DTD or XSD is given. Rejected documents never reach the application. Then the API (DOM or SAX) presents the document to the application.
- The parser is conceptually a separate stage, in practice usually a library your program calls **(extension)**.

## 8. DOM, SAX and StAX: use cases (saved with `!`)

| | DOM (tree of objects) | SAX (stream of events) |
|---|---|---|
| Memory | whole document in memory, can be memory intensive | small footprint, scalable |
| Access | direct access to any part | sequential only, cannot return |
| Many manipulations | well suited (prune, graft, move, delete sub-trees, add elements) | less suited |

- **DOM** (developed by W3C; Document Object Model is my expansion **(extension)**): direct access to parts, many manipulations, document that fits in memory.
- **SAX** (Simple API for XML): events like `start element: wine`, `text: ...`, `end element: wine`. Handler captures relevant events. Fits a big XML dump from a database, or real-time sensors sending XML, reacting for example to every wine before 2010.
- **StAX** (Streaming API for XML): compromise. The application **pulls** data from the stream with a **cursor**, like scrolling through the rows of a SQL query result. More control than SAX, still a stream.

## 9. Storing XML: three approaches

- **Document-oriented**: store the document as a whole, round trip gives the same document. In a relational database as a **CLOB** (character large object) or an XML data type (SQL/XML). Plain SQL only retrieves the whole cell, looking inside needs user-defined functions (term search) or an XML type with functions such as XPath. Good if the document matters and is mostly static. Dedicated document stores (next chapter) also store documents whole.
- **Data-oriented**: **shredding**: cut the document into pieces in individual cells via a mapping. SQL can query single values. Round trip gives the same data, not necessarily the same document. Good if the data matters and you want fine-grained access.
- **Combined**: see 10.

Example (illustrative): staff table with a resume column. CLOB version holds `<resume>...</resume>` per cell. Shredded version has columns degree and employer.

## 10. The combined approach and shredding depth

- Shred what you query, keep free-form parts as XML fragments. Lecture examples: insurance claim (date, claim number, contract number, address shredded, prose explanation as XML fragment), employees (name and number shredded, resume as XML).
- Shredding everything makes rebuilding the whole document need many joins, which is expensive, so you may shred only down to some level.
- The DBMS may pick a shredding level from past usage, or store both a shredded and a whole version and use whichever suits the query. Keeping a full copy in its own cell avoids joins at the cost of extra storage and keeping the copies in sync **(extension: trade-offs)**.

## 11. Why mapping is hard (1/2): shape

| XML | Relational problem | Relational fix | Object-relational fix |
|---|---|---|---|
| Ordered elements | tuples have no logical order | extra position column | list collection type (ordered) |
| Nesting (hierarchy) | first normal form forbids nested relations | separate table with foreign key (a relationship, not necessarily part-of) | nested / composite types |
| Repeated children (several grapes) | first normal form forbids multivalued attributes | additional table | collection types, so shredding is less deep |

- Composite = one value made of named parts of different kinds (country, region). Collection = several values of the same kind. Both put more than one atomic value in a cell **(extension: this contrast)**.
- If the order of repeated elements matters (also grapes), the extra table needs its own position column too.

## 12. Why mapping is hard (2/2): types and rigidity

- Relational: only atomic built-in types (integer, string, date). Plain XML: no types, everything characters. DTD: only #PCDATA. XML Schema: atomic and aggregated types. Aggregated types are modeled with user-defined types in object-relational databases, and with multiple tables in a plain relational one.
- XML is semi-structured: anomalies, missing or repeated values are possible. Relational is rigid: a change to the DTD or XSD (a new field) means regenerating the tables and populating them again, which is expensive. Rigidity is not simply bad: it enforces constraints more strongly and so guarantees data quality, at the cost of flexibility.

## 13. Mapping techniques and table-based mapping

Four techniques, sorted by where the mapping starts:

| Technique | Starts from |
|---|---|
| Table-based | the relational structure, gives a deterministic XML format |
| Schema-oblivious | the XML structure, imposes a rigid relational structure |
| Schema-aware | the conceptual (domain) model behind the schema |
| SQL/XML | an extension of SQL to cope with XML in a relational database |

Only table-based was covered in the lecture, the others continue next week. Table-based structure:

```
<database>
  <table>
    <row>
      <column1>Chablis</column1>      (column tags may be renamed: <name>, <year>)
      <column2>2019</column2>
    </row>
  </table>
</database>
```

- Perfect one-to-one mapping, simple, very deterministic. Disadvantage: rigid XML structure (can be mitigated with XSLT, slide 39). Convenient for dumping a relational database or migrating data from one relational database to another. Slide 39 adds that the structure can be implemented with an updatable SQL view, not mentioned by the professor.

## Not covered by the recorded lecture

Slides 40 to 79: schema-oblivious and schema-aware mapping, SQL/XML, searching XML (full-text, keyword-based, structured search with XQuery) and the whole JSON part. Also the sample exam questions on physical database organization discussed after the break (earlier chapters).

## Glossary quick list

CLOB: character large object. DTD: document type definition. XSD: XML Schema Definition. XSLT: XSL Transformations. URI: uniform resource identifier. PCDATA: parsed character data (strings). DOM: tree API. SAX: Simple API for XML. StAX: Streaming API for XML. API: application programming interface.

## Tutoring corrections made during the session

- Point 1 first skipped comments and processing instructions, added later.
- XPath: the slide says declarative, the professor said not a real query language. Both are noted above.
- Quiz wording on XSLT outputs was loosened: the two outputs are the main ones, not the only ones.
