# Sway document format

The Sway document is a collection of values and resources relating to one app whose values are managed with Sway.

On the editing side, the tool edits the values in this document.

On the device side, the device/library loads values from this document.

The job of Sway cloud service is, among other things, to efficiently get the document from the tool to devices.



## Design goals

* **Human-readable and -editable.** Humans should be able to just open the document and edit it with any tool of their liking.

* **Machine-readable and -editable.** It must be based on existing serialization formats with widespread tool support, so existing tools can be modified to easily work with the format.

* **Version-control-friendly.** The document format is explicitly designed to be friendly to version control systems.

* **Consistent serialization.** The format must be consistently deserializable from disk representation to memory, and vice versa. The same in-memory representation must always produce the same serialization. This also helps with version-control-friendliness.


## Container

A Sway document is a folder container with .sway extension. This is the package bundle document that the Sway tool works with. On the file system and version control level, though, it is just a folder with specified format and contents.

A Sway container layout is as follows:

    manifest.yaml
    default/
      values.yaml
    lightTheme/
      values.yaml

The container consists of a manifest, and one or more theme folders. Each theme folder contains a values file, and in future versions of the format, resource files.

YAML is used as the base format for the key/value stores (manifest and values).



### manifest.yaml

The manifest file specifies metadata about the document, as well as list of the other document resources and their checksums in the form of base64-encoded SHA256 hashes. A client can bootstrap or verify its state by obtaining the current version of the manifest, and comparing its contents with the data it has locally.

An example manifest.yaml file:

    - document_version: 1
    - app_token: 8Ap4f3SSqV4WW0cHAvT+k3NYP73AJbLIvfAmLMSPz/Q=
    - resources:
      - default
        - values.yaml: 6jKTFKr7U8CKUUGvSkrlS71ahtu3cD2lNy70EBPRvXg=
      - lightTheme
        - values.yaml: fSDObOmAC1XzreIzoOSAJfo6ueg0OkuoHsmaT/HSgwI=

`document_version` defines the Sway document version. Currently, only version 1 is defined.

`app_token` is the token to be used to identify the app in the Sway cloud service.

`resources` is a representation of each theme folder together with the resources it contains. Each resource is listed with the checksum of its contents calculated over the bytestream using SHA256 and output here as base64.

The “default” theme must always be present. It must contain all values and assets that can be possibly requested by clients, as it is the base/fallback used in case other themes do not contain the requested value/asset.



## values.yaml

The values file specifies the objects, keys and values specified by one theme. It has a two-level namespace, with an array of document objects, and each object containing an array keys and values.

An example values.yaml file:

    - Items:
      - background~color:
        - 1
        - 0
        - 0
        - 1
      - top_margin~float: 48
      - items~int: 6
      - someTitle~font:
          typeface: “Helvetica Light”
          size: 14
    - BackButton:
      - leftPadding~float: 24.0

This document contains two top-level objects, “Items” and “BackButton”. “Items” contains a background color, a top margin, an item count, and a font. “BackButton” contains left padding.

The top-level object keys are just names that are relevant to the application being developed. In object-oriented architecture, they might be the same as class names, or they might use another naming scheme if values from a given category are used by more than one object. The top-level object names must be unique.

The key names within one object should reflect the item in the application that the key represents. Typically, this would be “top_margin”, “left_padding”, “title”, “background” etc. The key names are suffixed by the Sway data type, separated by ~ (tilde), so the final key name in the document might be “top_margin~float”, to represent the top margin of a given object as a floating-point value.

The following data types are defined in Sway document v1.

### float

A signed floating point value with arbitrary precision. May be specified in the document with or without decimals.

    topMargin~float: 24
    left_padding~float: -24.0
    swipeTimeout~float: -0.4

### int

A signed integer value.

    items~int: 6
    negativeSteps~int: -2

### font

A font specification that consists of a typeface and size, expressed as a dictionary. Currently, no guarantees are made for the font being actually available on the target device.

    title~font:
      typeface: "Helvetica Neue"
      size: 14

### color

An array of RGBA values in device RGB color space, all expressed as float values between 0.0 and 1.0.

    background~color:
       - 0
       - 1
       - 0
       - 0.32
