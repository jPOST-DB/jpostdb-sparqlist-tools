# Develop
- metastanza chart data - multi value

## Parameters

* `type`
  * default: instrument
  * example: disease, sample_type, modification, instrument

## `filter`

```javascript
({type})=>{
  type = type.toLowerCase();
  if(type == "sample_type") type = "sampleType";
  else if(type == "disease") type = "diseaseClass";	
  var ontology = "";
  var label = "rdfs:label";
  var filter = "";

  if(type == "sampleType" || type == "cellLine" || type == "organ" || type == "diseaseClass"){
    ontology = "jpo:hasSample/jpo:" + type + " ?ontology .";
  }else if(type == "modification"){
     ontology = "jpo:hasEnzymeAndModification/(jpo:variableModification|jpo:fixedModification) [ a ?ontology ;\n     jpo:modificationClass jpo:JPO_022 ] .";
   }else if(type == "instrument"){
     ontology = "jpo:hasMsMode/jpo:instrument ?ontology .";
   }
   return {ontology: ontology, label: label, filter: filter}
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
SELECT ?label ?item (COUNT (DISTINCT ?dataset) AS ?value)
WHERE {
  ?dataset a jpo:Dataset ;
           jpo:hasProfile ?profile .
  ?profile jpo:hasSample/jpo:species/rdfs:seeAlso/skos:prefLabel ?label .
  ?profile {{filter.ontology}}
  ?ontology {{filter.label}} ?item .
  {{filter.filter}}
}
ORDER BY ?label ?category
```

## `format`

```javascript
({get_count}) => {
  const list = get_count.results.bindings;
  let labels = {};
  let items = {};
  let data = {};
  for(let el of list){
    if(!labels[el.label.value]) labels[el.label.value] = 0;
    if(!items[el.item.value]) items[el.item.value] = 0;
    if(!data[el.label.value]) data[el.label.value] = {};
    labels[el.label.value] += parseInt(el.value.value);
    items[el.item.value] += parseInt(el.value.value);
    data[el.label.value][el.item.value] = parseInt(el.value.value); 
  };
  let sort_labels = Object.keys(labels).sort( (a,b) => {
    return labels[b] - labels[a];
  });
  let sort_items = Object.keys(items).sort( (a,b) => {
    return items[b] - items[a];
  });

  let format_data = [];
  for(let label of sort_labels){
    let obj = {label: label};
    for(let item of sort_items){
      let value = 0;	
      if(data[label][item]) value = data[label][item];
      obj[item] = value;
    }
    format_data.push(obj);
  }
  return {series: sort_items, data: format_data};
};
```