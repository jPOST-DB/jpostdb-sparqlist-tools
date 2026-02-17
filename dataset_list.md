# dataset list (for タンパク質推定機)

## Endpoint

{{SPARQLIST_EP}}

## `dataset_items`

```sparql
PREFIX jpost: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX dct: <http://purl.org/dc/terms/>
SELECT DISTINCT ?dataset_id
WHERE {
  [] a jpost:Dataset ;
     dct:identifier ?dataset_id .
}
ORDER BY ?dataset_id
```