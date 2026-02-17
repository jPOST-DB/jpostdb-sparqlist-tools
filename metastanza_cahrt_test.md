# Develop
- metastanza chart data - single value

## Parameters

* `type`
  * default:
  * example: species, disease, sample_type, cell_line, modification, instrument

## `filter`

```javascript
({type})=>{
  type = type.toLowerCase();
  if(type == "sample_type") type = "sampleType";
  else if(type == "cell_line") type = "cellLine";
  else if(type == "disease") type = "diseaseClass";	
  var replace = "(REPLACE (STR(?ontology), obo:, '') AS ?id)";
  var ontology = "";
  var label = "rdfs:label";
  var filter = "";
  console.log(type);
  if(type == "species" || type == "sampleType" || type == "cellLine" || type == "organ" || type == "diseaseClass"){
    ontology = "jpo:hasSample/jpo:" + type + " ?ontology .";
    if(type == "species"){
      label = "rdfs:seeAlso/skos:prefLabel";
      replace = "(REPLACE (STR(?ontology), tax:, 'TAX_') AS ?id)";
    }else if(type == "sampleType" || type == "organ"){ 
      replace = "(REPLACE (STR(?ontology), ncit:, '') AS ?id)";
    }else if(type == "cellLine"){
      filter = "FILTER (REGEX (?ontology, 'obolibrary'))";
    }else if(type == "diseaseClass"){
      replace = "(REPLACE (REPLACE (STR(?ontology), obo:, ''), jpo:, '') AS ?id)";
    }
  }else if(type == "modification"){
     ontology = "jpo:hasEnzymeAndModification/(jpo:variableModification|jpo:fixedModification) [ a ?ontology ;\n     jpo:modificationClass jpo:JPO_022 ] .";
     replace = "(REPLACE (STR(?ontology), unimod:, '') AS ?id)";
   }else if(type == "instrument"){
     ontology = "jpo:hasMsMode/jpo:instrument ?ontology .";
   }
   return {replace: replace, ontology: ontology, label: label, filter: filter}
};
```

## Endpoint

{{SPARQLIST_EP}}

## `get_count`

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX ncit: <http://ncicb.nci.nih.gov/xml/owl/EVS/Thesaurus.owl#>
PREFIX tax: <http://identifiers.org/taxonomy/>
PREFIX unimod: <http://www.unimod.org/obo/unimod.obo#>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT ?label (COUNT (DISTINCT ?dataset) AS ?count) {{filter.replace}}
WHERE {
  ?dataset a jpo:Dataset ;
           jpo:hasProfile ?profile .
  ?profile {{filter.ontology}}
  ?ontology {{filter.label}} ?label .
  {{filter.filter}}
}
ORDER BY DESC (?count)
```
