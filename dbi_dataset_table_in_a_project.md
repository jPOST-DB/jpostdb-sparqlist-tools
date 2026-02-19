# dataset table in a project (for DB interface)

## Parameters

* `project` (Opt.)
  * default: JPST001631
* `limit`
  * default:
* `offset`
  * default:
* `line_count`
  * default:

## Endpoint

{{SPARQLIST_EP}}

## `filter`
```javascript
async ({limit, offset, line_count}) => {
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
  if(limit) params.push("limit=" + limit );
  if(offset) params.push("offset=" + offset );
  
  var res = await sparqlet("dbi_make_filter_code", params.join("&"));
  res.select_line = "DISTINCT ?dataset_id ?project_id ?project_title ?project_date ?species_label ?protein_count ?spectrum_count";
  if (line_count) {
    res.select_line = "(COUNT(DISTINCT ?dataset_id) AS ?line_count)";
    res.code_limit = "";
  } else {
    res.select_line = 'DISTINCT ?dataset_id ?species ?sample_type ?cell_line ?organ ?disease_class ?disease ?fractionation ?protein_count ?peptide_count ?spectrum_count ?rawdata_count ?note (GROUP_CONCAT(distinct ?mod ; separator = ", ") AS ?modification) (GROUP_CONCAT(distinct ?raw ; separator = ", ") AS ?raw_file_name) ?raw_file_url';
  }
  return res;
};
```

## `dataset_items`

```sparql
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
  :{{project}} jpo:hasDataset ?dataset .
  ?dataset a jpo:Dataset ;
           dct:identifier ?dataset_id ;
           jpo:hasProfile/jpo:hasSample ?sample ;
           sio:SIO_000216 [ a jpo:NumOfLeadingProteins ;
                          sio:SIO_000300 ?protein_count ] ;
           sio:SIO_000216 [ a jpo:NumOfPeptides ;
                              sio:SIO_000300 ?peptide_count ] ;
           sio:SIO_000216 [ a jpo:NumOfSpectra ;
                              sio:SIO_000300 ?spectrum_count ] ;
            sio:SIO_000216 [ a jpo:NumOfRawData ;
                              sio:SIO_000300 ?rawdata_count ] ;
           jpo:hasProfile/jpo:hasRawData/rdfs:label ?raw .
  ?sample jpo:species/rdfs:seeAlso/skos:prefLabel ?species .
  FILTER (LANG(?species) = 'en') 
  ?project jpo:hasDataset ?dataset ;
           dct:identifier ?project_id .
  OPTIONAL { 
    ?sample jpo:sampleType/rdfs:label ?sample_type . 
  }
  OPTIONAL { 
    ?sample jpo:cellLine/rdfs:label ?cell_line_tmp .
    FILTER (LANG(?cell_line_tmp) = 'en' OR LANG(?cell_line_tmp) = '')
    BIND(STR(?cell_line_tmp) AS ?cell_line) # delete lang 
  }
  OPTIONAL { 
    ?sample jpo:organ/rdfs:label ?organ . 
  }
  OPTIONAL { 
    ?sample jpo:diseaseClass/rdfs:label ?disease_class_pre .
    FILTER (LANG(?disease_class_pre) = 'en' OR LANG(?disease_class_pre) = '')
    BIND (STR(?disease_class_pre) AS ?disease_class)
  }
  OPTIONAL { 
    ?sample jpo:disease/rdfs:label ?disease . 
    FILTER (LANG(?disease) = 'en' OR LANG(?disease) = '')
  }
  OPTIONAL { 
    ?sample rdfs:comment ?note . 
  }
  OPTIONAL {
    ?dataset jpo:hasProfile/jpo:hasFractionation/jpo:hasFractionationType
             [ 
               a jpo:SubcellularFractionation;
               rdfs:label ?fractionation
             ] . 
  }
  OPTIONAL {
    ?dataset jpo:hasProfile/jpo:hasEnzymeAndModification/jpo:variableModification [
      a ?unimod ;
      jpo:modificationClass jpo:JPO_022
    ] .
    ?unimod rdfs:label ?mod.
    FILTER(REGEX(STR(?unimod), "UNIMOD"))
  }
  BIND(CONCAT('https://repository.jpostdb.org/entry/', ?project_id) AS ?raw_file_url)
}
{{filter.code_limit}}
```
