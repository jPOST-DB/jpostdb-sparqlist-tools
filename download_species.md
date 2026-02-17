# species list (for UniProt -> jPOST link)

- UniProt 側での Link 作成のため API で使用
  - taxonomy リスト

## Endpoint

{{SPARQLIST_EP}}

## `get_species`

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT DISTINCT ?species (REPLACE (STR(?tax), "http://identifiers.org/taxonomy/", "") AS ?tax_id)
WHERE {
  ?dataset a jpo:Dataset ;
           jpo:hasProfile ?profile .
  ?profile jpo:hasSample/jpo:species ?tax .
  ?tax rdfs:seeAlso/skos:prefLabel ?species .
}
```

## `return`

```javascript
({
  json({get_species}){
    return get_species;
  },
  text({get_species}){
    var list = get_species.results.bindings;
    var text = "# species\ttax_id\n";
    for(var i = 0; i < list.length; i++){
       text += list[i].species.value + "\t" + list[i].tax_id.value + "\n";
    }
   return text;
  }
});
```