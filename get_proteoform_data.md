# proteoform data via a protein (for Stanza 'proteoform_browser')

- 指定したタンパク質に Hit したペプチドが、Isoform 及び他のタンパク質のどこで共有 Hit するかを取得

## Parameters

* `uniprot` (Req.)
  * default: P01112
* `dataset` (Opt.)
  * default: DS1631_1 DS1632_1 DS1633_1


## `filter`

```javascript
({uniprot, dataset})=>{
  var code = "";
  if(dataset) code += "  VALUES ?dataset { :" + decodeURIComponent(dataset).split(/[ ,]/).join(" :") + " }\n";
  code += "  ?prt jpo:hasDatabaseSequence up:" + uniprot + " ;\n     a jpo:Protein .\n";
  if(dataset) code += "  ?prt ^jpo:hasProtein ?dataset .\n";
  code += "  ?prt jpo:hasPeptideEvidence/jpo:hasPeptide/^jpo:hasPeptide/^jpo:hasPeptideEvidence ?protein .\n";
  code += "  ?protein rdfs:label ?up .\n";
  return {code: code}
};
```

## Endpoint

{{SPARQLIST_EP}}

## `get_pep`

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX up: <http://purl.uniprot.org/uniprot/>
PREFIX uniprot: <http://purl.uniprot.org/core/>
PREFIX faldo: <http://biohackathon.org/resource/faldo#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT DISTINCT ?up ?seq ?begin ?end
WHERE {
{{filter.code}}
  ?protein jpo:hasPeptideEvidence ?ev .
  ?ev jpo:hasPeptide ?pep .
  ?ev faldo:location [ faldo:begin/faldo:position ?begin ;
                                   faldo:end/faldo:position ?end ].
  ?pep jpo:hasSequence [a obo:MS_1001344 ;
                       rdf:value ?seq ] .
}
ORDER BY ?begin ?end
```

## Endpoint

{{SPARQLIST_EP}}

## `get_seq`

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX up: <http://purl.uniprot.org/uniprot/>
PREFIX uniprot: <http://purl.uniprot.org/core/>
PREFIX faldo: <http://biohackathon.org/resource/faldo#>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT DISTINCT ?up (GROUP_CONCAT(DISTINCT ?reco_sub_name ; separator = ", ") AS ?name) ?iso ?seq
WHERE {
{{filter.code}}
  OPTIONAL { ?protein jpo:hasDatabaseSequence/(uniprot:recommendedName|uniprot:submittedName)/uniprot:fullName ?reco_sub_name . }
  ?protein jpo:hasDatabaseSequence/uniprot:sequence ?iso .
  FILTER (REGEX (STR (?iso), ?up))
  ?iso a uniprot:Simple_Sequence ;
          rdf:value ?seq .
}
ORDER BY ?iso
```

## Endpoint

{{SPARQLIST_EP}}

## `get_psm_num`

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX up: <http://purl.uniprot.org/uniprot/>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT ?seq (COUNT (DISTINCT ?psm) AS ?psm_num)
WHERE {
{{filter.code}}
  ?protein jpo:hasPeptideEvidence/jpo:hasPeptide ?pep .
  ?pep jpo:hasSequence [ a obo:MS_1001344;
                       rdf:value ?seq ] ;
       jpo:hasPsm ?psm .
}
```

## Endpoint

{{SPARQLIST_EP}}

## `get_prt_num`

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX up: <http://purl.uniprot.org/uniprot/>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT DISTINCT ?seq ?up_id
WHERE {
  {{filter.code}}
  ?protein jpo:hasPeptideEvidence/jpo:hasPeptide/jpo:hasSequence [ a obo:MS_1001344 ;
                                                 rdf:value ?seq ] .
  ?protein rdfs:label ?up_id .
}
```

## `return`

```javascript
({uniprot, get_pep, get_seq, get_psm_num, get_prt_num})=>{
  return {
    uniprot: uniprot, 
    pep: get_pep.results.bindings, 
    seq: get_seq.results.bindings, 
    psm_num: get_psm_num.results.bindings, 
    seq_prt: get_prt_num.results.bindings
  }
};
```