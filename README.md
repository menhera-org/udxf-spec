# udxf-spec
UUID Data Exchange Format (UDXF) specifications

## Objectives

- Define a universal and simple structure for structured data.
- Define a brief binary/text serialization format for data structured in this manner.
- (Optional) Define an embeddable form of serialization format for e.g. JSON.

## Methodology

- We use UUIDs for identifying types of objects and for identifying keys of dictionaries.
- We define only one primitive type for this format, i.e. a byte array.
- Meanings of data are interpreted based on Object-Type UUIDs and Dictionary-Key UUIDs. No particular definition of concrete meanings of these UUIDs is given in this specification.

## Structure

```
<value> := <byte_array> | <object>

<object> := <object_type_uuid> <object_body>

<object_body> := <list> | <dictionary>

<list> := "[" <value> * "]"

<dictionary> := "{" ( <dictionary_key_uuid> ":" <value> ) * "}"
```

## Semantics

- Object-Type UUIDs are for identifying classes of objects (i.e. types). These are not for identifying individual objects.
- One SHOULD use an in-band encoding for identifiers of individual objects.

## Serialization

The canonical serialization format is the binary serialization format defined below.

### Text serialization format (UDXF-Text)

### Binary serialization format (UDXF-Binary)

### JSON embedding

