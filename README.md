# file_upload_nullability_issue_reproduce

## Router config

```yaml
preview_file_uploads:
  enabled: true
  protocols:
    multipart:
      enabled: true
      mode: stream
      limits:
        max_file_size: 512kb
        max_files: 5
include_subgraph_errors:
  all: true
```

## Supergraph schema

```graphql
schema
  @link(url: "https://specs.apollo.dev/link/v1.0")
  @link(url: "https://specs.apollo.dev/join/v0.3", for: EXECUTION)
{
  query: Query
  mutation: Mutation
}

directive @join__enumValue(graph: join__Graph!) repeatable on ENUM_VALUE

directive @join__field(graph: join__Graph, requires: join__FieldSet, provides: join__FieldSet, type: String, external: Boolean, override: String, usedOverridden: Boolean) repeatable on FIELD_DEFINITION | INPUT_FIELD_DEFINITION

directive @join__graph(name: String!, url: String!) on ENUM_VALUE

directive @join__implements(graph: join__Graph!, interface: String!) repeatable on OBJECT | INTERFACE

directive @join__type(graph: join__Graph!, key: join__FieldSet, extension: Boolean! = false, resolvable: Boolean! = true, isInterfaceObject: Boolean! = false) repeatable on OBJECT | INTERFACE | UNION | ENUM | INPUT_OBJECT | SCALAR

directive @join__unionMember(graph: join__Graph!, member: String!) repeatable on UNION

directive @link(url: String, as: String, for: link__Purpose, import: [link__Import]) repeatable on SCHEMA

scalar join__FieldSet

enum join__Graph {
  UPLOADS @join__graph(name: "uploads", url: "http://127.0.0.1:6000/")
}

scalar link__Import

enum link__Purpose {
  """
  `SECURITY` features provide metadata necessary to securely resolve fields.
  """
  SECURITY

  """
  `EXECUTION` features provide metadata necessary for operation execution.
  """
  EXECUTION
}

type Mutation
  @join__type(graph: UPLOADS)
{
  collectionImageUpload(data: ImageData): Image @join__field(graph: UPLOADS)
}

type Query
  @join__type(graph: UPLOADS)
{
  hello: String @join__field(graph: UPLOADS)
}

scalar Upload
  @join__type(graph: UPLOADS)

input ImageData
  @join__type(graph: UPLOADS)
{
  image: Upload!
  width: Int!
  height: Int!
  fileSizeBytes: Int!
}

type Image
  @join__type(graph: UPLOADS)
{
  url: String
}
```

## Run subgraph

```sh
npm install
npm start
```

## Curl command

```sh
curl -X POST http://127.0.0.1:4000/ \
  -H 'apollo-require-preflight: true' \
  -F 'operations={"operationName":"imageUpload","variables":{"image":null,"height":3706,"width":5089,"fileSizeBytes":280456},"query":"mutation imageUpload($image: Upload!, $width: Int!, $height: Int!, $fileSizeBytes: Int!) {\n  collectionImageUpload(\n    data: {image: $image, width: $width, height: $height, fileSizeBytes: $fileSizeBytes}\n  ) {\n    url\n    __typename\n  }\n}"}' \
  -F 'map={"1":["variables.image"]}' \
  -F '1=@hello.txt;type=text/plain'
```

