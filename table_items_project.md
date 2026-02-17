# table items of ptoject (for Stanza of 'table_project')

* Project page,

## Parameters

* `project`
  * default: JPST001631

## Endpoint

{{SPARQLIST_EP}}

## `table_items`

```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX sio: <http://semanticscience.org/resource/>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT DISTINCT ?project_id ?title ?description ?note ?rpxd
       (COUNT(DISTINCT ?dataset) AS ?dataset_count) 
       (GROUP_CONCAT(distinct ?species_ds ; separator = ", ") AS ?species)
       (GROUP_CONCAT(distinct ?sample_type_ds ; separator = ", ") AS ?sample_type)
       (GROUP_CONCAT(distinct ?cell_line_ds ; separator = ", ") AS ?cell_line)
       (GROUP_CONCAT(distinct ?organ_ds ; separator = ", ") AS ?organ)
       (GROUP_CONCAT(distinct ?disease_class_ds ; separator = ", ") AS ?disease_class)
       (GROUP_CONCAT(distinct ?disease_ds ; separator = ", ") AS ?disease)
       (GROUP_CONCAT(distinct ?fractionation_ds ; separator = ", ") AS ?fractionation)
WHERE {
  VALUES ?project { :{{project}} }
  ?project jpo:hasDataset ?dataset ;
           dct:description ?description ;
           dct:identifier ?project_id ;
           dct:title ?title ;
           rdfs:seeAlso ?rpxd .
  ?dataset jpo:hasProfile/jpo:hasSample ?sample .
  ?sample jpo:species/rdfs:seeAlso*/skos:prefLabel ?species_ds .
  FILTER (LANG(?species_ds) = 'en') 
  FILTER (REGEX(STR(?rpxd), "RPXD"))
  OPTIONAL { 
    ?project rdfs:comment ?note . 
  }
  OPTIONAL { 
    ?sample jpo:sampleType/rdfs:label ?sample_type_ds . 
  }
  OPTIONAL { 
    ?sample jpo:cellLine/rdfs:label ?cell_line_ds . 
  }
  OPTIONAL { 
    ?sample jpo:organ/rdfs:label ?organ_ds . 
  }
  OPTIONAL { 
    ?sample jpo:diseaseClass/rdfs:label ?disease_class_ds . 
    FILTER (LANG(?disease_class_ds) = 'en' OR LANG(?disease_class_ds) = '')
  }
  OPTIONAL { 
    ?sample jpo:disease/rdfs:label ?disease_ds . 
    FILTER (LANG(?disease_ds) = 'en' OR LANG(?disease_ds) = '')
  }
  OPTIONAL {
    ?dataset jpo:hasProfile/jpo:hasFractionation/jpo:hasFractionationType
             [ 
               a jpo:SubcellularFractionation;
               rdfs:label ?fractionation_ds
             ] . 
  }
}
```

## `return`

```javascript
({table_items})=>{
  let data = table_items.results.bindings[0];
  let title_pre = data.title.value;
  let org_project_id = "";
  if (title_pre.match(/JPST\d+$/)) {
    org_project_id = title_pre.match(/(JPST\d+)$/)[1];
    title_pre = title_pre.match(/(.+)JPST\d+$/)[1];
  }
  let cell_line = "";
  let organ = "";
  let disease_class = "";
  let sample_type = "";
  let disease = "";
  let fractionation = "";
  let note = "";
  if(data.cell_line) cell_line = data.cell_line.value;
  if(data.organ) organ = data.organ.value;
  if(data.disease_class) disease_class = data.disease_class.value;	
  if(data.disease) disease = data.disease.value;
  if(data.sample_type) sample_type = data.sample_type.value;
  if(data.fractionation) fractionation = data.fractionation.value; 
  if(data.note) note = data.note.value;
  let disease_tmp = disease_class;
  if(disease && disease_class == "Other disease") disease_tmp = disease;
  let disease_label = disease_tmp.charAt(0).toUpperCase() + disease_tmp.slice(1);
  return {
    species: data.species.value,
    sample_type: sample_type,
    dataset_count: data.dataset_count.value.toString().replace(/(\d)(?=(\d{3})+$)/g , '$1,'),
    description: data.description.value,
    project_id: data.project_id.value,
    title_pre: title_pre,
    org_project_id: org_project_id,
    cell_line: cell_line,
    organ: organ,
    disease_class: disease_class,
    disease: disease,
    disease_label: disease_label,
    fractionation: fractionation,
    note: note,
    rpxd: data.rpxd.value.match(/(RPXD\d+)/)[1]
  };
};
```
