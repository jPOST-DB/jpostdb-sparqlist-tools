# table items of peptide sequence (for Stanza 'table_peptide')

* Peptide page, 配列から、ペプチドの情報の取得
  * タンパク質上でのポジション、PSM の数、MS で判別不可能な（I,L）配列の有無など

## Parameters

* `peptide`
  * default: YSPSQNSPIHHIPSR
* `tax`
  * default: 9606
  * example: TAX_9606, 9606


## `filter`

```javascript
({tax})=>{
  let tmp = tax.match(/(\d+)/);
  return {tax: "tax:" + tmp[1]};
}
```

## Endpoint

{{SPARQLIST_EP}}

## `table_items`
```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX tax: <http://identifiers.org/taxonomy/>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT DISTINCT (COUNT (DISTINCT ?psm) AS ?psm_count)
WHERE {
  ?peptide dct:identifier ?id ;
           jpo:hasSequence [ a obo:MS_1001344 ;
                               rdf:value "{{peptide}}" ] ;
       jpo:hasPsm ?psm .
  {{filter.tax}} ^jpo:species/^jpo:hasSample/^jpo:hasProfile/jpo:hasPeptide ?peptide .
}
```

## `get_protein`
```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX uniprot: <http://purl.uniprot.org/core/>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX faldo: <http://biohackathon.org/resource/faldo#>
PREFIX tax: <http://identifiers.org/taxonomy/>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT DISTINCT ?uniprot ?mnemonic ?begin ?end ?gene_name
WHERE {
  ?peptide jpo:hasSequence [ a obo:MS_1001344 ;
                             rdf:value "{{peptide}}" ] .
  {{filter.tax}} ^jpo:species/^jpo:hasSample/^jpo:hasProfile/jpo:hasPeptide ?peptide .
  ?pe jpo:hasPeptide ?peptide ;
      a jpo:PeptideEvidence ;
      faldo:location/faldo:begin/faldo:position ?begin ;
      faldo:location/faldo:end/faldo:position ?end .
  ?prt jpo:hasPeptideEvidence ?pe ;
       rdfs:label ?uniprot ;
       dct:identifier ?prt_id ;
       jpo:hasDatabaseSequence ?up .
  ?up uniprot:mnemonic ?mnemonic ;
      uniprot:encodedBy/skos:prefLabel ?gene_name .
}
ORDER BY DESC(?count)
```

## `get_sim_pep`
```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX uniprot: <http://purl.uniprot.org/core/>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX tax: <http://identifiers.org/taxonomy/>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT DISTINCT ?sim_pep_id ?sequence
WHERE {
  ?peptide jpo:hasSequence [ a obo:MS_1001344 ;
                             rdf:value "{{peptide}}" ] ;
           jpo:hasIndistinguishablePeptide ?sim_pep .
  {{filter.tax}} ^jpo:species/^jpo:hasSample/^jpo:hasProfile/jpo:hasPeptide ?peptide .
  ?sim_pep dct:identifier ?sim_pep_id ;
           jpo:hasSequence [ a obo:MS_1001344 ;
                               rdf:value ?sequence ] .
}
ORDER BY ?sim_pep_id
```

## `get_mod`
```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX ncit: <http://ncicb.nci.nih.gov/xml/owl/EVS/Thesaurus.owl#>
PREFIX unimod: <http://www.unimod.org/obo/unimod.obo#>
PREFIX uniprot: <http://purl.uniprot.org/core/>
PREFIX tax: <http://identifiers.org/taxonomy/>
PREFIX owl: <http://www.geneontology.org/formats/oboInOwl#>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX sio: <http://semanticscience.org/resource/>
PREFIX faldo: <http://biohackathon.org/resource/faldo#>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT DISTINCT ?mod ?position ?site
WHERE {
  ?peptide jpo:hasSequence [ a obo:MS_1001344 ;
                             rdf:value "{{peptide}}" ] .
  {{filter.tax}} ^jpo:species/^jpo:hasSample/^jpo:hasProfile/jpo:hasPeptide ?peptide .
  ?ds jpo:hasPeptide ?peptide ;
      a jpo:Dataset ;
      jpo:hasProfile/jpo:hasEnzymeAndModification/(jpo:fixedModification|jpo:variableModification) [ a ?mod_meta ;
                  jpo:modificationClass jpo:JPO_022 ] .
  ?peptide jpo:hasPsm ?psm .
  ?psm jpo:hasModification [ a ?mod_psm ;
                           jpo:modificationSite ?site ;
                           faldo:location/faldo:position ?position ] .
  FILTER (?mod_psm = ?mod_meta)
  ?mod_psm rdfs:label ?mod .
}
ORDER BY DESC (?position)
```

## `get_other_tax`
```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX tax: <http://identifiers.org/taxonomy/>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT DISTINCT ?tax ?label
WHERE {
  ?peptide jpo:hasSequence [ a obo:MS_1001344 ;
                             rdf:value "{{peptide}}" ] .
  ?tax ^jpo:species/^jpo:hasSample/^jpo:hasProfile/jpo:hasPeptide ?peptide .
  ?tax rdfs:seeAlso/skos:prefLabel ?label .
  FILTER (?tax != {{filter.tax}})
}
```

## `return`

```javascript
({peptide, table_items, get_protein, get_sim_pep, get_mod, get_other_tax})=>{
  var list = get_mod.results.bindings;
  var mod = [];
  for(var i = 0; i < list.length; i++){
    mod.push({
      mod: list[i].mod.value, 
      position: list[i].position.value, 
      site: list[i].site.value
    });
  } 
  list = get_protein.results.bindings;
  var protein = [];
  var gene_name_chk = {};
  for(var i = 0; i < list.length; i++){
    protein.push({
  //    prt_id: list[i].prt_id.value, 
      gene_name: list[i].gene_name.value, 
      uniprot: list[i].uniprot.value, 
      mnemonic: list[i].mnemonic.value,
      begin: list[i].begin.value,
      end: list[i].end.value
    });
    gene_name_chk[list[i].gene_name.value] = 1;
  }	
  var ref = peptide.split('');
  list = get_sim_pep.results.bindings;
  var sim_pep = [];
  for(var i = 0; i < list.length; i++){
    var aa = list[i].sequence.value.split('');
    var seq = [];
    for(var j = 0; j < ref.length; j++){
      if(ref[j] != aa[j]) seq.push({matched: "", unmatched: aa[j]});
      else seq.push({matched: aa[j], unmatched: ""});
    }
    sim_pep.push({
      pep_id: list[i].sim_pep_id.value, 
      sequence: seq
    });
  }
  var items = table_items.results.bindings[0];
  var type = "Unique peptide at MS level";
  if(sim_pep.length > 0) type = "Unique peptide (shared at MS level)";
  if(protein.length > 1){
    type = "Shared peptide";
    if(Object.keys(gene_name_chk).length == 1) type = "Unique peptide at gene name level (shared at UniProt entry level)";
  }
  var list = get_other_tax.results.bindings;
  var tax = [];
  for(var i = 0; i < list.length; i++){
    tax.push({tax: list[i].tax.value.replace(/.+identifiers\.org\/taxonomy\//, "TAX_"), label: list[i].label.value})
  }
  return {
   // id: items.id.value,
    sequence: peptide,
    length: peptide.length,
    psm_count: items.psm_count.value,
    type: type,
    mod: mod,
    protein: protein,
    sim_pep: sim_pep,
    other_tax: tax,
    peptide_id_flag: 0
  }
};
```
