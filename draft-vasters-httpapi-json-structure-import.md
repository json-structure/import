---

title: "JSON Structure Import"
category: info

docname: draft-vasters-httpapi-json-structure-import
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date: 2025-03-24
consensus: true
v: 3
area: AREA
workgroup: WG Working Group
keyword:
 - JSON
 - schema
venue:
  group: WG
  type: Working Group
  mail: WG@example.com
  arch: https://example.com/WG
  github: json-structure/import
  latest: https://json-structure.github.io/import

author:
 -
    fullname: Clemens Vasters
    organization: Microsoft Corporation
    email: clemensv@microsoft.com

normative:
  RFC2119:
  RFC3986:
  RFC3987:
  RFC8174:
  JSTRUCT-CORE:
    title: "JSON Structure Core"
    author:
      - fullname: Clemens Vasters
    target: https://json-structure.github.io/core/draft-vasters-httpapi-json-structure-core.html

informative:


--- abstract

This document specifies the `$import` and `$importdefs` keywords as extensions
to JSON Structure Core. These keywords allow a schema to import definitions from
external schema documents.


--- middle

# Introduction {#introduction}

This document specifies the `$import` and `$importdefs` keywords, as extensions
to JSON Structure Core {{JSTRUCT-CORE}}. These keywords allow a schema to import
definitions from external schema documents.

All type reference expressions in JSON Structure Core, `$ref` and `$extends` and
`$addins`, are limited to references within the current schema document.

The `$import` and `$importdefs` keywords enable schema authors to incorporate
external schema documents into a schema.

Imports do not establish a reference relationship between the importing schema
and the imported schema. Imports _copy_ definitions from the imported schema
into the importing schema and those definitions are then treated as if they were
defined locally.

# Conventions {#conventions}

{::boilerplate bcp14}

# The `$import` and `$importdefs` Keywords {#import-and-importdefs-keywords}

The `$import` and `$importdefs` keywords are used to import definitions from
external schema documents into a local namespace within the current schema
document.

A schema processor MUST process the `$import` and `$importdefs` keywords before
processing any other keywords in the schema document.

The result of importing definitions is that the imported definitions are merged
into the local `$defs` section under the designated namespace as if they were
defined locally.

A schema that uses `$import` or `$importdefs` MAY _shadow_ any imported
definitions with local definitions of the same name and in the same namespace,
replacing the imported definition entirely. Local definitions take precedence
over the imported definitions. A shadowing type cannot reference the imported
type that it shadows.

When importing definitions into a local namespace, the processor MUST ensure
that all imported cross-references are resolved within the imported definitions
themselves and not to the local schema. That means that any `jsonpointer`
instance (`$ref` or `$extends` or `$addins`) within imported definitions MUST be
prefixed with the local namespace under which the definitions were imported.
This applies recursively to any imported schema that itself contains imports.

## `$import` Keyword {#import-keyword}

The `$import` keyword is a reference expression whose value is an absolute URI
pointing to an external schema document. It is used to import all type
definitions of the external schema into a local namespace within the current
schema document.

When the keyword is used at the root level of a schema, the imported definitions
are available in the schema's root namespace. When used within the `$defs`
section, the imported definitions are available in the respective local
namespace.

> **Reminder**: Any type declaration at the root level of a schema and any type
> declaration at the root level of the `$defs` section is placed in the schema's
> root namespace per section 3.3 of {{JSTRUCT-CORE}}.

Example for `$import` at the root level:

~~~json
{
  "$schema": "https://schemas.vasters.com/experimental/json-structure-core/v0",
  "$import": "https://example.com/people.json"
}
~~~

Importing into the root namespace within the `$defs` section is equivalent to
the prior example:

~~~json
{
  "$schema": "https://schemas.vasters.com/experimental/json-structure-core/v0",
  "$defs": {
    "$import": "https://example.com/people.json"
  }
}
~~~

One can also import into any local namespace within the `$defs` section:

~~~json
{
  "$defs": {
    "People": {
      "$import": "https://example.com/people.json"
    }
  }
}
~~~

The result of the import is that _all_ definitions from the external schema are
available under the `People` namespace. The namespace structure and any
cross-references that exist within an imported schema, including any imports
that it may have, are unaffected by being imported.

The `$import` keyword MAY be used many times within a schema to import multiple
external schemas into distinct local namespaces.

## `$importdefs` Keyword {#importdefs-keyword}

The `$importdefs` keyword is a reference expression whose value is an absolute
URI pointing to an external schema document.

`$importdefs` works the same as `$import`, with the exception that it only
imports the `$defs` section of the external schema and not the root type.

The purpose of `$importdefs` is to use the type definitions from an external
schema as a library of types that can be referenced from within the local schema
without importing the root type of the external schema.

# Examples {#examples}

## Example: Using `$import` to import an external schema {#example-import-external-schema}

Let the external schema be defined as follows:

~~~json
{
  "$schema": "https://schemas.vasters.com/experimental/json-structure-core/v0",
  "$id": "https://example.com/people.json",
  "name": "Person",
  "type": "object",
  "properties": {
    "firstName": { "type": "string" },
    "lastName": { "type": "string" },
    "address": { "$ref": "#/$defs/Address" }
  },
  "$defs": {
    "Address": {
      "type": "object",
      "properties": {
        "street": { "type": "string" },
        "city": { "type": "string" }
      }
    }
  }
}
~~~

The importing schema uses `$import` to import the external schema into the
"People" namespace. The imported `Person` type is then used in the local schema
as the type of the `person` property:

~~~json
{
  "$schema": "https://schemas.vasters.com/experimental/json-structure-core/v0",
  "type": "object",
  "properties": {
    "person": {
      "type": { "$ref": "#/$defs/People/Person" }
    },
    "shippingAddress": {
      "type": { "$ref": "#/$defs/People/Address" }
    }
  },
  "$defs": {
    "People": {
      "$import": "https://example.com/people.json"
    }
  }
}
~~~

The imported `Person` type from the root of the external schema is available
under the `People` namespace in the local schema, alongside the imported
`Address` type.

The external schema can also be imported into the root namespace of the local
schema by using `$import` at the root level of the schema document—in which case
the imported definitions are available in the root namespace of the local
schema:

~~~json
{
  "$schema": "https://schemas.vasters.com/experimental/json-structure-core/v0",
  "$import": "https://example.com/people.json",
  "type": "object",
  "properties": {
    "person": {
      "type": { "$ref": "#/$defs/Person" }
    },
    "shippingAddress": {
      "type": { "$ref": "#/$defs/Address" }
    }
  }
}
~~~

The following schema is equivalent to the prior example:

~~~json
{
  "$schema": "https://schemas.vasters.com/experimental/json-structure-core/v0",
  "type": "object",
  "properties": {
    "person": {
      "type": { "$ref": "#/$defs/Person" }
    },
    "shippingAddress": {
      "type": { "$ref": "#/$defs/Address" }
    }
  },
  "$defs": {
    "$import": "https://example.com/people.json"
  }
}
~~~

## Example: Using `$import` with shadowing {#example-import-shadowing}

The external schema remains the same as in {{example-import-external-schema}}.

The importing schema uses `$import` to import the external schema into the
"People" namespace. The imported `Person` type is then used in the local schema
as the type of the `person` property. The local schema then also defines an
`Address` type that shadows the imported `Address` type within the same
namespace:

~~~json
{
  "$schema": "https://schemas.vasters.com/experimental/json-structure-core/v0",
  "type": "object",
  "properties": {
    "person": {
      "type": { "$ref": "#/$defs/People/Person" }
    }
  },
  "$defs": {
    "People": {
      "$import": "https://example.com/people.json",
      "Address": {
        "type": "object",
        "properties": {
          "street": { "type": "string" },
          "city": { "type": "string" },
          "postalCode": { "type": "string" },
          "country": { "type": "string" }
        }
      }
    }
  }
}
~~~

## Example: Using `$importdefs` to import the `$defs` section of an external schema {#example-importdefs}

The external schema remains the same as in Example 4.1.

The importing schema uses `$importdefs` to import the `$defs` section of the
external schema into the "People" namespace. The imported `Address` type is then
used in the local schema as the type of the `shippingAddress` property as
before. However, the `Person` type is not imported and available.

~~~json
{
  "$schema": "https://schemas.vasters.com/experimental/json-structure-core/v0",
  "type": "object",
  "properties": {
    "shippingAddress": {
      "type": { "$ref": "#/$defs/People/Address" }
    }
  },
  "$defs": {
    "People": {
      "$importdefs": "https://example.com/people.json"
    }
  }
}
~~~

# Resolving URIs {#resolving-uris}

When resolving URIs, schema processors MUST follow the rules defined in
{{RFC3986}} and {{RFC3987}}

This specification does not define any additional rules for resolving URIs into
schema documents.

# Security and Interoperability {#security-and-interoperability}

- Schema processing engines MUST resolve the absolute URIs specified in
  `$import` and `$importdefs`, fetch the external schemas, and validate them to
  be schema documents.
- Implementations SHOULD employ caching and robust error handling for remote
  schema retrieval.
- External schema URIs SHOULD originate from trusted sources.
- Remote fetching of schemas SHOULD be performed over secure protocols (e.g.,
  HTTPS) to mitigate tampering.
- Excessively deep or circular import chains MUST be detected and mitigated to
  avoid performance degradation and potential denial-of-service conditions.

# IANA Considerations {#iana-considerations}

This document does not require any IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
