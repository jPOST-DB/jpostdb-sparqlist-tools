# project table (for DB interface) req. dbi_make_filter_code

## Parameters

* `datasets` (Opt.)
  * example: DS1631_1,DS1631_2,DS1631_3
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
* `instrument_mode` (Opt.)
  * example: JPO_006
* `instrument` (Opt.)
* `project_keywords` (Opt.)
  * example: CH2O,by trypsin
* `dataset_keywords` (not work)
  * example: CH2O,by trypsin
* `protein_keywords` (not work)
* `excluded_datasets` (Opt.)
  * example: DS1631_1,DS1631_2
* `excluded_proteins` (not work)
* `order` (Opt.)
  * default: project_id
* `desc` (Opt.)
  * example: 1
* `limit` (Opt.)
  * exmaple: 10
* `offset` (Opt.)
  * example: 0
* `line_count`
  * example 1

## `filter`

```javascript
async ({datasets, species, species_s, sample_type, cell_line, organ, disease, disease_s, modification, 
       instrument_mode, instrument, project_keywords, dataset_keywords, protein_keywords, excluded_datasets, excluded_proteins, 
       order, desc, limit, offset, line_count}) => {
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
  params.push("mode=project");		
  if(datasets) params.push("datasets=" + datasets);
  if(species) params.push("species=" + species);
  if(species_s) params.push("species_s=" + species_s);
  if(sample_type) params.push("sample_type=" + sample_type);
  if(cell_line) params.push("cell_line=" + cell_line );
  if(organ) params.push("organ=" + organ );
  if(disease) params.push("disease=" + disease );
  if(disease_s) params.push("disease_s=" + disease_s );
  if(modification) params.push("modification=" + modification );
  if(instrument) params.push("instrument=" + instrument );
  if(instrument_mode) params.push("instrument_mode=" + instrument_mode );
  if(project_keywords) params.push("dataset_keywords=" + project_keywords );
  if(dataset_keywords) params.push("dataset_keywords=" + dataset_keywords );
  if(protein_keywords) params.push("protein_keywords=" + protein_keywords );
  if(excluded_datasets) params.push("excluded_datasets=" + excluded_datasets );
  if(excluded_proteins) params.push("excluded_proteins=" + excluded_proteins );
  if(order) params.push("order=" + order );
  if(desc) params.push("desc=" + desc );
  if(limit) params.push("limit=" + limit );
  if(offset) params.push("offset=" + offset );

  var res = await sparqlet("dbi_make_filter_code", params.join("&"));
  res.select_line = "DISTINCT ?project_id ?project_title (GROUP_CONCAT(DISTINCT ?species_label_pre ; separator = \", \") AS ?species_label) (GROUP_CONCAT(DISTINCT ?species_label_pre ; separator = \", \") AS ?species_list) (COUNT(DISTINCT ?dataset) AS ?dataset_count)";
  if(line_count){
    res.select_line = "(COUNT(DISTINCT ?project_id) AS ?line_count)";
    res.code_limit = "";
  }
  return res;
};
```

## Endpoint

{{SPARQLIST_EP}}

## `dataset_items`

```sparql
#DEFINE sql:select-option "order"
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX ncit: <http://ncicb.nci.nih.gov/xml/owl/EVS/Thesaurus.owl#>
PREFIX unimod: <http://www.unimod.org/obo/unimod.obo#>
PREFIX uniprot: <http://purl.uniprot.org/core/>
PREFIX tax: <http://purl.bioontology.org/ontology/NCBITAXON/>
PREFIX owl: <http://www.geneontology.org/formats/oboInOwl#>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX sio: <http://semanticscience.org/resource/>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT {{filter.select_line}}
WHERE {
{{filter.code_value}}
  ?project jpo:hasDataset ?dataset ;
           dct:identifier ?project_id ;
           dct:title ?project_title .
  ?dataset jpo:hasProfile/jpo:hasSample/jpo:species/rdfs:seeAlso/skos:prefLabel ?species_label_pre .
  FILTER (LANG(?species_label_pre) = 'en') 
{{filter.code_dataset}}
}
{{filter.code_limit}}
```
