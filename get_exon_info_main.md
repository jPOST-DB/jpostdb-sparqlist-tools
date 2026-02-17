# exon intron data form togogenome.org (for Stanza 'proteoform_brpwser')

- Exon, Intron 構造情報を取得
  - togogenome の endpoint から

## Parameters

* `uniprot`
  * default: P01112
* `endpoint`
  * default: http://dev.togogenome.org/sparql
  
## Endpoint

{{endpoint}}

## `get_exon`

```sparql
PREFIX insdc: <http://ddbj.nig.ac.jp/ontologies/nucleotide/>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX skoc: <http://www.w3.org/2004/02/skos/>
PREFIX faldo: <http://biohackathon.org/resource/faldo#>

SELECT DISTINCT ?id ?location  ?sequence
FROM <http://togogenome.org/graph/refseq>
FROM <http://togogenome.org/graph/tgup>
WHERE {
  ?gene rdfs:seeAlso <http://identifiers.org/uniprot/{{uniprot}}> .
  ?gene skos:exactMatch ?refseq_gene .
  ?refseq_gene a insdc:Gene .
  ?seq obo:so_part_of ?refseq_gene ;
       a insdc:Coding_Sequence ;
       insdc:translation ?sequence ;
       insdc:location ?location .
  BIND( REPLACE( STR(?seq), ".+:", "") AS ?id)
}
ORDER BY ?id
```

## `return`

```javascript
({uniprot, get_exon})=>{
  var list = get_exon.results.bindings;
  var up = uniprot;
  var r = [];
  for(var i = 0; i < list.length; i++){
    var obj = {
      sequence: list[i].sequence.value,
      location: list[i].location.value
    }
    r.push(obj)
  }
  return {uniprot: up, isoform: r};
};
```

