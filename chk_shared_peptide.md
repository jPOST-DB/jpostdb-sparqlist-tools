# check shared peptide in isoforms and protein_families (for Stanza proteoform_browser)

- 指定したタンパク質がアイソフォームを持つか判定。他のタンパク質との間で共有ペプチドを持つか判定。
  - proteoform_browser stanza でデフォルトをアイソフォームにするか、タンパク質にするか、切り替えボタンを表示するかどうかに使用
  - Optional: dataset 指定で判定

## Parameters

* `uniprot` (Req.)
  * example: P01112
* `dataset` (Opt.)
  * example: DS1631_1 DS1631_2 DS1631_3


## `filter`

```javascript
({uniprot, dataset})=>{
  var code = "";
  if(dataset) code += "  VALUES ?dataset { :" + decodeURIComponent(dataset).split(/[ ,]/).join(" :") + " }\n";
  code += "  ?prt jpo:hasDatabaseSequence up:" + uniprot + " ;\n        a jpo:Protein .\n";
  if(dataset) code += "  ?prt ^jpo:hasProtein ?dataset .\n";
  return {code: code}
};
```

## Endpoint

{{SPARQLIST_EP}}/

## `chk_isoform`

```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX up: <http://purl.uniprot.org/uniprot/>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT ?iso
WHERE {
{{filter.code}}
  ?prt jpo:hasIsoform ?iso .
}
LIMIT 1
```

## Endpoint

{{SPARQLIST_EP}}/

## `chk_share`

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX up: <http://purl.uniprot.org/uniprot/>
PREFIX uniprot: <http://purl.uniprot.org/core/>
PREFIX faldo: <http://biohackathon.org/resource/faldo#>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT ?pep
WHERE {
 # VALUES ?type { jpo:SharedPeptide jpo:SharedPeptideAtMsLevel }
{{filter.code}}
  ?prt jpo:hasPeptideEvidence/jpo:hasPeptide ?pep .
  ?pep a jpo:SharedPeptide .
}
LIMIT 1
```

## `return`

```javascript
({chk_isoform, chk_share})=>{
  var isoform = false;
  var protein = false;
  if(chk_isoform.results.bindings[0] && chk_isoform.results.bindings[0].iso.value) isoform = true;
  if(chk_share.results.bindings[0] && chk_share.results.bindings[0].pep.value) protein = true;
  return {isoform: isoform, protein: protein};
};
```

