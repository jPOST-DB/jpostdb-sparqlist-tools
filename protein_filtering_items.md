# Unknown
- protein_filtering_items

## Parameters

* `dataset`
  * default: DS1740_1 D1741_1 DS1742_1

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
SELECT DISTINCT ?go ?label (COUNT (DISTINCT ?up) AS ?count)
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
  ?up uniprot:classifiedWith ?go .
  ?go rdfs:subClassOf obo:GO_0008150 ;
      rdfs:label ?label .
}
ORDER BY  DESC (?count)
```

## Endpoint

{{SPARQLIST_EP}}

## `go_mf`

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX uniprot: <http://purl.uniprot.org/core/>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT DISTINCT ?go ?label (COUNT (DISTINCT ?up) AS ?count)
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
  ?up uniprot:classifiedWith ?go .
  ?go rdfs:subClassOf obo:GO_0003674  ;
      rdfs:label ?label .
}
ORDER BY  DESC (?count)
```

## Endpoint

{{SPARQLIST_EP}}

## `go_cc`

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX uniprot: <http://purl.uniprot.org/core/>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT DISTINCT ?go ?label (COUNT (DISTINCT ?up) AS ?count)
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
  ?up uniprot:classifiedWith ?go .
  ?go rdfs:subClassOf obo:GO_0005575  ;
      rdfs:label ?label .
}
ORDER BY  DESC (?count)
```

## Endpoint

{{SPARQLIST_EP}}

## `mod`

```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX uniprot: <http://purl.uniprot.org/core/>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT DISTINCT ?modification ?label (COUNT (DISTINCT ?protein) AS ?count)
WHERE {
  {{mk_filter.jpost_values}}
  ?dataset jpo:hasProtein ?protein ;
           jpo:hasProfile/jpo:hasEnzymeAndModification/(jpo:variableModification|jpo:fixedModification) ?mod_node .
  ?mod_node rdf:type ?modification ;
            jpo:modificationClass jpo:JPO_022 .
  ?protein a obo:MS_1002401 ;
           jpo:hasPeptideEvidence/jpo:hasPeptide/jpo:hasPsm/jpo:hasModification/rdf:type ?modification .
  ?modification rdfs:label ?label .
}
ORDER BY DESC (?count)
```

## `Output`

```javascript
({go_bp, go_mf, go_cc, mod})=>{
  return {go_bp: go_bp.results.bindings, go_mf: go_mf.results.bindings, go_cc: go_cc.results.bindings, mod: mod.results.bindings}
};
```
