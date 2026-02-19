# Discontinued
- protein_filtering

## Parameters

* `dataset`
  * default: DS1725_1 DS1726_1 DS1727_1
* `go_bp`
  * default: GO_0009987 GO_0008283
* `go_mf`
  * default: 
* `go_cc`
  * default: GO_0044421
* `modification`
  * default: UNIMOD_21
* `keyword`
  * default: GTP
* `order`
  * default: mnemonic
* `limit`
  * default: 10
* `offset`
  * default: 0
* `line_count`
  * default: 0


## `mk_filter`

```javascript
({dataset, go_bp, go_mf, go_cc, modification, keyword, order, limit, offset, line_count})=>{
  var jpost_values = [];
  var values = [];
  var go_code = [];
  var mod_code = [];
  var key_code = "";
  var select_code = "DISTINCT ?up ?mnemonic ?protein_name ?gene_name";
  var limit_code = "ORDER BY ?" + order + "\nLIMIT " + limit + "\nOFFSET " + offset;
  if(dataset){ 
    jpost_values.push("  VALUES ?dataset { :" + dataset.split(/\s+/).join(" :") + " }"); 
  }
  if(go_bp){ 
    values.push("  VALUES ?go_bp { obo:" + go_bp.split(/\s+/).join(" obo:") + " }");
    go_code.push("  ?up uniprot:classifiedWith/rdfs:subClassOf ?go_bp .");
  }
  if(go_mf){
    values.push("  VALUES ?go_mf { obo:" + go_mf.split(/\s+/).join(" obo:") + " }");
   	go_code.push("  ?up uniprot:classifiedWith/rdfs:subClassOf ?go_mf .");
  }
  if(go_cc){
    values.push("  VALUES ?go_cc { obo:" + go_cc.split(/\s+/).join(" obo:") + " }");
  	go_code.push("  ?up uniprot:classifiedWith/rdfs:subClassOf ?go_cc .");
  }
  if(modification){
    jpost_values.push("  VALUES ?modification { unimod:" + modification.split(/\s+/).join(" unimod:") + " }");
  	mod_code.push("  ?protein jpo:hasPeptideEvidence/jpo:hasPeptide/jpo:hasPsm/jpo:hasModification/rdf:type ?modification .");
  }
  if(keyword){
    var array = decodeURIComponent(keyword).split(" ");
    key_code += "  ?up  uniprot:mnemonic|(uniprot:encodedBy/skos:prefLabel)|((uniprot:recommendedName/uniprot:fullName)|(uniprot:submittedName/uniprot:fullName)) ?protein_search_space .\n"; 
    key_code += "  FILTER( true &&\n";
    key_code += "    (REGEX(?protein_search_space, '" + array.join("', 'i' )\n    || REGEX(?protein_search_space, '") + "', 'i' ))\n  )\n";
  }
  if(line_count == 1){
    select_code = "(COUNT (DISTINCT ?up) AS ?count)";
    limit_code = "";
  }
  return {jpost_values: jpost_values.join("\n"), values: values.join("\n"), go_code: go_code.join("\n"), mod_code: mod_code.join("\n"), key_code: key_code, select_code: select_code, limit_code: limit_code}
}
```

## Endpoint

{{SPARQLIST_EP}}

## Output

```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX unimod: <http://www.unimod.org/obo/unimod.obo#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX uniprot: <http://purl.uniprot.org/core/>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT {{mk_filter.select_code}}
WHERE {
  {
    SELECT DISTINCT ?up
    WHERE {
      {{mk_filter.jpost_values}}
      ?dataset jpo:hasProtein ?protein .
      ?protein a obo:MS_1002401 ;
               rdfs:seeAlso ?up .
      {{mk_filter.mod_code}}
      }
    }
  {{mk_filter.values}}
  {{mk_filter.go_code}}
  {{mk_filter.key_code}}
  ?up uniprot:mnemonic ?mnemonic ;
      uniprot:encodedBy/skos:prefLabel ?gene_name ;
      (uniprot:recommendedName/uniprot:fullName)|(uniprot:submittedName/uniprot:fullName) ?protein_name .
}
{{mk_filter.limit_code}}
```
