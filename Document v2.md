# Hone document format, version 2

Hone document format version 2 is a superset of [format version 1.](Document.md) Everything in v1 applies equally to v2, and is not repeated here.

Version 2 adds custom types. In version 1, the only supported parameter types were `int`, `float`, `bool`, `string`, `color` and `font`. These continue to work the same way in version 2, but in addition to them, documents can contain custom type specifications, and values with these custom types.



## Manifest changes in v2

The format is `2`.

If the current document contains custom types, the manifest contains an entry for `types.yaml`. Using custom types is optional: if there are no custom types present (no entry for `types.yaml` in manifest), a v2 document is functionally equivalent to a v1 document.

Manifest example with format 2 and custom types:

    format: 2
    project_identifier: 1234abcd
    types.yaml: WvO1pS/mA0ztlVnq2oJe9u3if+ynvpPDE6+sgi1ncPQ=
    resources:
      default:
        values.yaml: EElYe2kjuTU5lIiYJF+s77TGP7Pkb4qQLC4suN2lkO4=



## types.yaml — custom types specification

`types.yaml` is the specification of custom types in individual Hone documents. It is located at the root of a Hone document package, next to `manifest.yaml`.

There are two main uses for the types specification:

1. the Hone tool uses information about the types to generate the correct editing UI for documents with custom types

1. individual Hone loader libraries on any platform can use the types file to validate the custom types possibly present in a document. If there is a value with a custom type in any `values.yaml`, the library can assume that `types.yaml` contains a corresponding type specification. If there is no matching type specification for a custom type, this can be considered an error.

The custom types don’t contain any semantics and aren’t specific to any platform. The goal of the custom types is to enable expressing richer data structures than is possible with the previous Hone primitive types. The semantics of the types are the domain of each individual loader library. The libraries may have built-in knowledge of some custom types and possibly provide hooks to the library users to add their own custom processing for some types.

For a complete example of a custom types file, see the [Android custom types.](https://github.com/honetools/android/blob/master/app/src/main/assets/example.hone/types.yaml) What follows is a discussion and breakdown of the format.

There are four kinds of custom types—aliases, enums, font name, and compound types.



### Alias type

    lineSpacing:
      name: Line spacing
      behavior: alias
      backing_type: float
      can_add_in_tool: 1

The alias type specifies an alias for an existing primitive type. This may be useful on its own, and it comes handy in compound types seen later.

`name` is the user-visible name that’s used in the tool UI when displaying and editing values of this type.

`backing_type` refers to one of Hone’s primitive types, and this type is used to actually hold the value.

`can_add_in_tool` specifies whether the user should be allowed to directly add values of this type in the Hone tool. If this is 0, this aliased type is not used on its own; rather it’s used as part of a compound type only.



### Enum type

    NSTextAlignment:
      name: Text alignment
      behavior: enum
      backing_type: int
      can_add_in_tool: 1
      values:
      - 0: Left
      - 1: Center
      - 2: Right
      - 3: Justified
      - 4: Natural

Enums specify a selection for the user to choose from. Enums are backed by Hone’s primitive values (int, float or string). The `values` array specifies the list of possible values for this enum, as well as the display string for each value. The display string is shown to the user in Hone tool.



### Font name type

    FontName:
      name: Font name
      behavior: font_name
      can_add_in_tool: 1

Font name implies that the backing type is string. From the library’s perspective, the behavior is identical to a string. The functionality of the font name field in the tool is that instead of a freeform string, the tool displays a button to pick the font, where the list of available fonts is retrieved from the connected device.



### Compound type

    NSParagraphStyle:
      name: Paragraph style
      behavior: compound
      can_add_in_tool: 1
      values: [ alignment~NSTextAlignment, lineSpacing~lineSpacing, paragraphSpacing~float ]

Compound type refers to other custom and primitive types that make up its value. Here, we see that a paragraph style value consists of three parts: a text alignment enum (seen above), a line spacing aliased value, and a floating-point paragraph spacing value.



## Using custom types

The above type specifications may seem verbose, but they provide for a tight format for the actual values. Given the above custom types, here’s what an entry in an actual `values.yaml` file might look like:

    paragraphObject:
      style~NSParagraphStyle:
        alignment: 2
        lineSpacing: 1.5
        paragraphSpacing: 1.5

We see that we refer to the NSParagraphStyle compound type for this value, which has three subvalues. We don’t need to specify anythign about the subvalues here since they are specified in types.yaml.
