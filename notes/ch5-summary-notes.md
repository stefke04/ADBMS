# Chapter 5 summary notes (items marked with !)

## Slide 25: processing XML documents (schema/diagram)
Student instruction: "include the schema of slide 25 in the summary".
Include the diagram (notes/slide-25.png). Flow:
- XML document -> XSLT processor (the XSL stylesheet is an extra input to it)
- XSLT processor -> XML parser (the DTD or XSD is an extra input to it)
- XML parser <-> DOM or SAX (double arrow) <-> Application
Lecture points to attach: XSLT processor is optional (only if the application expects another format);
the parser checks well-formed and, with a DTD or XSD, valid; non-well-formed or invalid documents are rejected;
the application only sees accepted documents; DOM or SAX is how the application gets at the content.
