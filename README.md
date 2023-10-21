# udxf-spec
UUID Data Exchange Format (UDXF) specifications

**Version 0.1.0**

2023-10-21

_By Yuka MORI (https://github.com/metastable-void)_

## Objectives

- Define a universal and simple structure for structured data.
- Define a brief binary/text serialization format for data structured in this manner.
- (Optional) Define an embeddable form of serialization format for e.g. JSON.

## Methodology

- We use UUIDs for identifying types of objects and for identifying keys of dictionaries.
- We define only one primitive type for this format, i.e. a byte array.
- Meanings of data are interpreted based on Object-Type UUIDs and Dictionary-Key UUIDs. No particular definition of concrete meanings of these UUIDs is given in this specification.

## Structure

- A UDXF value is either an array of bytes (octets) or an UDXF object.
- An UDXF object has an Object-Type UUID for identifying the type of that object.

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
- What kind of data types and processing are assumed for a field of bytes is to be inferred from the corresponding Dictionary-Key UUID and/or the Object-Type UUID of the owner object.

### Two interpretation for Lists

There are two possible interpretations for a List defined in this specification.
One MUST choose and define one for a type of Objects identified by an Object-Type UUID.

#### The array interpretation

The List is interpreted as an array of a particular data type.
This specification does not forbid one from including both Objects and Bytes in one such array as long as a uniform semantics is defined for that Object type.

#### The enum interpretation

The List is interpreted as an enum of possibly-different types, where each of the fields are interpreted by the position of the field.
Variable-length enums are allowed in this specification, as long as these meanings are defined for that Object type.

## Serialization

The canonical serialization format is the binary serialization format defined below.

### Text serialization format (UDXF-Text)

- UUIDs in UDXF-Text format are encoded in hexadecimal 8-4-4-4-12 format with one prepended less-than "<" symbol and one appended greater-than ">" symbol.

```
<uuid> := "<" xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx ">"
```

where x's are hexadecimal digits.

### Binary serialization format (UDXF-Binary)

- This format is based on an array of bytes, and thus it is a byte stream format. (i.e. we do not define any bit order, for example.)

### JSON embedding

