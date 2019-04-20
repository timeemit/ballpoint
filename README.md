# Ballpoint

Stop writing your Swagger file by hand and compose with __Ballpoint__

Ballpoint implements opinionated RESTful resource scaffolding that offers:

- Pagination with Filtering
- Summary queries with Filtering
- Bulk insert
- Bulk update
- Individual deletion

Ballpoint lets you build quickly without the hangover of technical debt.  Where
previous web development imposes technology lock-in, Ballpoint frees developers
because it is __language ambivolent__.

## Language Ambivolence?

Execution of `ballpoint compose` produces only Swagger files: static `.yml` or
`.json` files that can be used to generate code for _both_ clients _and_
servers.  This makes refactoring hasty prototypes into sturdy implementations
significantly less painful and makes Ballpoint ideal for greenfielding the full
stack of a web, mobile application, or both.

## Install

```bash
npm install -g ballpoint
```

## Use

Add operations for a new model with an array of properties:

```bash
ballpoint compose model property1[:type1][:modifier1] [property2[:type2][:modifier2] ...]
```

A model's property type defaults to a string but can otherwise be anything from
the possible data type [common
names](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/2.0.md#data-types):

- integer
- long
- float
- double
- string
- byte
- binary
- boolean
- date
- dateTime
- password

Modifiers further enrich the data type.  Currently supported modifiers are:

- indexed
- key

The `indexed` modifier suggests to __Ballpoint__ that the API should support
sorting, filtering, and grouping by the indicated property.

The `key` modifier implies the same as `indexed` but specifies that any
record is uniquely retrievable by this property.  Multiple properties specified
as `key` are assumed to be part of the same composite index.

## Example

Suppose we run the following command:

```bash
ballpoint compose notes id:key archived:boolean:indexed content created_at:dateTime:indexed
```

The compose command will check for a Swagger file at _./swagger/swagger.yml_
relative to your current working directory.

If it's absent then it'll be initialized with a boilerplate swagger that
accepts HTTP JSON requests at the URL localhost:10010.  Either way, this root
swagger file is then further modified to include the following definitions,
paths, operations, and responses.

### Paths

The following paths & operation will be added:

Purpose | Operation | HTTP Verb | Path
--- | --- | ---
Index | NotesIndex | GET | /notes
Group | NotesGroup | GET | /notes/group
Create | NotesCreate | POST | /notes
Show | NotesShow | GET | /notes/{id}
Replace | NotesReplace | PUT | /notes/{id}
Update  | NotesMupdate | PATCH | /notes/{id}
Destroy | NotesDestroy | DELETE | /notes/{id}

### Definition

This will add the following definition to the swagger file specifed at
`./swagger.yml`:

```yaml
parameters:
  page_number:
    name: page_number
    in: query
    description: Page of a paginated response
    required: false
    type: integer
    minimum: 1
    default: 1

  page_size:
    name: page_size
    in: query
    description: Size of a paginated response
    required: false
    type: integer
    minimum: 1
    default: 1

  sort_ascending:
    name: sort_ascending
    in: query
    description: Direction to sort by
    required: false
    type: boolean
    default: false

  notes_sort_column:
    name: sort_column
    in: query
    description: Column to sort notes by the filter by
    required: false
    type: string
    enum:
      - id
      - archived
      - created_at
    default: created_at

  notes_filter_archived:
    name: filter_archived
    in: query
    description: Filter notes by the "archived" property
    required: false
    type: boolean

  notes_filter_created_at:
    name: filter_created_at
    in: query
    description: Filter notes by the "created_at" property
    required: false
    type: string
    format: dateTime

definitions:
  NotesInstance:
    properties:
      id:
        type: string
      content:
        type: string
      created_at:
        type: string
        format: dateTime
```

#### Index

The index path will have the following parameters

```yaml
/notes
  get:
    operation: index_models
    parameters:
      - "#/parameters/page_number"
      - "#/parameters/page_size"
      - "#/parameters/sort_ascending"
      - "#/parameters/note_sort_column"
      - "#/parameters/note_filter_id"
      - "#/parameters/note_filter_created_at"
    responses:
      "200":
        description: Success
        schema:
          type: array
          items:
            $ref: "#/definitions/NotesInstance"
```

#### Group

The group path will have the following parameters

```yaml
/notes/group
  get:
    operation: group_models
    parameters:
      - "#/parameters/page_number"
      - "#/parameters/page_size"
      - "#/parameters/sort_ascending"
      - "#/parameters/note_sort_column"
      - "#/parameters/note_filter_id"
      - "#/parameters/note_filter_created_at"
      - name: notes_group
        in: query
        description: Notes property to group by
        required: true
        type: string
        enum:
          - id
          - archived
          - created_at
    responses:
      "200":
        description: "Success"
        schema:
          type: array
          items:
            type: object
            properties:
              key:
                type: string
              count:
                type: number

```

#### Update

The group path will have the following parameters

```yaml
/notes
  get:
    operation: group_models
    parameters:
      - name: id
        in: body
        description: IDs to determine
        type: array
        items: string
        required: true
      - name: content
        in: body
        description: Set the content property for each of the specified notes
        required: false
        type: string
      - name: archived
        in: body
        description: Set the archived property for each of the specified notes
        required: false
        type: boolean
      - name: created_at
        in: body
        description: Set the created_at property for each of the specified notes
        required: false
        type: string
        format: dateTime
```

## Common Options

Specify the swaggerfile content type:

```bash
--format=[json|yaml]
```

Override the destination swagger file:

```bash
--file=[/absolute/path.yml|./relative/path]
```

## OpenAPI

Swagger was officially renamed into OpenAPI in 2017 when the specification went
through a breaking revision to version 3.  Many of the changes are very
sensible but Ballpoint will focus on version 2 until the deficit of tools for
the newer version is remedied.

## The Name

From [Wikipedia's Ballpoint Pen](https://en.wikipedia.org/wiki/Ballpoint_pen)

> A ballpoint pen was conceived and developed as a cleaner and more reliable
> alternative to dip pens and fountain pens, and it is now the world's
> most-used writing instrument: millions are manufactured and sold daily.

The major purpose of this library is to make API composition fast, easy, and
free of technical debt.
