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

We define only one **atom** (primitive type) for this format, i.e. a **BLOB** (byte array).

We use **UUIDs** for identifying types of values and for identifying labels of labeled Structs.

A type of a value is determined by the associated **Constructor UUID**.
Constructor UUIDs should be able to be omitted to save space.
When omitted, that is the same as the **NULL UUID**.

No particular definition of concrete meanings of these UUIDs is given in this specification.

## Structure

A **UDXF value** is either a **BLOB** (an array of bytes, or octets), a **List**, or a **Struct**.
We call this distiction the **Form** of a value.

We call a List and a Struct the **Compound** Forms of values.

A UDXF value has a **Constructor UUID** for identifying the type of that value.
The Constructor UUID for a value can be omitted.

A Struct is either an unlabeled Struct or a labeled Struct.

```
<value> = <blob> | <list> | <struct>

<blob> = [ <uuid> ] *<octet>

<list> = [ <uuid> ] "[" *<value> "]"

<struct> = <unlabeled_struct> | <labeled_struct>

<unlabeled_struct> = [ <uuid> ] "{" *<value> "}"

<labeled_struct> = [ <uuid> ] "{" *( <uuid> ":" <value> ) "}"
```

## Semantics

This data model uses a **value-based semantics**. The same values are to be interpreted the same.

A Constructor UUID is a "type constructor" as it makes mere data into a value with a particular meaning.
Constructor UUIDs are for identifying types of values and these are not for identifying individual values.
If you need to encode identities of individual values, you can as always use an in-band encoding of identifiers (i.e. embedded into the values),
to convert the identities into the values.

For Constructor UUIDs, the **NULL UUID** `00000000-0000-0000-0000-000000000000` can be used to represent an unspecified type.
An absence of a Constructor UUID for a value is the same as a value with the NULL UUID for its Constructor UUID.
You SHOULD add a Constructor UUID when there is a type applicable AND the type is not apparent from the location AND optimizing for space is less of a concern.

### Lists

A List represents an one-dimentional array of values of particular type-wise characteristics.
This specification does not forbid one from mixing several Forms of values in one List as long as a uniform semantics is defined for that List type.
However, other specifications extending this specification MAY limit that kind of mixing.

The type of the container List can provide type interpretations for its items possibly with the NULL UUID.

### Structs

The relation between a container labeled Struct and its field value is determined by the corresponding Label UUIDs.
The meaning of an item of a unlabeled Struct is determined by its position inside the Struct.

The type of the container Struct can provide type interpretations for its items possibly with the NULL UUID.

Variable-length Structs are allowed in this specification, as long as these meanings are defined for that Struct type.
However, other specifications extending this specification MAY limit that.

## Serialization

The canonical serialization format is the binary serialization format defined below.

### Text serialization format (UDXF-Text)

UUIDs in **UDXF-Text** format are encoded in hexadecimal 8-4-4-4-12 format with one prepended less-than "<" symbol and one appended greater-than ">" symbol.
**Hexadecimal notation in this format MUST use lowercase digits.**
Also, **UUID notation (including "<>" symbols) CANNOT include white space characters.**
However, there can be ASCII white space characters around the encoded UUIDs.

```
<uuid> = "<" 8<hex_digit> "-" 4<hex_digit> "-" 4<hex_digit> "-" 4<hex_digit> "-" 12<hex_digit> ">"
```

where x's are hexadecimal digits.

A Value can be in any of the three defined Forms.

```
<value> = <blob> | <list> | <struct>
```

#### BLOB serialization

BLOBs in UDXF-Text format are encoded in the standard Base64 with two ASCII double quotation marks '"', one prepended and one appended.
ASCII white-space characters (including newlines) inside a Base64-encoded BLOB are ignored.

A BLOB value can have a Constructor UUID.

```
<blob> = [ <uuid> ] '"' <base64> '"'
```

#### Compound-value serialization

ASCII white-space characters are allowed and ignored between tokens composing a serialization of a Compound value.

**A preceding comma is allowed for the serialization of a Compound value.** A succeeding comma is also allowed.
**A serialization of an empty Compound value MUST NOT contain any comma.**

A List value is serialized as follows:

```
<list> = [ <uuid> ] "[" [ [ "," ] <value> *( "," <value> ) [ "," ] ] "]"
```

A unlabeled Struct value is serialized as follows:

```
<unlabeled_struct> = [ <uuid> ] "{" [ [ "," ] <value> *( "," <value> ) [ "," ] ] "}"
```

A labeled Struct value is serialized as follows:

```
<labeled_struct> = "{" [ [ "," ] <uuid> ":" <value> *( "," <uuid> ":" <value> ) [ "," ] ] "}"
```

#### Examples

A BLOB is itself a valid UDXF expression.
In this case, the Constructor UUID is omitted (meaning the NULL UUID).

```
"BWSbmY92ig3cVYjmzR2Ij7JwPbXWL/zRn8zo+m7eXR4HbcsM3FjpWwM8a/NI"
```

Empty BLOBs are allowed.

```
<9eb7a55e-3f49-4719-9db8-7d47d2f9a61a> ""
```

A simple labeled Struct.

```
<68362700-8574-44b5-8048-c378e95fe3d9> {
    ,<5f498a76-cca9-4c50-8eea-b9b9ff1fda3d>: "mkU84gyMHt9xC40pfPOQZYsJDwg5tdHWqw=="
    ,<b3ebc11b-b51c-4ced-9498-3c7a8df4ef00>: "LjE8ylc+N1LGzwKpUVjY2QK/9g=="
    ,<f06cbebb-e026-4c2b-9f34-15dddc986bea>: "iTs1vvCvccMpSrx6R9HOOh/g266mN5pd+PbN+5QCXQ=="
}
```

A more complex example (a List containing unlabeled Structs).

```
<49058051-8c6e-4a56-afac-3d8c08b7d1bd> [
    ,<21e3208b-fc4d-4113-8800-ce3a96fe4fff> { ,"AZlhan5yJsa35b0YCcGFE9u6GGdGFuSN6XGZFYnsuwbWBfXHvXKmr9A=" ,"S/QqfA==" }
    ,<21e3208b-fc4d-4113-8800-ce3a96fe4fff> { ,"YiYhR2aERVcrAiXQ3e65OlcO/v1nA0nvG2NGqvKGqkysHIHC9QaI" ,"z7Shrw==" }
    ,<21e3208b-fc4d-4113-8800-ce3a96fe4fff> { ,"AZlhan5yJsa35b0YCcGFE9u6GGdGFuSN6XGZFYnsuwbWBfXHvXKmr9A=" ,"S/QqfA==" }
    ,<21e3208b-fc4d-4113-8800-ce3a96fe4fff> { ,"RiguPTgbm4sRgUnvaIcux/IyzSg1Nsh8GVwh+JlxO2D4PEcOU8bNgTPP" }
]
```

### Binary serialization format (UDXF-Binary)

- This format is based on an array of bytes, and thus it is a byte stream format. (i.e. we do not define any bit order, for example.)

### JSON embedding

