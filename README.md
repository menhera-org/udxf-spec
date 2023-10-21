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
- Meanings of data are interpreted based on Object-Type UUIDs and Dictionary-Key UUIDs. No particular definition of concrete meanings of these UUIDs is given in this specification. There is an exception for the NULL UUID.

## Structure

- A UDXF value is either a BLOB (an array of bytes, or octets) or an UDXF object.
- An UDXF object has an Object-Type UUID for identifying the type of that object.

```
<value> = <blob> | <object>

<object> = <object_type_uuid> <object_body>

<object_body> = <list> | <dictionary>

<list> = "[" * <value> "]"

<dictionary> = "{" * ( <dictionary_key_uuid> ":" <value> ) "}"
```

## Semantics

- This data model uses a value-based semantics. The same values are to be interpreted the same.
- Object-Type UUIDs are for identifying classes of objects (i.e. types). These are not for identifying individual objects.
- For Object-Type UUIDs, the NULL UUID `00000000-0000-0000-0000-000000000000` can be used to represent an unspecified type.
- One SHOULD use an in-band encoding for identifiers of individual objects.
- What kind of data types and processing are assumed for a BLOB field is to be inferred from the corresponding Dictionary-Key UUID and/or the Object-Type UUID of the owner object.

### Two interpretations for Lists

There are two possible interpretations for a List defined in this specification.
One MUST choose and define one for a type of Objects identified by an Object-Type UUID.

#### The array interpretation

The List is interpreted as an array of a particular data type.
This specification does not forbid one from including both Objects and BLOBs in one such array as long as a uniform semantics is defined for that Object type.

#### The enum interpretation

The List is interpreted as an enum of possibly-different types, where each of the fields are interpreted by the position of the field.
Variable-length enums are allowed in this specification, as long as these meanings are defined for that Object type.

## Serialization

The canonical serialization format is the binary serialization format defined below.

### Text serialization format (UDXF-Text)

- UUIDs in UDXF-Text format are encoded in hexadecimal 8-4-4-4-12 format with one prepended less-than "<" symbol and one appended greater-than ">" symbol.
- **Hexadecimal notation in this format MUST use lowercase digits.**

```
<uuid> = "<" 8<hex_digit> "-" 4<hex_digit> "-" 4<hex_digit> "-" 4<hex_digit> "-" 12<hex_digit> ">"
```

where x's are hexadecimal digits.

- BLOBs in UDXF-Text format are encoded in the standard Base64 with two ASCII double quotation marks '"', one prepended and one appended.
- ASCII white-space characters (including newlines) inside a Base64-encoded BLOB are ignored.

```
<blob> = '"' <base64> '"'
```

A Value is either a byte array or an Object.

```
<value> = <bytes> | <object>
```

An Object is composed of an Object-Type UUID and an Object body (a List or a Dictionary).

```
<object> = <uuid> ( <list> | <dictionary> )
```

ASCII white-space characters are allowed and ignored between tokens composing a Dictionary or a List.
**Note that a preceding comma is allowed for a Dictionary or a List.** A succeeding comma is also allowed.
**An empty Dictionary or List MUST NOT contain any comma.**

```
<dictionary> = "{" [ [ "," ] <uuid> ":" <value> *( "," <uuid> ":" <value> ) [ "," ] ] "}"
```

```
<list> = "[" [ [ "," ] <value> *( "," <value> ) [ "," ] ] "]"
```

#### Examples

A BLOB is itself a valid UDXF expression.

```
"BWSbmY92ig3cVYjmzR2Ij7JwPbXWL/zRn8zo+m7eXR4HbcsM3FjpWwM8a/NI"
```

Empty BLOBs are allowed.

```
""
```

A simple Object which is a Dictionary.

```
<68362700-8574-44b5-8048-c378e95fe3d9> {
    ,<5f498a76-cca9-4c50-8eea-b9b9ff1fda3d>: "mkU84gyMHt9xC40pfPOQZYsJDwg5tdHWqw=="
    ,<b3ebc11b-b51c-4ced-9498-3c7a8df4ef00>: "LjE8ylc+N1LGzwKpUVjY2QK/9g=="
    ,<f06cbebb-e026-4c2b-9f34-15dddc986bea>: "iTs1vvCvccMpSrx6R9HOOh/g266mN5pd+PbN+5QCXQ=="
}
```

A more complex example.

```
<49058051-8c6e-4a56-afac-3d8c08b7d1bd> [
    ,<21e3208b-fc4d-4113-8800-ce3a96fe4fff> [ ,"AZlhan5yJsa35b0YCcGFE9u6GGdGFuSN6XGZFYnsuwbWBfXHvXKmr9A=" ,"S/QqfA==" ]
    ,<21e3208b-fc4d-4113-8800-ce3a96fe4fff> [ ,"YiYhR2aERVcrAiXQ3e65OlcO/v1nA0nvG2NGqvKGqkysHIHC9QaI" ,"z7Shrw==" ]
    ,<21e3208b-fc4d-4113-8800-ce3a96fe4fff> [ ,"AZlhan5yJsa35b0YCcGFE9u6GGdGFuSN6XGZFYnsuwbWBfXHvXKmr9A=" ,"S/QqfA==" ]
    ,<21e3208b-fc4d-4113-8800-ce3a96fe4fff> [ ,"RiguPTgbm4sRgUnvaIcux/IyzSg1Nsh8GVwh+JlxO2D4PEcOU8bNgTPP" ]
]
```

### Binary serialization format (UDXF-Binary)

- This format is based on an array of bytes, and thus it is a byte stream format. (i.e. we do not define any bit order, for example.)

### JSON embedding

