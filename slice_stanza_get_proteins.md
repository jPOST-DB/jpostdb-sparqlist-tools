# Develop: get protein names for slice stanza (for Stanza 'slice_comparison' :quant_test)

## Parameters


* `dataset1` (Req.)
  * example: DS1631_1 DS81632_1 DS1633_1 DS1634_1
* `dataset2` (Req.)
  * example: DS1637_1 DS1638_1 DS1639_1

## `set_values`

```javascript
({dataset1, dataset2}) => {
  var dataset = ":" + decodeURIComponent(dataset1 + " " + dataset2).split(/[ ,]/).join(" :");
  return { dataset: dataset };
};
```

## Endpoint

{{SPARQLIST_EP}}

## `spectral_counting`

```sparql
DEFINE sql:select-option "order"
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX uniprot: <http://purl.uniprot.org/core/>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX : <http://rdf.jpostdb.org/entry/>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT DISTINCT ?accs ?name ?id ?gene
WHERE {
  VALUES ?dataset { {{set_values.dataset}} }
  ?dataset jpo:hasProtein ?protein .
  ?protein rdfs:label ?accs ;
           a jpo:Protein ;
           jpo:hasDatabaseSequence ?up .
  ?up  uniprot:mnemonic ?id ;
         uniprot:encodedBy/skos:prefLabel ?gene .
  OPTIONAL { ?up (uniprot:recommendedName|uniprot:submittedName)/uniprot:fullName ?name . }          
}
```