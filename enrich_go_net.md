# Go term network (for Stanza 'slice_comparison' :enrich #4 GO)

- GO （カテゴリ指定）の親子関係 DAG を取得

## Parameters

* `go`
  * default: biological_process

## Endpoint

{{SPARQLIST_EP}}

## `go_edge`

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
SELECT DISTINCT ?child ?child_label ?parent ?parent_label
FROM <http://jpost.org/graph/ontology> 
WHERE {
  {
    SELECT DISTINCT ?parent
    WHERE {
      ?parent rdfs:subClassOf*/rdfs:label "{{go}}" .
    }
  }
  ?child rdfs:subClassOf ?parent .
  ?child rdfs:label ?child_label .
  ?parent rdfs:label ?parent_label .
  FILTER (DATATYPE (?child_label) = xsd:string)
  FILTER (DATATYPE (?parent_label) = xsd:string)
}
```

## `return`

```javascript
({go_edge})=>{
  var list = go_edge.results.bindings;
  var r = [];
  for(var i = 0; i < list.length; i++){
    var obj = { child: list[i].child.value.replace(/http:\/\/purl\.obolibrary\.org\/obo\/GO_/, ""),
                parent: list[i].parent.value.replace(/http:\/\/purl\.obolibrary\.org\/obo\/GO_/, ""),
                child_label: list[i].child_label.value,
                parent_label: list[i].parent_label.value
    }
    r.push(obj);
  }
  return r;
};
```