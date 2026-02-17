# GO term count in a taxonomy (for Stanza 'slice_comparison' :enrich #2 GO)

- 指定した種における GO (カテゴリ指定)毎のタンパク質数を取得

## Parameters

* `tax`
  * default: 9606
* `go`
  * default: biological_process


## Endpoint

{{SPARQLIST_EP}}

## `get_go_count`

```sparql
PREFIX uniprot: <http://purl.uniprot.org/core/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
SELECT ?go (COUNT (DISTINCT ?up) AS ?count)
FROM <http://jpost.org/graph/uniprot>
FROM <http://jpost.org/graph/ontology> 
WHERE {
  ?up uniprot:organism <http://purl.uniprot.org/taxonomy/{{tax}}> ;
      uniprot:reviewed 1 ;
      uniprot:classifiedWith/rdfs:subClassOf* ?go .
  FILTER (REGEX (STR (?go), "obo/GO_"))
  ?go <http://www.geneontology.org/formats/oboInOwl#hasOBONamespace> "{{go}}" . 
}
```

## `total`

```sparql
PREFIX uniprot: <http://purl.uniprot.org/core/>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
SELECT (COUNT (DISTINCT ?up) AS ?count)
FROM <http://jpost.org/graph/uniprot>
FROM <http://jpost.org/graph/ontology> 
WHERE {
  ?up uniprot:organism <http://purl.uniprot.org/taxonomy/{{tax}}> ;
      uniprot:classifiedWith ?go ;
      uniprot:reviewed 1 .
  FILTER (REGEX (STR (?go), "obo/GO_"))
  ?go <http://www.geneontology.org/formats/oboInOwl#hasOBONamespace> "{{go}}" . 
}
```

## `return`

```javascript
({get_go_count, total})=>{
  var list = get_go_count.results.bindings;
  var r = [];
  for(var i = 0; i < list.length; i++){
    if(list[i].count.value - 0 > 1){
      var obj = { go: list[i].go.value.replace(/http:\/\/pur\l.obolibrary\.org\/obo\/GO_/, ""),
                 count: list[i].count.value
                };
      r.push(obj);
    }
  }
  return { db_total: total.results.bindings[0].count.value, go: r}
};
```
