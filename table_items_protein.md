# table items of protein (for Stanza 'table_protein')

## Parameters

* `uniprot`
  * default: P01111
* `dataset`  (opt.)
  * example: DS1631_1 DS1631_2 DS1631_3 DS1632_1 DS1632_2 DS1632_3


## `filter`

```javascript
({uniprot, dataset})=>{
  var code = "";
  var code2 = "";
  if(dataset){
    code += "  VALUES ?dataset { :" + decodeURIComponent(dataset).split(/[\s,]+/).join(" :") + " }\n";
    code2 += code;
    code += "  ?prt ^jpo:hasProtein ?dataset .\n";	
    code2 += "  ?pep ^jpo:hasPeptide ?dataset .\n";
  }
  code += "  ?prt jpo:hasDatabaseSequence up:" + uniprot + " .\n";
  return {code: code, gene_name_code: code2}
};
```

## Endpoint

{{SPARQLIST_EP}}

## `table_items`
```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX up: <http://purl.uniprot.org/uniprot/>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX uniprot: <http://purl.uniprot.org/core/>
SELECT DISTINCT ?mnemonic ?full_name ?gid ?seq ?chr ?gene_name
WHERE {
  VALUES ?uniprot { up:{{uniprot}} }
  ?uniprot uniprot:mnemonic ?mnemonic ;
           uniprot:encodedBy/skos:prefLabel ?gene_name ;
           uniprot:sequence ?sequenceObject .
    FILTER (REGEX (STR (?sequenceObject), "{{uniprot}}"))
    ?sequenceObject a uniprot:Simple_Sequence ;
                    rdf:value ?seq . 
  ?uniprot (uniprot:recommendedName|uniprot:submittedName)/uniprot:fullName ?full_name .
  OPTIONAL {
    ?uniprot rdfs:seeAlso ?gid .
    ?gid uniprot:database <http://purl.uniprot.org/database/GeneID> .
  }
  OPTIONAL {
    ?uniprot uniprot:proteome ?chr .
  }
}
```

## `prt_type`
```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX up: <http://purl.uniprot.org/uniprot/>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT DISTINCT ?type ?lead
WHERE {
{{filter.code}}
  ?prt a ?type_id .
  FILTER( ?type_id != jpo:Protein )
  ?type_id rdfs:label ?type .
  OPTIONAL { ?prt jpo:hasLeadingProtein/rdfs:label ?lead . }
}
```

## `pep_count`
```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX up: <http://purl.uniprot.org/uniprot/>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT (COUNT(DISTINCT ?pepseq) AS ?count)
WHERE {
{{filter.code}}
  ?prt jpo:hasPeptideEvidence/jpo:hasPeptide/jpo:hasSequence [ a obo:MS_1001344 ;
                                                               rdf:value ?pepseq ] .
}
```

## `spectrum_count`
```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX up: <http://purl.uniprot.org/uniprot/>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT (COUNT(DISTINCT ?spectrum) AS ?count) 
WHERE {
  {{filter.code}}
  ?prt jpo:hasPeptideEvidence/jpo:hasPeptide/jpo:hasPsm/jpo:hasSpectrum ?spectrum .
}
```

## `uniq_pep_count`
```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX up: <http://purl.uniprot.org/uniprot/>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT (COUNT(DISTINCT ?pepseq) AS ?count)
WHERE {
{{filter.code}}
  ?prt jpo:hasPeptideEvidence/jpo:hasPeptide ?pep .
  ?pep a jpo:UniquePeptideAtMsLevel ;
       jpo:hasSequence [ a obo:MS_1001344 ;
                         rdf:value ?pepseq ] .
}
```

## `uniq_pep_gene_name_base`

```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX up: <http://purl.uniprot.org/uniprot/>
PREFIX uniprot: <http://purl.uniprot.org/core/>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT DISTINCT ?genes ?seq ?gene
WHERE {
  {{filter.gene_name_code}}
  up:{{uniprot}} uniprot:encodedBy/skos:prefLabel ?gene ;
            ^jpo:hasDatabaseSequence/jpo:hasPeptideEvidence/jpo:hasPeptide ?pep .
  ?genes ^skos:prefLabel/^uniprot:encodedBy/^jpo:hasDatabaseSequence/jpo:hasPeptideEvidence/jpo:hasPeptide ?pep .
  ?pep jpo:hasSequence/rdf:value ?seq .
}
```

## `return`

```javascript
({dataset, uniprot, table_items, prt_type, pep_count, spectrum_count, uniq_pep_count, uniq_pep_gene_name_base})=>{
  var list = uniq_pep_gene_name_base.results.bindings;
  var peps = {};
  for(var i = 0; i < list.length; i++){
    if(list[i].gene.value != list[i].genes.value) peps[list[i].seq.value] = 1;
  }
  var uniq_pep_count_gene = pep_count.results.bindings[0].count.value - Object.keys(peps).length;
  var data = table_items.results.bindings[0];
  var lead = [];
  var single_ds = false;
  if(dataset && dataset.split(/ +/).length == 1) single_ds = true;
  if(single_ds == true){
    var list = prt_type.results.bindings;
    if(!list[0].type.value.match("leading protein")){	
      for(var i = 0; i < list.length; i++){	
        if(list[i].lead) lead.push(list[i].lead.value);	
      }	
    }
  }
  var human = 0;
  if(data.mnemonic.value.match(/_HUMAN$/)) human = 1;
  var gid = "";
  if(data.gid) gid = data.gid.value.replace("http://purl.uniprot.org/geneid/", "");
  var chr = "";
  if(data.chr) chr = data.chr.value.match(/#([^#]+)$/)[1].replace("%20", " ");
  return {
    accession: uniprot,
    sequence: data.seq.value,
    length: data.seq.value.length, 
    id: data.mnemonic.value, 
    protein_name: data.full_name.value,
    gene_name: data.gene_name.value,
    pep_count: pep_count.results.bindings[0].count.value.replace(/(\d)(?=(\d{3})+$)/g , '$1,'),
    spc_count: spectrum_count.results.bindings[0].count.value.replace(/(\d)(?=(\d{3})+$)/g , '$1,'),
    uniq_pep_count: uniq_pep_count.results.bindings[0].count.value.replace(/(\d)(?=(\d{3})+$)/g , '$1,'),
    uniq_pep_count_gene: uniq_pep_count_gene.toString().replace(/(\d)(?=(\d{3})+$)/g , '$1,'),
    single_ds: single_ds,
    prt_type: prt_type.results.bindings[0].type.value,
    leading: lead.join(", ", lead),
    human: human,
    position: chr,
    gid: gid
         }
};
```

