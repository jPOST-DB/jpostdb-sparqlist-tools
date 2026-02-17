# UniProt list to GO (for Stanza 'slice_comparison' :enrich #3 GO)

- 入力タンパク質の GO (カテゴリ指定) を取得

## Parameters

* `up` UniProt IDs (Req.)
  * default: P02751 P24821 Q14764 Q9NZM1 P12111 Q6YHK3 P08473 P12110 Q15582 Q9Y6N5
* `go` GO term (Req.)
  * default: biological_process
  
## Endpoint

{{SPARQLIST_EP}}


## `filter`

```javascript
({up})=>{
  return {values: "up:" + decodeURIComponent(up).split(/[ ,]/).join(" up:") };
};
```

## `get_go`

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX up: <http://purl.uniprot.org/uniprot/>
PREFIX uniprot: <http://purl.uniprot.org/core/>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
SELECT DISTINCT ?up ?go ?go_label
WHERE {
VALUES ?up { {{filter.values}} }
  ?up uniprot:reviewed 1 ;
       uniprot:classifiedWith/rdfs:subClassOf* ?go .
  FILTER (REGEX (STR (?go), "obo/GO_"))
  ?go rdfs:label ?go_label ;
      <http://www.geneontology.org/formats/oboInOwl#hasOBONamespace> "{{go}}" .
  FILTER( DATATYPE(?go_label) = xsd:string)
}
```

## `return`

```javascript
({get_go})=>{
  var list = get_go.results.bindings;
  var r = [];
  for(var i = 0; i < list.length; i++){
    var obj = { up: list[i].up.value.replace(/http:\/\/purl\.uniprot\.org\/uniprot\//, ""),
                go: list[i].go.value.replace(/http:\/\/purl\.obolibrary\.org\/obo\/GO_/, ""),
                name: list[i].go_label.value,
              };
    r.push(obj);
  }
  return { list: r}
};
```