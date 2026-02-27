# dataset table in a project (for DB interface)

* 遅いので sparql クエリ分割
  * 必要な offset limit 分だけ別クエリで取得
  * javascript セクションで必要な sparql json 形式を返す
  * line count のときは unless でスキップ

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
    res.select_line = 'DISTINCT ?dataset_id ?species ?sample_type ?cell_line ?organ ?disease_class ?disease ?fractionation ?note (GROUP_CONCAT(distinct ?mod ; separator = ", ") AS ?modification) (GROUP_CONCAT(distinct ?raw ; separator = ", ") AS ?raw_file_name) ?raw_file_url';
  }
  return res;
};
```

## `info`

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
           ^jpo:hasDataset / dct:identifier ?project_id ;
           jpo:hasProfile/jpo:hasRawData/rdfs:label ?raw .
  ?sample jpo:species/rdfs:seeAlso/skos:prefLabel ?species .
  FILTER (LANG(?species) = 'en') 
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

## `datasets`
```javascript
({line_count, info}) => {
  if (line_count) return 0;
  return ":" + info.results.bindings.map(d => d.dataset_id.value).join(" :");
}
```

## `count`
```sparql
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX sio: <http://semanticscience.org/resource/>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT ?dataset_id ?protein_count ?peptide_count ?spectrum_count ?rawdata_count
WHERE {
  {{#unless line_count}}
  VALUES ?dataset { {{datasets}} }
  ?dataset dct:identifier ?dataset_id ;
           sio:SIO_000216 [ a jpo:NumOfLeadingProteins ;
                          sio:SIO_000300 ?protein_count ] ;
           sio:SIO_000216 [ a jpo:NumOfPeptides ;
                              sio:SIO_000300 ?peptide_count ] ;
           sio:SIO_000216 [ a jpo:NumOfSpectra ;
                              sio:SIO_000300 ?spectrum_count ] ;
           sio:SIO_000216 [ a jpo:NumOfRawData ;
                              sio:SIO_000300 ?rawdata_count ] .
  {{/unless}}
}
```

## `dataset_items`
```javascript
({line_count, info, count}) => {
  if (line_count) return info;
  let ds2count = {};
  count.results.bindings.forEach(d => {
    ds2count[d.dataset_id.value] = {
      protein_count: d.protein_count,
      peptide_count: d.peptide_count,
      spectrum_count: d.spectrum_count,
      rawdata_count: d.rawdata_count
    };
  });
  for (let i = 0; i < info.results.bindings.length; i++) {
    const dataset_id = info.results.bindings[i].dataset_id.value;
    info.results.bindings[i].protein_count = ds2count[dataset_id].protein_count;
    info.results.bindings[i].peptide_count = ds2count[dataset_id].peptide_count;
    info.results.bindings[i].spectrum_count = ds2count[dataset_id].spectrum_count;
    info.results.bindings[i].rawdata_count = ds2count[dataset_id].rawdata_count;
  }
  return info;
}
```