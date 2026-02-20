# table items of dataset (for Stanza of 'table_dataset')

* Dataset page, key-value table sranza

## Parameters

* `dataset`
  * default: DS1631_1


## Endpoint

{{SPARQLIST_EP}}

## `table_items`

```sparql
#DEFINE sql:select-option "order"
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX sio: <http://semanticscience.org/resource/>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT DISTINCT ?id ?spc_count ?pep_count ?prt_count ?species ?sample_type ?title ?description ?project_id ?cell_line ?organ ?disease_class ?disease (COUNT(?ds) AS ?dataset_count) ?fractionation 
WHERE {
  VALUES ?dataset { :{{dataset}} }
#  ?dataset sio:SIO_000216 [ a jpo:NumOfSpectra ;   # sparql-proxy だと、前で繋げると何故か遅くなるので後ろで
#                          sio:SIO_000300 ?spc_count ] ;
#           sio:SIO_000216 [ a jpo:NumOfPeptides ;
#                          sio:SIO_000300 ?pep_count ] ;
#           sio:SIO_000216 [ a jpo:NumOfLeadingProteins ;
#                          sio:SIO_000300 ?prt_count ] .
  ?dataset dct:identifier ?id ;
           jpo:hasProfile/jpo:hasSample ?sample .
  ?sample jpo:species/rdfs:seeAlso*/skos:prefLabel ?species .
  FILTER (LANG(?species) = 'en') 
  ?project jpo:hasDataset ?dataset ;
           dct:description ?description ;
           dct:identifier ?project_id ;
           dct:title ?title .
  OPTIONAL { ?sample jpo:sampleType/rdfs:label ?sample_type . }
  OPTIONAL { ?sample jpo:cellLine/rdfs:label ?cell_line . }
  OPTIONAL { ?sample jpo:organ/rdfs:label ?organ . }
  OPTIONAL { ?sample jpo:diseaseClass/rdfs:label ?disease_class . 
           FILTER (LANG(?disease_class) = 'en' OR LANG(?disease_class) = '') }
  OPTIONAL { ?sample jpo:disease/rdfs:label ?disease . 
           FILTER (LANG(?disease) = 'en' OR LANG(?disease) = '') }
  OPTIONAL { ?dataset jpo:hasProfile/jpo:hasFractionation/jpo:hasFractionationType [ a jpo:SubcellularFractionation;
                      rdfs:label ?fractionation ] . }       
  ?dataset sio:SIO_000216 [ a jpo:NumOfSpectra ;
                          sio:SIO_000300 ?spc_count ] ;
           sio:SIO_000216 [ a jpo:NumOfPeptides ;
                          sio:SIO_000300 ?pep_count ] ;
           sio:SIO_000216 [ a jpo:NumOfLeadingProteins ;
                          sio:SIO_000300 ?prt_count ] .
  ?project jpo:hasDataset ?ds .
}
```

## Endpoint

{{SPARQLIST_EP}}

## `seealso`

```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT DISTINCT ?seealso
WHERE {
  VALUES ?dataset { :{{dataset}} }
  ?project jpo:hasDataset ?dataset ;
           rdfs:seeAlso ?seealso .
}
```

## Endpoint

{{SPARQLIST_EP}}

## `plex`

```sparql
#DEFINE sql:select-option "order"
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX sio: <http://semanticscience.org/resource/>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT DISTINCT ?plex_label ?plex_description
WHERE {
  VALUES ?dataset { :{{dataset}} }
  ?dataset jpo:hasProfile/jpo:hasPlex ?plex .
            ?plex rdfs:label ?plex_label ;
                  dct:description ?plex_description . 
}
ORDER BY ?plex_label
```

## `return`

```javascript
({table_items, seealso, plex})=>{
  var list = plex.results.bindings;
  var plex_list = [];
  for(var i = 0; i < list.length; i++){
     list[i].plex_description.value = list[i].plex_description.value.replace(" with with ", " with "); // for miss of metadata
    plex_list.push({label: list[i].plex_label.value, description: list[i].plex_description.value});
  }
  var data = table_items.results.bindings[0];
  var cell_line = "";
  var organ = "";
  var disease_class = "";
  var sample_type = "";
  var disease = "";
  var fractionation = "";
  if(data.cell_line) cell_line = data.cell_line.value;
  if(data.organ) organ = data.organ.value;
  if(data.disease_class) disease_class = data.disease_class.value;	
  if(data.disease) disease = data.disease.value;
  if(data.sample_type) sample_type = data.sample_type.value;
  if(data.fractionation) fractionation = data.fractionation.value; 
  var disease_tmp = disease_class;
  if(disease && disease_class == "Other disease") disease_tmp = disease;
  var disease_label = disease_tmp.charAt(0).toUpperCase() + disease_tmp.slice(1);
  var obj =  {
    id: data.id.value,
    species: data.species.value,
    sample_type: sample_type,
    spc_count: data.spc_count.value.replace(/(\d)(?=(\d{3})+$)/g , '$1,'),
    pep_count: data.pep_count.value.replace(/(\d)(?=(\d{3})+$)/g , '$1,'),
    prt_count: data.prt_count.value.replace(/(\d)(?=(\d{3})+$)/g , '$1,'),
    dataset_count: data.dataset_count.value.toString().replace(/(\d)(?=(\d{3})+$)/g , '$1,'),
    description: data.description.value,
    project_id: data.project_id.value,
    title: data.title.value,
    cell_line: cell_line,
    organ: organ,
    disease_class: disease_class,
    disease: disease,
    disease_label: disease_label,
    plex: plex_list,
    fractionation: fractionation
  };
  list = seealso.results.bindings;
  for(var i in list){
    var uri = list[i].seealso.value;
    if(uri.match(/JPST\d{6}/)) obj.orig_jpst = uri.match(/(JPST\d{6})/)[1];
    else if(uri.match(/RPXD\d+/)) obj.rpxd = uri.match(/(RPXD\d+)/)[1];
    else if(uri.match(/PXD\s+/)) obj.orig_pxd = uri.match(/(PXD\d+)/)[1];
  }
  return obj;
};
```
