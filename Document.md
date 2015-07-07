# Hone document format

The Hone document is a collection of values and resources relating to one project whose values are managed with Hone.

On the editing side, the tool edits the values in this document.

On the device side, the document is typically bundled in the app, and the library loads values from this bundled document.

The job of Hone cloud service is, among other things, to efficiently get the document from the tool to devices.



## Design goals

* **Human-readable and -editable.** Humans should be able to just open the document source and edit it with any tool of their liking.

* **Machine-readable and -editable.** It must be based on existing serialization formats with widespread tool support, so existing tools can be modified to easily work with the format.

* **Version-control-friendly.** The document format is explicitly designed to be friendly to version control systems.

* **Consistent serialization.** The format must be consistently deserializable from disk representation to memory, and vice versa. The same in-memory representation must always produce the same serialization. This also helps with version-control-friendliness.


## Container

A Hone document is a folder container with .hone extension. This is the package bundle document that the Hone tool works with. On the file system and version control level, though, it is just a folder with specified format and contents.

A minimal Hone container layout is as follows:

    manifest.yaml
    default/
      values.yaml

A more complete container with some themes and alias info looks like this:

    manifest.yaml
    aliases.yaml
    default/
      values.yaml
    otherTheme/
      values.yaml
    anotherTheme/
      values.yaml

The container consists of a manifest, and one or more theme folders. Each theme folder contains a values file, and in future versions of the format, resource files.

The `default` theme folder must always be present. In addition, the document may also contain one or more additional themes.

YAML is used as the base format for the key/value stores (manifest, aliases and values).



### manifest.yaml

The manifest file specifies metadata about the document, as well as list of the other document resources and their checksums in the form of base64-encoded SHA256 hashes. A client can bootstrap or verify its state by obtaining the current version of the manifest, and comparing its contents with the data it has locally.

An example manifest.yaml file:

    format: 1
    project_identifier: 5363a960e914803c292bbd4b
    aliases.yaml: 7ghj4FKr7U8CKUUGvSkrlS71ahtu3cD2lNy7f34f=
    resources:
      default:
        values.yaml: 6jKTFKr7U8CKUUGvSkrlS71ahtu3cD2lNy70EBPRvXg=
      lightTheme:
        values.yaml: fSDObOmAC1XzreIzoOSAJfo6ueg0OkuoHsmaT/HSgwI=

`format` defines the Hone document version. Currently, only version 1 is defined.

`project_identifier` is the token to be used to identify the project in the Hone cloud service.

`resources` is a representation of each theme folder together with the resources it contains. Each resource is listed with the checksum of its contents calculated over the resource’s bytestream using SHA256-base64.

`aliases.yaml` is a checksum of the optional aliases file, if present.

The “default” theme must always be present. It must contain all values and assets that can be possibly requested by clients, as it is the base/fallback used in case other themes do not contain the requested value/asset.



## values.yaml

The values file specifies the objects, keys and values specified by one theme. It has a two-level namespace, with a dictionary of document objects, and each object containing a dictionary keys and values.

Standard YAML serialization does not guarantee preserving the ordering of dictionary keys across invocations. The Hone document loader/saver uses a modified serializer that treats the top-level objects, and the list of values in each object, as arrays and preserves their ordering across saves.

An example values.yaml file:

    CollectionEntries:
      background~color: [ 1, 0, 0, 1 ]
      top_margin~float: 48
      items~int: 6
      someTitle~font:
        typeface: “Helvetica Light”
        size: 14
    BackButton:
      leftPadding~float: 24.0

This document contains two top-level objects, “CollectionEntries” and “BackButton”. “CollectionEntries” contains a background color, a top margin, an item count, and a font. “BackButton” contains left padding.

The top-level object keys are just names that are relevant to the application being developed. In object-oriented architecture, they might be the same as class names, or they might use another naming scheme if values from a given category are used by more than one object.

The key names within one object should reflect the item in the application that the key represents. Typically, this would be “top_margin”, “left_padding”, “title”, “background” etc. The key names are suffixed by the Hone data type, separated by ~ (tilde), so the final key name in the document might be “top_margin~float”, to represent the top margin of a given object as a floating-point value.

The top-level object names must be unique. The value names must be unique within each object. For example, there may not be two objects called `container`. One `container` object may not have two values called `background`. However, there may be values called `background1` and `background2` in `container`. If there are objects `container` and `item`, both of them may have a value called `background`.

The following data types are defined in Hone document format v1.

### bool

A boolean with two possible values: 1 (“yes”, “on”) or 0 (“no”, “off”).

    enabled~bool: 1
    shouldReallyHappen~bool: 0

### float

A signed floating point value with arbitrary precision. May be specified in the document with or without decimals.

    topMargin~float: 24
    left_padding~float: -24.0
    swipeTimeout~float: -0.4

### int

A signed integer value.

    items~int: 6
    negativeSteps~int: -2

### string

A string value. In YAML, strings without spaces don’t have to be enclosed in quotes.

    title~string: "Hello world"
    label~string: Welcome

### font

A font specification that consists of typeface, size, number style and spacing expressed as a dictionary. Currently, no guarantees are made for the font being actually available on the target device, or the font supporting the number style and spacing OpenType attributes.

    title~font:
      typeface: "Helvetica Neue"
      size: 14
      number_style: oldstyle
      number_spacing: proportional

Allowed values for `number_style` are `regular` (default) and `oldstyle`. Allowed values for `number_spacing` are `proportional` (default) and `monospaced`.

The number_style and number_spacing keys are optional. If not present, they must be treated as having the default values for this font.

### color

An array of RGBA values in device RGB color space, all expressed as float values between 0.0 and 1.0.

    background~color: [ 0, 1, 0, 0.32 ]



## aliases.yaml

This file defines “aliases”, or “nice names”, for themes, objects and values. The goal of aliases is to provide friendlier names to these items. The original object and value names are defined by the engineer in code and cannot be easily changed, but the engineer-defined names may not be the most meaningful to the non-technical Hone tool users. The aliases helps to bridge this gap by letting the tool users easily rename the items to something more meaningful.

Using aliases is completely optional. Only some, or all, or no themes, objects and values can have aliases defined. If there are no aliases defined, the `aliases.yaml` file, and the corresponding entry in the document’s `manifest.yaml` may not exist.

An example aliases file:

    themes:
      default:
        alias: Theme 1
    objects:
      container:
        alias: Widgets
        values:
          background: "Background color"
          borderStroke: "Border width"

The top-level keys are “objects” and “themes”.

“objects” contains the same objects that are defined in `values.yaml`. If the object itself has an alias, it is under the `alias` key. If any of the object’s values have aliases, these are defined in the `values` dictionary.

“themes” contains the themes defined in the document. At the very least, it contains a “default” theme since one is always present in all Hone documents. Initially, the default theme’s alias is “Theme 1” but the user can rename it. All additional theme aliases will also be listed here if the user has created more themes and has renamed them.

The naming rules for aliases are the same as in `values.yaml`:

* Theme aliases must not be the same as alias or real name of any other theme.
* Object aliases must be unique and must not be the same as alias or real name of any other object.
* Value aliases must be unique within the same object and must not be the same as alias or real name of any other value within the same object.
