# PTM init info (for Stanza 'protein_browser')

- タンパク質の PTM, Phospho linkage, UniProt AA annotation, mutation などの有無を確認
  - Stanza でスイッチの生成に使用
  - TogoVar から variant を取ってくる部分は要修正
    - ENSP で同一性チェックしていたが、TogoVar から ENSP 情報がなくなった？

## Parameters

* `uniprot` Uniprot ID
  * default: Q9NYF8
* `dataset` (Opt.)
  * example: DS1740_1 DS1701_1 DS1742_1


## `filter`

```javascript
({dataset, uniq}) => {
  var value = "";
  if(dataset){
    value += "  VALUES ?dataset { :" + decodeURIComponent(dataset).split(/[ ,]/).join(" :") + " }\n";
  }
  return { value: value };
};
```

## Endpoint

{{SPARQLIST_EP}}

## `get_ptm_list`

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX up: <http://purl.uniprot.org/uniprot/>
PREFIX faldo: <http://biohackathon.org/resource/faldo#>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX unimod: <http://www.unimod.org/obo/unimod.obo#UNIMOD_>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT ?mod ?mod_label (COUNT (DISTINCT ?pos) AS ?count)
WHERE {
{{filter.value}}
  ?dataset jpo:hasProtein ?protein .
  ?protein jpo:hasDatabaseSequence up:{{uniprot}} ;
           jpo:hasPeptideEvidence ?pe .
  ?pe faldo:location/faldo:begin/faldo:position ?begin ;
      jpo:hasPeptide/jpo:hasPsm/jpo:hasModification ?blank .
  ?blank a ?mod ;
         faldo:location/faldo:position ?position .
  ?mod rdfs:label ?mod_label . 
  BIND (?position + ?begin - 1 AS ?pos)
  ?dataset jpo:hasProfile/jpo:hasEnzymeAndModification/(jpo:fixedModification|jpo:variableModification) [ a ?ptm ;
                                                                                                            jpo:modificationClass jpo:JPO_022 ] .
  FILTER (?ptm = ?mod) 
}
ORDER BY DESC (?count)
```

## `get_seq`

```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX up: <http://purl.uniprot.org/uniprot/>
PREFIX uniprot: <http://purl.uniprot.org/core/>
SELECT ?iso ?seq
WHERE {
  up:{{uniprot}} uniprot:sequence ?iso .
  FILTER (REGEX (STR (?iso), "{{uniprot}}"))
  ?iso a uniprot:Simple_Sequence ;
          rdf:value ?seq .
}
ORDER BY ?iso
LIMIT 1
```

## `get_phospho_link`

```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX up: <http://purl.uniprot.org/uniprot/>
PREFIX faldo: <http://biohackathon.org/resource/faldo#>
PREFIX unimod: <http://www.unimod.org/obo/unimod.obo#UNIMOD_>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT ?position_a ?position_b
WHERE {
  VALUES ?mods { unimod:21 }
{{filter.value}}
  ?mods rdfs:label ?mod_label .
  ?dataset jpo:hasProtein ?protein .
  ?protein jpo:hasDatabaseSequence up:{{uniprot}} ;
           jpo:hasPeptideEvidence/jpo:hasPeptide/jpo:hasPsm ?psm .
  ?psm jpo:hasModification [ a ?mods ;
                              faldo:location/faldo:position ?position_a ] .
  ?psm jpo:hasModification [ a ?mods ;
                              faldo:location/faldo:position ?position_b ] .
  FILTER (?position_a < ?position_b)
}
LIMIT 1
```

## `get_up_info`

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX up: <http://purl.uniprot.org/uniprot/>
PREFIX faldo: <http://biohackathon.org/resource/faldo#>
PREFIX uniprot: <http://purl.uniprot.org/core/>
SELECT DISTINCT ?anotation_type
WHERE {
  VALUES ?anotation_type { uniprot:Modified_Residue_Annotation uniprot:Natural_Variant_Annotation }
  up:{{uniprot}} uniprot:annotation ?an .
  ?an  a  ?anotation_type .
}
LIMIT 1
```

## `get_mutate`

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX up: <http://purl.uniprot.org/uniprot/>
PREFIX faldo: <http://biohackathon.org/resource/faldo#>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX unimod: <http://www.unimod.org/obo/unimod.obo#UNIMOD_>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT ?mutation
WHERE {
  {{filter.value}}
  ?dataset jpo:hasProtein ?protein .
  ?protein jpo:hasDatabaseSequence up:{{uniprot}} ;
           jpo:hasPeptideEvidence/jpo:hasPeptide/jpo:hasPsm/jpo:hasMutation ?mutation .
}
LIMIT 1
```

## `symbol`

```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX core: <http://purl.uniprot.org/core/>
PREFIX up: <http://purl.uniprot.org/uniprot/>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
SELECT ?symbol ?ensp ?type
FROM <http://jpost.org/graph/uniprot>
WHERE {
  up:{{uniprot}} core:encodedBy/skos:prefLabel ?symbol ;
                                           rdfs:seeAlso ?enst .
  ?enst core:translatedTo ?ensp .
  OPTIONAL { ?enst rdfs:seeAlso/rdf:type ?type .}
  FILTER (REGEX (?ensp, "ENSP"))
}
```

## `string`

```javascript
({symbol})=>{
  let obj = { symbol: "--------", ensp: "ENSP"};
  if(symbol.results.bindings[0]){
    let flag = 0;
    if(symbol.results.bindings[1]){
      for(let i = 0; i < symbol.results.bindings.length; i++){
        if(symbol.results.bindings[i].type && symbol.results.bindings[i].type.value.match("Simple_Sequence")){
          obj.symbol = symbol.results.bindings[i].symbol.value;
          obj.ensp = symbol.results.bindings[i].ensp.value.replace("http://rdf.ebi.ac.uk/resource/ensembl.protein/", "");
          flag = 1;
          break;
        }
      }
    }
    if(flag == 0){
      obj.symbol = symbol.results.bindings[0].symbol.value;
      obj.ensp = symbol.results.bindings[0].ensp.value.replace("http://rdf.ebi.ac.uk/resource/ensembl.protein/", "");
    }
  }
  return obj;
}
```

## Endpoint

https://grch38.togovar.org/proxy/sparql

## `get_tgv`

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX tgdb: <http://togodb.org/ontology/proteome_ips_test#>
#PREFIX tgvo: <http://togovar.biosciencedbc.jp/ontology/>
PREFIX tgvo: <http://togovar.biosciencedbc.jp/vocabulary/>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX dct: <http://purl.org/dc/terms/>
SELECT DISTINCT ?tgv ?hgvs
WHERE {
  ?var tgvo:hasConsequence ?cons ;
       dct:identifier ?tgv .
  ?cons a ?cons_type ;
        tgvo:hgvsp ?hgvs ;
        tgvo:gene/rdfs:label "{{string.symbol}}" .
  FILTER (REGEX (?hgvs, "{{string.ensp}}"))
  FILTER (!REGEX (?hgvs, "="))
}
LIMIT 1
```

## `return`

```javascript
({get_seq, get_ptm_list, get_phospho_link, get_up_info, get_mutate, get_tgv}) => {
  var obj = { 
    seq: get_seq.results.bindings[0].seq.value, 
    p_link: false,
    up_info: false,
    mutation: false,
    ptm_list: get_ptm_list.results.bindings,
    tgv: false,
  };	
  if(get_phospho_link.results.bindings[0]) obj.p_link = true;
  if(get_up_info.results.bindings[0]) obj.up_info = true;
  if(get_mutate.results.bindings[0]) obj.mutation = true;
  if(get_tgv.results.bindings[0]) obj.tgv = true;
  return obj;
};
```

