# Discontinued ?
* table items for peptide sequence

* Peptide page, 配列から、ペプチドの情報
  * タンパク質上でのポジション、PSM の数、MS で判別不可能な（I,L）配列の有無など

## Parameters

* `pepseq`
  * default: TAFDEAIAELDTLNEESYK
* `tax`
  * default: 9606


## Endpoint

{{SPARQLIST_EP}}

## `count`

```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX tax: <http://identifiers.org/taxonomy/>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT (COUNT (DISTINCT ?psm) AS ?psm_count) ?ds_count (SAMPLE (?pep_id) AS ?pep_sample)
WHERE {
  {
    SELECT (COUNT (DISTINCT ?dataset) AS ?ds_count)
    WHERE {
      ?pep a jpo:Peptide ;
           jpo:hasSequence [ a obo:MS_1001344 ;
                               rdf:value "{{pepseq}}" ] .
      ?dataset jpo:hasPeptide ?pep ;
               jpo:hasProfile/jpo:hasSample/jpo:species tax:{{tax}} .
    }
  }
  ?pep a jpo:Peptide ;
       dct:identifier ?pep_id ;
       jpo:hasSequence [ a obo:MS_1001344 ;
                           rdf:value "{{pepseq}}" ] ;
       jpo:hasPsm ?psm .
  ?dataset jpo:hasPeptide ?pep ;
           jpo:hasProfile/jpo:hasSample/jpo:species tax:{{tax}} .
}
```


## `other`

```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX tax: <http://identifiers.org/taxonomy/>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT DISTINCT ?species ?tax
WHERE {
  ?pep a jpo:Peptide ;
       jpo:hasSequence [ a obo:MS_1001344 ;
                           rdf:value "{{pepseq}}" ] .
  ?pep ^jpo:hasPeptide/jpo:hasProfile/jpo:hasSample/jpo:species ?tax .
  ?tax rdfs:seeAlso/skos:prefLabel ?species .
  FILTER(?tax != tax:{{tax}})
}
```

## `pep_sample`

```javascript
({count})=>{
  return count.results.bindings[0].pep_sample.value;
};
```

## `position`

```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX uniprot: <http://purl.uniprot.org/core/>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX faldo: <http://biohackathon.org/resource/faldo#>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT DISTINCT ?uniprot ?mnemonic ?begin ?end
WHERE {
  ?pe jpo:hasPeptide :{{pep_sample}} ;
      a jpo:PeptideEvidence ;
      faldo:location/faldo:begin/faldo:position ?begin ;
      faldo:location/faldo:end/faldo:position ?end .
  ?prt jpo:hasPeptideEvidence ?pe ;
       rdfs:label ?uniprot ;
       jpo:hasDatabaseSequence ?up .
  ?up uniprot:mnemonic ?mnemonic .
}
ORDER BY ?prt_id
```

## `sim_pep`

```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX uniprot: <http://purl.uniprot.org/core/>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT DISTINCT ?sequence
WHERE {
  :{{pep_sample}} jpo:hasIndistinguishablePeptide ?sim_pep .
  ?sim_pep jpo:hasSequence [ a obo:MS_1001344 ;
                               rdf:value ?sequence ] .
}
ORDER BY ?sim_pep_id
```

## `modification`

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
  ?pep a jpo:Peptide ;
       jpo:hasSequence [ a obo:MS_1001344 ;
                           rdf:value "{{pepseq}}" ] .
  ?ds jpo:hasPeptide ?pep ;
      jpo:hasProfile/jpo:hasSample/jpo:species tax:{{tax}} ;
      a jpo:Dataset ;
      jpo:hasProfile/jpo:hasEnzymeAndModification/(jpo:fixedModification|jpo:variableModification) [ a ?mod_meta ;
                  jpo:modificationClass jpo:JPO_022 ] .
  ?pep jpo:hasPsm ?psm .
  ?psm jpo:hasModification [ a ?mod_psm ;
                           jpo:modificationSite ?site ;
                           faldo:location/faldo:position ?position ] .
  FILTER (?mod_psm = ?mod_meta)
  ?mod_psm rdfs:label ?mod . 
}
ORDER BY DESC (?position)
```

## `return`

```javascript
({pepseq, count, position, sim_pep, modification, other})=>{
  var list = modification.results.bindings;
  var mod = [];
  for(var i = 0; i < list.length; i++){
    mod.push({
      mod: list[i].mod.value, 
      position: list[i].position.value, 
      site: list[i].site.value
    });
  } 
  list = position.results.bindings;
  var protein = [];
  for(var i = 0; i < list.length; i++){
    protein.push({
      uniprot: list[i].uniprot.value, 
      mnemonic: list[i].mnemonic.value,
      begin: list[i].begin.value,
      end: list[i].end.value
    });
  }	
  var ref = pepseq.split('');
  list = sim_pep.results.bindings;
  var sim_pep = [];
  for(var i = 0; i < list.length; i++){
    var aa = list[i].sequence.value.split('');
    var seq = [];
    for(var j = 0; j < ref.length; j++){
      if(ref[j] != aa[j]) seq.push({matched: "", unmatched: aa[j]});
      else seq.push({matched: aa[j], unmatched: ""});
    }
    sim_pep.push({
      sequence: seq
    });
  }
  list = other.results.bindings;
  var species = [];
  for(var i = 0; i < list.length; i++){
    species.push({label: list[i].species.value, tax: list[i].tax.value.replace(/http:\/\/identifiers\.org\/taxonomy\//, '')});
  }
  var count = count.results.bindings[0];
  var type = "Unique peptide at MS level";
  if(sim_pep.length > 0) type = "Unique peptide (shared at MS level)";
  if(position.length > 1) type = "Shared peptide";
  return {
    sequence: pepseq,
    length: pepseq.length,
    psm_count: count.psm_count.value,
    ds_count: count.ds_count.value,
    type: type,
    mod: mod,
    protein: protein,
    sim_pep: sim_pep,
    other_species: species
  }
};
```

