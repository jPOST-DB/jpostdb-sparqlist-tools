# peptide table (for DB interface) req. 'dbi_make_filter_code'

## Parameters

* `datasets` (Req. or proteins)
  * example: DS1624_1
* `proteins`  (Req. or datasets)
  * example: PRT1631_1_P01111 P01111
* `peptides`
  * example 
* `order` (Opt.)
  * default: mnemonic
* `desc` (Opt.)
  * example: 1
* `limit` (Opt.)
  * default: 10
* `offset` (Opt.)
  * default: 0
* `line_count`
  *example 1


## `filter`

```javascript
async ({datasets, proteins, peptides, order, desc, limit, offset, line_count}) => {
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
  if(proteins) params.push("proteins=" + proteins);
  if(peptides) params.push("peptides=" + peptides);
  if(order) params.push("order=" + order );
  if(desc) params.push("desc=" + desc );
  if(limit) params.push("limit=" + limit );
  if(offset) params.push("offset=" + offset );
  var res = await sparqlet("dbi_make_filter_code", params.join("&"));
  res.select_line = "DISTINCT ?peptide_id ?dataset_id ?accession ?mnemonic ?sequence ?full_name";
  if(line_count){
    let distinct = "";
    if (proteins || peptides) distinct = "DISTINCT ";
     res.select_line = "(COUNT(" + distinct + "?peptide) AS ?line_count)";
     res.code_limit = "";
   }
  return res;
};
```

## Endpoint

{{SPARQLIST_EP}}

## `protein_items`

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
    SELECT DISTINCT ?dataset_id ?protein ?db_protein ?accession
    WHERE {
      { 
        SELECT DISTINCT ?dataset
        WHERE {
{{filter.code_value}}
          ?dataset a jpo:Dataset .
          ?project jpo:hasDataset ?dataset .
{{filter.code_dataset}}
        }
      }
{{filter.code_protein_value}}
{{filter.code_up_label_value}}
      ?dataset dct:identifier ?dataset_id ;
               jpo:hasProtein ?db_protein .
{{filter.code_up_label}}
      ?db_protein jpo:hasDatabaseSequence ?protein ;
                  rdfs:label ?accession .
    }
  }

  OPTIONAL { # for obsoleted uniprot entry
    ?protein a uniprot:Protein ;  
             uniprot:mnemonic ?mnemonic . 
    ## virtuoso property path bug ?
  	# ?protein (uniprot:recommendedName|uniprot:submittedName)/uniprot:fullName ?full_name . 
    {
      ?protein uniprot:recommendedName/uniprot:fullName ?full_name .
    } UNION {
      ?protein uniprot:submittedName/uniprot:fullName ?full_name .
    }
  }
             
{{filter.code_peptide_value}}
{{filter.code_protein}}
  {
    ?db_protein jpo:hasPeptideEvidence/jpo:hasPeptide ?peptide .
  } UNION {
    ?db_protein jpo:hasIsoform/jpo:hasPeptideEvidence/jpo:hasPeptide ?peptide .    
  }
  ?peptide dct:identifier ?peptide_id;
           jpo:hasSequence [ a obo:MS_1001344 ;
                              rdf:value ?sequence ] .
}
{{filter.code_limit}}
```