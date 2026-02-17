# protein with go term (for Stanza 'go_count')

## Parameters

* `dataset` (Req.)
  * default: DS81_1
* `level` (Req.)
  * default: 2
  * example: 1: protein, 2: leading protein, 3: protein with uniquq-pep
* `category`
  * default: biological_process
  * example: biological_process, molecular_function, cellular_component
* `term`
  * default: cellular process


## `filter`

```javascript
({dataset, level, category, term}) => {
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

## `get_list`

```sparql
#DEFINE sql:select-option "order"
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX uniprot: <http://purl.uniprot.org/core/>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT DISTINCT ?upid ?name ?symbol (COUNT (DISTINCT ?pep) AS ?pep_count) (COUNT (?psm) AS ?psm_count)
WHERE {
  VALUES ?dataset { {{filter.values}} }
  ?dataset jpo:hasProtein ?protein .
  ?protein a {{filter.type}} ;
           {{filter.code}}
            rdfs:label ?upid ;
           jpo:hasDatabaseSequence ?up .
  ?protein  jpo:hasPeptideEvidence/jpo:hasPeptide ?pep .
  ?pep jpo:hasPsm ?psm .
  ?up  uniprot:mnemonic ?symbol .
  OPTIONAL { ?up (uniprot:recommendedName|uniprot:submittedName)/uniprot:fullName ?name . }
  ?up uniprot:classifiedWith ?go .
  FILTER (REGEX (?go, obo:GO_))
  ?go rdfs:subClassOf* ?level .
  ?level rdfs:subClassOf obo:{{filter.go}} .
  ?level rdfs:label "{{term}}" . #^^xsd:string .
}
ORDER BY DESC (?pep_count) ?name
```

## `return`

```javascript
({term, get_list}) =>{
  var list = get_list.results.bindings;
  var head = ["Protein Name", "Accession", "ID", "# Peptides", "# PSMs"];
  var arg = ["name", "uniprot", "symbol", "pep_count", "psm_count"];
  var align = [ 0, 0, 0, 1, 1];
  var data = [];
  for(var i = 0; i < list.length; i++){
    var name = "";
    if(list[i].name) name = list[i].name.value;
    var obj = {
      name: name,
      _alink_name: "./protein.php?id=" + list[i].upid.value,
//      _alink_name: "javascript:jPost.openProtein('" + list[i].upid.value + "')",
      symbol: list[i].symbol.value,
      uniprot: list[i].upid.value,
      _alink_uniprot: "http://www.uniprot.org/uniprot/" + list[i].upid.value,
      pep_count: list[i].pep_count.value - 0,
      psm_count: list[i].psm_count.value
    };
    data.push(obj);
  }
 // data = data.sort(function(a,b){
 //   if(a.sort>b.sort) return 1;
 //   if(a.sort<b.sort) return -1;
//    if(a.pep_count>b.pep_count) return -1;
//    if(a.pep_count<b.pep_count) return 1;
//    return 0;
//    });
  return {title: term, head: head, arg: arg, align: align, data: data};
};
```
