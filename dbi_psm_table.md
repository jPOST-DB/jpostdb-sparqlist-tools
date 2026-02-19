# PSM table (for DB interface) req. 'dbi_make_filter_code'

## Parameters

* `proteins` (Req. or peptides)
  * example: PRT1631_1_P01111 P01111
* `peptides` (Req. or proteins)
  * example: SSSPYSKSPVSK, PEP1631_1_1
* `tax` (Req. when peptides are sequence)
  * example: 9606
* `datasets` (Opt.)
  * example: DS1631_1
* `order` (Opt.)
  * default: jpost_score
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
async ({datasets, proteins, peptides, tax, order, desc, limit, offset, line_count}) => {
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
  res.select_line = "DISTINCT ?psm_id ?sequence ?jpost_score ?charge ?calc_mass ?exp_mass";
  if(line_count){
    res.select_line = "(COUNT(DISTINCT ?psm_id) AS ?line_count)";
    res.code_limit = "";
  }
  if(peptides && !peptides.match(/^PEP\d+_/)){
    res.code_peptide_value = "  ?peptide jpo:hasSequence [ a obo:MS_1001344 ;\n                     rdf:value '" + peptides + "' ] .\n";
  }
  if(tax){
    let tmp = tax.match(/(\d+)/);
    tax = tmp[1];
    if(tax) res.code_tax = "  ?peptide ^jpo:hasPeptide/jpo:hasProfile/jpo:hasSample/jpo:species/rdfs:seeAlso/rdfs:subClassOf* tax:" + tax + " .";
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
{{filter.code_species_value}}
{{filter.code_protein_value}}
{{filter.code_peptide_value}}
{{filter.code_up_label_value}}
{{filter.code_up_label}}
{{filter.code_tax}}
  ?db_protein jpo:hasPeptideEvidence/jpo:hasPeptide ?peptide .
  ?peptide jpo:hasPsm ?psm ;
           jpo:hasSequence [ a obo:MS_1001344 ;
                               rdf:value ?sequence ] .
  ?psm dct:identifier ?psm_id ;
       sio:SIO_000216 [ a obo:MS_1000041 ;
                         sio:SIO_000300 ?charge ] ; 
       sio:SIO_000216 [ a jpo:UniScore ;
                          sio:SIO_000300 ?jpost_score ] ;
       sio:SIO_000216 [ a jpo:CalculatedMassToCharge ;
                         sio:SIO_000300 ?calc_mass ] ;
       sio:SIO_000216 [ a jpo:ExperimentalMassToCharge ;
                          sio:SIO_000300 ?exp_mass ] .
}
{{filter.code_limit}}
```

## `precision`

```javascript
({protein_items})=>{
  if(!protein_items.results.bindings[0].line_count){
    for(var i = 0; i < protein_items.results.bindings.length; i++){
      protein_items.results.bindings[i].calc_mass.value = parseFloat(protein_items.results.bindings[i].calc_mass.value).toPrecision(10);        
      protein_items.results.bindings[i].exp_mass.value = parseFloat(protein_items.results.bindings[i].exp_mass.value).toPrecision(10);        
    } 
  }
  return protein_items;
}
```