# status chart of database with filtering(for Stanza stat_pie_chart)

* Dataset の内訳 pie chart 用

## Parameters

* `type`
  * default: species
* `species` (Opt.)
  * example: TAX_9606
* `species_s` (Opt.)
  * example: vertebrates,plants
* `sample_type` (Opt.)
  * example: C16403
* `cell_line` (Opt.)
* `organ` (Opt.)
* `disease` (Opt.)
* `disease_s` (Opt.)
  * example: cancer
* `modification` (Opt.)
  * example: UNIMOD_21
* `instrument` (Opt.)

## `search_filter`

```javascript
async ({species, species_s, sample_type, cell_line, organ, disease, disease_s, modification, instrument}) => {
  var sparqlet = (api, body) => {
    const options = {
      method: 'POST',
      body: body,
      headers: {
        'Accept': 'application/json',
        'Content-Type': 'application/x-www-form-urlencoded'
      }
    };
    var res = fetch(api, options).then(res=>res.json());
    return res;
  }
  var params = [];
  params.push("mode=dataset");		
  if(species) params.push("species=" + species);
  if(species_s) params.push("species_s=" + species_s);
  if(sample_type) params.push("sample_type=" + sample_type);
  if(cell_line) params.push("cell_line=" + cell_line );
  if(organ) params.push("organ=" + organ );
  if(disease) params.push("disease=" + disease );
  if(disease_s) params.push("disease_s=" + disease_s );
  if(modification) params.push("modification=" + modification );
  if(instrument) params.push("instrument=" + instrument );

  var res = await sparqlet("https://db-dev.jpostdb.org/rest/api/dbi_make_filter_code", params.join("&"));
  
  return res;
};
```

## `type_filter`

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
      replace = "(REPLACE (STR(?ontology), id_tax:, 'TAX_') AS ?id)";
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
PREFIX id_tax: <http://identifiers.org/taxonomy/>
PREFIX tax: <http://purl.bioontology.org/ontology/NCBITAXON/>
PREFIX unimod: <http://www.unimod.org/obo/unimod.obo#>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT DISTINCT ?label (COUNT (DISTINCT ?dataset) AS ?count) {{type_filter.replace}}
WHERE {
  {{search_filter.code_value}}
  ?dataset a jpo:Dataset ;
           jpo:hasProfile ?profile .
  ?profile {{type_filter.ontology}}
  ?ontology {{type_filter.label}} ?label_tmp .
  FILTER (LANG(?label_tmp) = 'en' OR LANG(?label_tmp) = '') 
  BIND(STR(?label_tmp) AS ?label) # delete lang
  {{type_filter.filter}}
  {{search_filter.code_dataset}}
}
ORDER BY DESC (?count)
```

## `json`

```javascript
({type, get_count}) => {
    var list = get_count.results.bindings;
    var data = [];
    for(var i = 0; i < list.length; i++){
      data.push({
        label: list[i].label.value.charAt(0).toUpperCase() + list[i].label.value.slice(1),
        count: list[i].count.value,
        onclick_list: [ {
          type: type,
          label: list[i].label.value,
          id: list[i].id.value
        } ]
      });
    }
    return {type: type.replace(/_/, " "), unit: "datasets", data: data};
}
```

## Output

```javascript
({
  json({json}) {
    return json;
  },
  html: hbs(`
   <script src="https://cdn.jsdelivr.net/npm/@webcomponents/webcomponentsjs@1.3.0/webcomponents-loader.js" crossorigin=""></script>
   <link rel="import" href="https://db-dev.jpostdb.org/ts/stanza/stat_pie_chart/">
   <togostanza-stat_pie_chart type='{{type}}' species='{{species}}' species_s='{{species_s}}' sample_type='{{sample_type}}' cell_line='{{cell_line}}' organ='{{organ}}' disease='{{disease}}' disease_s='{{disease_s}}' modification='{{modification}}' instrument='{{instrument}}'></togostanza-stat_pie_chart>
  `)
})
```
