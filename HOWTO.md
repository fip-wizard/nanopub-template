# How to use Nanopub Template

## Nanopublication

Each nanopublication generated using this template is RDF that contains:

* identifier (base URI)
* prefixes
* set of assertions (triples)
* provenance information
* publication information

Optionally, it is possible to enable creation of so-called index nanopublication that links to all of the other created nanopublications.

## Customization in KM

The main idea of this template is customization through annotations in Knowledge Model. The template itself defines only a set of variables and order of traversal in which goes through the KM entities and retrieves data for creating nanopublications using annotations and replies. Therefore, it lays no assumptions about nanopublication contents (statements, provenance, nor publication information). The template traverses in DFS manner and does certain tasks based on annotations both before and after traversal of its children.

The customization is done using setting variables, providing configuration options, and supplying Jinja2 template fragments that are rendered in the template.

Only these entities are used: knowledge model, chapter, question, answer, choice. Any other KM entity is ignored by this template (e.g. reference, expert, or phase).

## Pre-set variables

There are few pre-set variables that the template defines and KM annotations can use in Jinja2 template fragments:

* `config.dsw_url` = string value of DSW instance URL (can be used for publication info)
* `config.timestamp` = current timestamp in ISO format (e.g. `"2022-01-01T12:00:00Z"`)
* `qtn.uuid` = UUID (string) of the questionnaire
* `qtn.name` = name (string) of the questionnaire
* `qtn.description` = description (string) of the questionnaire
* `qtn.tags` = project tags (list of strings) of the questionnaire
* `doc.uuid` = UUID (string) of the questionnaire
* `entity.uuid` = UUID of the current entity

## Annotations

* `nanopub.var.set.<name>` = set a variable by providing name and value, value is always plain string, name must comply with Python variable naming rules (a-z, A-Z, 0-9, underscore)
* `nanopub.var.j2.<name>` = set a variable by providing name and value, value is always Jinja2 template, name must comply with Python variable naming rules (a-z, A-Z, 0-9, underscore)

### Defaults

* `nanopub.default.prefixes`
* `nanopub.default.uri`
* `nanopub.default.statements`
* `nanopub.default.pubinfo`
* `nanopub.default.provenance`
* `nanopub.default.before`
* `nanopub.default.after`

### Nanopub creation

A nanopublication can be created on any entity that has subtrees (KM, chapter, answer, list question):

* `nanopub.uri` = instruction to create a new nanopublication, it may contain Jinja2 template for URI (if empty, then the one from `nanopub.default.uri`)
* `nanopub.prefixes`
* `nanopub.statements.start`
* `nanopub.statements.end`
* `nanopub.pubinfo`
* `nanopub.provenance`
* `nanopub.before`
* `nanopub.after`

Then, any child of that entity can contribute by either setting a variable that is then used in `nanopub.statements.end` Jinja2 template or directly add statements using `nanopub.statements.add`. It is naturally not possible to define a new nanopub inside other but variables are all in global scope (can be redefined).

### Reply as variable

Natural case is to set or use variable to represent reply of user in questionnaire. For that, on a question you can use `nanopub.var.reply` and provide name of the variable. Based on type of question, the variable will contain a certain information:

* Value question = string value
* Integration question = object with `value` and `id` attributes
* Options question = object with `value` and `id` attributes, where `value` is label (string) of the selected option and `id` its UUID
* Multi-choice question = list of objects with `value` and `id` attributes, where `value` is label (string) of the selected choices
* List question = dict objects with `id` (UUID of the item) as keys and then variables set inside the list, for example, if you have a question under list question that sets variable `foo` (as custom variable or reply), there will be attribute `foo` with its value

If question is not answered, the reply is set to `None`.

### Nanopub index

Details for nanopublication index must be specified using KM annotations:

* `nanopub.index` = `true` / `false` (default: `false`)
* `nanopub.index.uri` = Jinja2 template for constructing URI of index nanopub
* ...

## Nanopublication templates

### Basic nanopublication

It automatically creates the base prefix with URI based on variables and adds `np` prefix.

```turtle
# $nanopub.default.before
# $nanopub.before
@prefix : <$nanopub.uri> .
@prefix np: <http://www.nanopub.org/nschema#> .
# $nanopub.default.prefixes
# $nanopub.prefixes

:Head {
  : a np:Nanopublication ;
    np:hasAssertion :assertion ;
    np:hasProvenance :provenance ;
    np:hasPublicationInfo :pubinfo .
}

:assertion {
  # $nanopub.default.statements
  # $nanopub.statements.start
  # $nanopub.statements
  # $nanopub.statements.end
}

:provenance {
  # $nanopub.default.provenance
  # $nanopub.provenance
}

:pubinfo {
  # $nanopub.default.pubinfo
  # $nanopub.pubinfo
}
# $nanopub.after
# $nanopub.default.after
```

### Index nanopublication

Notice that index does not use the defaults for basic nanopublications. It automatically uses `npx:includesElement` with all basic nanopublications created. It already has both `np` and `npx` prefixes.

```turtle
# $nanopub.index.before
@prefix : $nanopub.index.uri .
@prefix np: <http://www.nanopub.org/nschema#> .
@prefix npx: <http://purl.org/nanopub/x/> .
# $nanopub.index.prefixes

:Head {
  : a np:Nanopublication ;
    np:hasAssertion :assertion ;
    np:hasProvenance :provenance ;
    np:hasPublicationInfo :pubinfo .
}

:assertion {
  : npx:includesElement $NANOPUBS .
  # $nanopub.index.statements
}

:provenance {
  # $nanopub.index.provenance
}

:pubinfo {
  # $nanopub.index.pubinfo
}
# $nanopub.index.after
```
