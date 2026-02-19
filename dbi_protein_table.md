# protein table (for DB interface) req. 'dbi_make_filter_code'

* OPTIONAL の中が多いと遅いので sparql クエリ分割
  * 必要な offset limit 分だけ別クエリで取得
  * 一つの sparql json 形式に加工して返す
  * line count のときは unless でスキップ

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
  * eaxmple: JPO_006
* `instrument` (Opt.)
* `project_keywords` (Opt.)
  * example: iPS
* `dataset_keywords` (Opt.)
  * example: CH2O,by trypsin
* `protein_keywords` (Opt.)
* `excluded_datasets` (Opt.)
  * example: DS1631_1,DS1631_2
* `excluded_proteins` (Opt.)
* `order` (Opt.)
  * default: mnemonic
* `desc` (Opt.)
  * example: 1
* `limit` (Opt.)
  * default: 10
* `offset` (Opt.)
  * default: 0
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
  if(datasets) params.push("datasets=" + datasets);
  if(species) params.push("species=" + species);
  if(species_s) params.push("species_s=" + species_s);
  if(sample_type) params.push("sample_type=" + sample_type);
  if(cell_line) params.push("cell_line=" + cell_line );
  if(organ) params.push("organ=" + organ );
  if(disease) params.push("disease=" + disease );
  if(disease_s) params.push("disease_s=" + disease_s );
  if(modification) params.push("modification=" + modification );
  if(instrument_mode) params.push("instrument_mode=" + instrument_mode );
  if(instrument) params.push("instrument=" + instrument );
  if(project_keywords) params.push("project_keywords=" + project_keywords );
  if(dataset_keywords) params.push("dataset_keywords=" + dataset_keywords );
  if(protein_keywords) params.push("protein_keywords=" + protein_keywords );
  if(excluded_datasets) params.push("excluded_datasets=" + excluded_datasets );
  if(excluded_proteins) params.push("excluded_proteins=" + excluded_proteins );
  if(order) params.push("order=" + order );
  if(desc) params.push("desc=" + desc );
  if(limit) params.push("limit=" + limit );
  if(offset) params.push("offset=" + offset );

  var res = await sparqlet("dbi_make_filter_code", params.join("&"));
  res.select_line = "DISTINCT ?protein ?accession ?mnemonic";
  if(line_count){
    res.select_line = "(COUNT(DISTINCT ?protein) AS ?line_count)";
    res.code_limit = "";
  }
  if (res.code_value || res.code_dataset) res.has_filter = true;
  return res;
};
```

## Endpoint

{{SPARQLIST_EP}}

## `mnemonic`

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
  { 
    SELECT DISTINCT ?protein
    WHERE {
      {{#if filter.has_filter}}
      { 
        SELECT DISTINCT ?dataset
        WHERE {
{{filter.code_value}}
          ?dataset a jpo:Dataset .
          ?project jpo:hasDataset ?dataset .
{{filter.code_dataset}}
        }
      }
      {{/if}}
      ?dataset jpo:hasProtein ?db_prt .
      ?db_prt a obo:MS_1002401 ;
              jpo:hasDatabaseSequence ?protein .
    }
  }
  ?protein ^jpo:hasDatabaseSequence/rdfs:label ?accession .
  {{#unless line_count}}
  OPTIONAL {
    ?protein uniprot:mnemonic ?mnemonic .
  }
  {{/unless}}
  FILTER (! REGEX (?accession, "corona"))
{{filter.code_protein}}
}
{{filter.code_limit}}
```

## `uniprot`
```javascript
({mnemonic, line_count}) => {
  if (line_count) return 0;
  return mnemonic.results.bindings.map(d => "up:" + d.accession.value).join(" ");
}
```

## `sequence`
```sparql
#DEFINE sql:select-option "order"
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX uniprot: <http://purl.uniprot.org/core/>
PREFIX up: <http://purl.uniprot.org/uniprot/>
SELECT DISTINCT ?accession ?full_name (STRLEN (?sequence) AS ?length) ?sequence
WHERE {
  {{#unless line_count}}
  VALUES ?protein { {{uniprot}} }
  ?protein ^jpo:hasDatabaseSequence/rdfs:label ?accession ;
           (uniprot:recommendedName/uniprot:fullName)|(uniprot:submittedName/uniprot:fullName) ?full_name ; 
           uniprot:sequence ?seqEnt .
  FILTER (REGEX (STR (?seqEnt), ?accession))
  ?seqEnt a uniprot:Simple_Sequence ;
          rdf:value ?sequence .
{{/unless}}
}
```

## `protein_items`
```javascript
({line_count, mnemonic, sequence}) => {
  if (line_count) return mnemonic;
  let acc2seq = {};
  sequence.results.bindings.forEach(d => {
    acc2seq[d.accession.value] = {
      full_name: d.full_name,
      sequence: d.sequence,
      length: d.length
    }
  });
  mnemonic.head.vars.push("full_name", "length", "sequence");
  for (let i = 0; i < mnemonic.results.bindings.length; i++) {
    const acc = mnemonic.results.bindings[i].accession.value;
    if (acc2seq[acc]) {
      mnemonic.results.bindings[i].full_name = acc2seq[acc].full_name;
      mnemonic.results.bindings[i].sequence = acc2seq[acc].sequence;
      mnemonic.results.bindings[i].length = acc2seq[acc].length;
    }
  }
  return mnemonic;
}
```