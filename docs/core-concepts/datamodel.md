---
icon: lucide/component
---

# Understanding Data Models

Welcome to the Data Models reference guide. APGAP is built on a modular architecture composed of several distinct classes, each designed to handle specific data entities and logical operations.

To ensure clarity and ease of navigation, we have documented these classes separately for each component of the applicatiopn. This granular approach allows you to focus on the specific data structures relevant to your current integration task without navigating through unrelated schemas.

## Metadata Data Model

The Metadata Data Model is used to add metadata tags to sequence files. Each tag consists of a key and a value. Platform Administrators can add and edit metadata keys and define what type of value a key has.  

```mermaid
---
title: Metadata tags
---

classDiagram
    Key: +String name
    Key: +String data_type
    note for Key "datas_type can be:<br>text<br>number<br>date<br>select<br>GPS"

    Value: +String name
    note for Value "Instances of Value hold the actual<br> value of a metadata tag on a<br> file."

    MetadataTag: +Key key
    MetadataTag: +Value value
    MetadataTag: +[Dataset] dataset
    MetadataTag: +[File] file
    `MetadataTag` --> `Key`
    `MetadataTag` --> `Value`
    note for MetadataTag "Instances of MetadataTag hold the<br> references to the key of the tag and the<br> actual value. Values are being reused, <br>so multiple files/datasets can<br> point to the same metadata tag."
    
    SourceType: +String name
    SourceType: +int sort_order
    note for SourceType "e.g. Human Host, Wastewater Sample, etc."
    
    MetadataTemplate: +Key key
    MetadataTemplate: +String name
    MetadataTemplate: +SourceType source_type
    MetadataTemplate: +bool is_core
    `MetadataTemplate` --> `SourceType`
    `MetadataTemplate` --> `Key`
    note for MetadataTemplate "instances of MetadataTemplate<br> map a key to the source type"
 
```