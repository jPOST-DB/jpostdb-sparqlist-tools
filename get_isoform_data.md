# isoform data (for Stanza 'proteoform_browser')

- 指定した UniProt の isoform 情報を取得

## Parameters

* `uniprot` (Req.)
  * default: O00429
* `dataset` (Opt.)
  * example: DS1631_1 DS1632_1 DS1633_1


## `filter`

```javascript
({uniprot, dataset})=>{
  if(!dataset && uniprot.match(/^PRT\d+_\d+_\w+$/)){
    dataset = "DS" + uniprot.match(/^PRT(\d+_\d+)_/)[1];
    uniprot = uniprot.match(/^PRT\d+_\d+_(\w+)/)[1];
  }
  var code = "";
  if(dataset) code += "  VALUES ?dataset { :" + decodeURIComponent(dataset).split(/[ ,]/).join(" :") + " }\n";
  code += "  ?prt jpo:hasDatabaseSequence up:" + uniprot + " ;\n        a jpo:Protein .\n";
  if(dataset) code += "  ?prt ^jpo:hasProtein ?dataset .\n";
  return {code: code, uniprot: uniprot, dataset: dataset}
};
```

## Endpoint

{{SPARQLIST_EP}}

## `get_pep`

```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX uniprot: <http://purl.uniprot.org/core/>
PREFIX up: <http://purl.uniprot.org/uniprot/>
PREFIX faldo: <http://biohackathon.org/resource/faldo#>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT DISTINCT (REPLACE(REPLACE(STR(?up_iso), "http://purl.uniprot.org/isoforms/", ""), "http://purl.uniprot.org/uniprot/", "") AS ?up) ?seq ?begin ?end ?type
WHERE {
{{filter.code}}
  ?prt jpo:hasIsoform* ?iso .
  ?iso jpo:hasDatabaseSequence ?up_iso ;
           jpo:hasPeptideEvidence ?pe .
  ?pe jpo:hasPeptide ?pep .
  ?pep jpo:hasSequence [ a obo:MS_1001344 ;
                                         rdf:value ?seq ] .
  ?pe faldo:location ?location .
  ?location faldo:begin/faldo:position ?begin ;
                   faldo:end/faldo:position ?end .
 # OPTIONAL {?pep a ?type .
 #           FILTER (?type = jpo:UniquePeptide)}
}ORDER BY ?begin ?end
```

## `get_seq`

```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX uniprot: <http://purl.uniprot.org/core/>
PREFIX up: <http://purl.uniprot.org/uniprot/>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT DISTINCT ?up ?seq
WHERE {
  {
    SELECT DISTINCT ?up ?seq
    WHERE {
{{filter.code}}
      ?prt rdfs:label ?up ;
           jpo:hasDatabaseSequence/uniprot:sequence [ a uniprot:Simple_Sequence ;
                                                        rdf:value ?seq ] .
    }
  } UNION {
    SELECT DISTINCT (REPLACE(STR(?up_iso), "http://purl.uniprot.org/isoforms/", "") AS ?up) ?seq
    WHERE {
{{filter.code}}
      ?prt jpo:hasIsoform ?iso .
      ?iso jpo:hasDatabaseSequence ?up_iso .
      ?up_iso rdf:value ?seq .
    } ORDER BY ?up
  }
}
```

## `get_psm_num`

```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX uniprot: <http://purl.uniprot.org/core/>
PREFIX up: <http://purl.uniprot.org/uniprot/>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT DISTINCT ?seq (COUNT (DISTINCT ?psm) AS ?psm_num)
WHERE {
{{filter.code}}
  ?prt jpo:hasIsoform* ?iso .
  ?iso jpo:hasPeptideEvidence/jpo:hasPeptide ?pep .
  ?pep jpo:hasSequence [ a obo:MS_1001344 ;
                           rdf:value ?seq ] ;
       jpo:hasPsm ?psm .
}
```

## `get_seq_prt`

```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX uniprot: <http://purl.uniprot.org/core/>
PREFIX up: <http://purl.uniprot.org/uniprot/>
PREFIX faldo: <http://biohackathon.org/resource/faldo#>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT DISTINCT (REPLACE(REPLACE(STR(?up_iso), "http://purl.uniprot.org/isoforms/", ""), "http://purl.uniprot.org/uniprot/", "") AS ?up_id) ?seq
WHERE {
{{filter.code}}
  ?prt jpo:hasIsoform* ?iso .
  ?iso jpo:hasDatabaseSequence ?up_iso ;
       jpo:hasPeptideEvidence/jpo:hasPeptide/jpo:hasSequence [ a obo:MS_1001344 ;
                                                                 rdf:value ?seq ] .
}
```

## `get_shared_count`

```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX uniprot: <http://purl.uniprot.org/core/>
PREFIX up: <http://purl.uniprot.org/uniprot/>
PREFIX faldo: <http://biohackathon.org/resource/faldo#>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT ?seq (COUNT (DISTINCT ?up_iso) AS ?count)
WHERE {
{{filter.code}}
  ?prt jpo:hasIsoform* ?iso .
  ?iso jpo:hasDatabaseSequence ?up_iso ;
       jpo:hasPeptideEvidence/jpo:hasPeptide/jpo:hasSequence [ a obo:MS_1001344 ;
                                                                 rdf:value ?seq ] .
}
```

##  `up_to_tax`

```sparql
PREFIX up: <http://purl.uniprot.org/uniprot/>
PREFIX uniprot: <http://purl.uniprot.org/core/>
SELECT DISTINCT ?tax
FROM <http://jpost.org/graph/uniprot>
WHERE {
  up:{{uniprot}} uniprot:organism ?tax .
}
```

## Endpoint

http://togogenome.org/sparql

## `get_exon`

```sparql
PREFIX insdc: <http://ddbj.nig.ac.jp/ontologies/nucleotide/>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX skoc: <http://www.w3.org/2004/02/skos/>
PREFIX faldo: <http://biohackathon.org/resource/faldo#>
SELECT DISTINCT ?id ?location ?seq
FROM <http://togogenome.org/graph/refseq>
FROM <http://togogenome.org/graph/tgup>
WHERE {
  ?gene rdfs:seeAlso <http://identifiers.org/uniprot/{{filter.uniprot}}> .
  ?gene skos:exactMatch ?refseq_gene .
  ?refseq_gene a insdc:Gene .
  ?nuc obo:so_part_of ?refseq_gene ;
       a insdc:Coding_Sequence ;
       insdc:translation ?seq ;
       insdc:location ?location .
  BIND( REPLACE( STR(?nuc), ".+:", "") AS ?id)
}
ORDER BY ?id
```

## `return`

```javascript
({filter, get_pep, get_seq, get_psm_num, get_seq_prt, get_exon, get_shared_count, up_to_tax})=>{
  var uniprot = filter.uniprot;
  var seq = get_seq.results.bindings;
  var iso = get_exon.results.bindings;	
  var exon = [];
  var up_list = [];
  for(var i = 0; i < seq.length; i++){
    up_list.push({up: seq[i].up.value, eval: 0});
    for(var j = 0; j < iso.length; j++){
      if(seq[i].seq.value == iso[j].seq.value){
        var obj = {
          uniprot: seq[i].up.value,
          isoform: [{location: iso[j].location.value, sequence: seq[i].seq.value}]
        };
        exon.push(obj)
        continue;
      }
    }   
  }
  var num2color = function(num){
    // 99 = 153, cc = 204, ff = 255
    // 99ccff => ccff99 (0 => 1)
    var r = parseInt((204 - 153) * num) + 153;
    var g = parseInt((255 - 204) * num) + 204;
    var b = parseInt((153 - 255) * num) + 255;
    return "#" + r.toString(16) + g.toString(16) + b.toString(16);
  };
  var iso_num = get_seq.results.bindings.length;
  var pep = get_shared_count.results.bindings;
  var pep_color = [];
  for(var i = 0; i < pep.length; i++){
  	pep_color[i] = {
      seq: pep[i].seq.value,
      count: pep[i].count.value
    }
    if(parseInt(pep[i].count.value) == 1){
      pep_color[i].color = "#ff9999";
    }else{
      pep_color[i].color = num2color(parseInt(pep[i].count.value) / iso_num);
    }
  }
  return {
    tax: up_to_tax.results.bindings[0].tax.value.replace("http://purl.uniprot.org/taxonomy/", ""),
    list: up_list, 
    exon: exon,
    data: {
      uniprot: uniprot,
      pep: get_pep.results.bindings,
      seq: get_seq.results.bindings	,
      psm_num: get_psm_num.results.bindings,
      seq_prt: get_seq_prt.results.bindings,
      pep_color: pep_color
         }
  };
};
```
