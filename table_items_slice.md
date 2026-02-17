# table items of slice (for Stanza of 'table_slice)

* Slice page,

## Parameters

* `dataset`
  * default: DS1631_1 DS1631_2 DS1631_3 DS1632_1 DS1632_2 DS1632_3


## `filter`

```javascript
({dataset}) => {
  var value = "";
  if(dataset){
    value += "  VALUES ?dataset { :" + decodeURIComponent(dataset).split(/[ ,]/).join(" :") + " }";
  }
  return { value: value };
};
```

## Endpoint

{{SPARQLIST_EP}}

## `id`
```sparql
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT DISTINCT ?label
WHERE {
{{filter.value}}
  ?dataset dct:identifier ?label .
  ?project jpo:hasDataset ?dataset .
} 
ORDER BY ?project ?label
```

## `species`
```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT DISTINCT ?label (COUNT(?label) AS ?count)
WHERE {
{{filter.value}}
  ?dataset  jpo:hasProfile/jpo:hasSample/jpo:species/rdfs:seeAlso*/skos:prefLabel ?label .
}
ORDER BY DESC (?count)
```

## `disease`
```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT DISTINCT ?label (COUNT(?label) AS ?count)
WHERE {
{{filter.value}}
  ?dataset  jpo:hasProfile/jpo:hasSample/jpo:diseaseClass/rdfs:label ?label .
}
ORDER BY DESC (?count)
```

## `organ`
```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT DISTINCT ?label (COUNT(?label) AS ?count)
WHERE {
{{filter.value}}
  ?dataset  jpo:hasProfile/jpo:hasSample/jpo:organ/rdfs:label ?label .
}
ORDER BY DESC (?count)
```

## `protein`
```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT (COUNT(DISTINCT ?up)AS ?count)
WHERE {
{{filter.value}}
  ?dataset  jpo:hasProtein ?prt .
  ?prt a obo:MS_1002401 ;
       jpo:hasDatabaseSequence ?up .
}
```

## `peptide`
```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT (COUNT(DISTINCT ?seq)AS ?count)
WHERE {
{{filter.value}}
  ?dataset  jpo:hasPeptide/jpo:hasSequence [ a obo:MS_1001344 ;
                                           rdf:value ?seq ] .
}
```

## `uniq_pep`
```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT (COUNT(DISTINCT ?seq)AS ?count)
WHERE {
{{filter.value}}
  ?dataset  jpo:hasPeptide ?pep .
  ?pep a jpo:UniquePeptideAtMsLevel ;
       jpo:hasSequence [ a obo:MS_1001344 ;
                                           rdf:value ?seq ] .
}
```

## `return`

```javascript
({id, species, organ, disease, protein, peptide, uniq_pep})=>{
  var obj = {
    dataset_count: id.results.bindings.length.toString().replace(/(\d)(?=(\d{3})+$)/g , '$1,'),
    protein_count: protein.results.bindings[0].count.value.replace(/(\d)(?=(\d{3})+$)/g , '$1,'),
    peptide_count: peptide.results.bindings[0].count.value.replace(/(\d)(?=(\d{3})+$)/g , '$1,'),
    uniq_pep_count: uniq_pep.results.bindings[0].count.value.replace(/(\d)(?=(\d{3})+$)/g , '$1,')
  };
  var array = [];
  var list = id.results.bindings;
  for(var i = 0; i < list.length; i++){ array.push( list[i].label.value ) }
  obj.id = array.join(", ");
  array = [];
  list = species.results.bindings;
  for(var i = 0; i < list.length; i++){ array.push( list[i].label.value ) }
  obj.species = array.join(", ");
  array = [];
  list = organ.results.bindings;
  for(var i = 0; i < list.length; i++){ array.push( list[i].label.value ) }
  obj.organ = array.join(", ");
  array = [];
  list = disease.results.bindings;
  for(var i = 0; i < list.length; i++){ array.push( list[i].label.value.charAt(0).toUpperCase() + list[i].label.value.slice(1) ) }
  obj.disease = array.join(", ");
  return obj;
};
```