# Unknown
- protein_filtering_items

## Parameters

* `dataset`
  * default: DS1740_1 DS1741_1 DS1742_1

## `mk_filter`

```javascript
({dataset})=>{
  var jpost_values = [];
  if(dataset){ 
    jpost_values.push("  VALUES ?dataset { :" + dataset.split(/\s+/).join(" :") + " }"); 
  }
  return {jpost_values: jpost_values.join("\n")}
}
```

## Endpoint

{{SPARQLIST_EP}}

## `go_bp`

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX uniprot: <http://purl.uniprot.org/core/>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT ?up
WHERE {
  {
    SELECT DISTINCT ?up
    WHERE {
      {{mk_filter.jpost_values}}
      ?dataset jpo:hasProtein ?protein .
      ?protein a obo:MS_1002401 ;
               rdfs:seeAlso ?up .
    }
  }
  ?up uniprot:classifiedWith/rdfs:subClassOf* obo:GO_0008150 .
}
LIMIT 1
```

## `go_mf`

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX uniprot: <http://purl.uniprot.org/core/>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT ?up
WHERE {
  {
    SELECT DISTINCT ?up
    WHERE {
      {{mk_filter.jpost_values}}
      ?dataset jpo:hasProtein ?protein .
      ?protein a obo:MS_1002401 ;
               rdfs:seeAlso ?up .
    }
  }
  ?up uniprot:classifiedWith/rdfs:subClassOf* obo:GO_0003674  .
}
LIMIT 1
```

## `go_cc`

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX uniprot: <http://purl.uniprot.org/core/>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT ?up
WHERE {
  {
    SELECT DISTINCT ?up
    WHERE {
      {{mk_filter.jpost_values}}
      ?dataset jpo:hasProtein ?protein .
      ?protein a obo:MS_1002401 ;
               rdfs:seeAlso ?up .
    }
  }
  ?up uniprot:classifiedWith/rdfs:subClassOf* obo:GO_0005575  .
}
LIMIT 1
```

## `mod`

```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX uniprot: <http://purl.uniprot.org/core/>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT ?modification
WHERE {
  {{mk_filter.jpost_values}}
  ?dataset jpo:hasProtein ?protein ;
           jpo:hasProfile/jpo:hasEnzymeAndModification/(jpo:variableModification|jpo:fixedModification) ?mod_node .
  ?mod_node rdf:type ?modification ;
            jpo:modificationClass jpo:JPO_022 .
  ?protein a obo:MS_1002401 ;
           jpo:hasPeptideEvidence/jpo:hasPeptide/jpo:hasPsm/jpo:hasModification/rdf:type ?modification .
}
LIMIT 1
```

## `Output`

```javascript
({go_bp, go_mf, go_cc, mod})=>{
  var bp = false;
  var mf = false;
  var cc = false;
  var md = false;
  if(go_bp.results.bindings[0]) bp = true;
  if(go_mf.results.bindings[0]) mf = true;
  if(go_cc.results.bindings[0]) cc = true;
  if(mod.results.bindings[0]) md = true;
  return {go_bp: bp, go_mf: mf, go_cc: cc, mod: md}
};
```
