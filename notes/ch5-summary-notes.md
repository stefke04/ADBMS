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

## Use cases for DOM, SAX and StAX (slides 26 to 29 and the lecture)
Student instruction: "the use cases for DOM, SAX (and StAX)".
- DOM (tree of objects in memory, W3C): direct access to specific parts and a high number of manipulations
  (prune, graft, move or delete a sub-tree, add elements). Works well if the document is not too big, memory intensive.
- SAX (Simple API for XML, stream of events, handler captures relevant events): sequential access, small memory, scalable.
  Fits a big dump of XML from a database that does not fit in memory, or a real-time stream such as sensors sending XML,
  reacting only to relevant events (for example every wine before 2010). Less suited to random access and heavy manipulation,
  and one cannot return to an earlier event.
- StAX (Streaming API for XML): compromise, the application pulls data from the stream with a cursor
  (like scrolling through a query result row by row), more control than SAX but still a stream of events.
Tutor extension (not from the lecture): the expansion of DOM is Document Object Model.
