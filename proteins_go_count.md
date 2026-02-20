# proteins GO count (for Stanza 'go_count')

## Parameters

* `dataset` (Req.)
  * default: DS1634_2
* `level` (Req.)
  * default: 2
  * example: 1: protein, 2: leading protein, 3: protein with uniquq-pep
* `category`
  * default: biological_process
  * example: biological_process, molecular_function, cellular_component


## `filter`

```javascript
({dataset, level, category}) => {
  var array = decodeURIComponent(dataset).split(/[ ,]/);
  var type = "obo:MS_1002401";
  var code = "";
  var go = "GO_0008150";
  if(level == 1){
  	type = "jpo:Protein";
  }else if(level == 3){
    code = "jpo:hasPeptideEvidence/jpo:hasPeptide/a jpo:UniquePeptideAtMsLevel ;"
  }
  if(category == "biological_process"){
    go = "GO_0008150";
  }else if(category == "molecular_function"){
    go = "GO_0003674";
  }else if(category == "cellular_component"){
    go = "GO_0005575";
  }
  return  { values: ":" + array.join(" :"), type: type, code: code, go: go};
};
```

## Endpoint

{{SPARQLIST_EP}}

## `get_go`

```sparql
#DEFINE sql:select-option "order"
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX uniprot: <http://purl.uniprot.org/core/>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX jpost: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT DISTINCT ?label (COUNT (DISTINCT ?up) AS ?count)
WHERE {
  VALUES ?dataset { {{filter.values}} }
  ?dataset jpost:hasProtein ?protein .
  ?protein a {{filter.type}} ;
           {{filter.code}}
           rdfs:seeAlso ?up .
  ?up uniprot:classifiedWith ?go .
  FILTER (REGEX (?go, obo:GO_))
  ?go rdfs:subClassOf* ?level .
  ?level rdfs:subClassOf obo:{{filter.go}} .
  ?level rdfs:label ?label .
  #FILTER (DATATYPE(?label) = xsd:string)
  # FILTER (LANG (?label) = "en")
}
ORDER BY DESC(?count)
```

## `return`

```javascript
({get_go}) => {
  var data = get_go.results.bindings;
  r = [];
  for(var i = 0; i < data.length; i++){
    r.push({label: data[i].label.value, id: data[i].label.value, count: data[i].count.value}); 
  }
  return r
};
```

