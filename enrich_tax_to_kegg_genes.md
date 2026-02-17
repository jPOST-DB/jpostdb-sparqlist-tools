# uniprot - kegg genes list from taxonomy (for Stanza 'slice_comparison' :enrich #2 KO)

- 指定された種の UniProt - KEGG Genes 対応表を取得
  - enrich_tax_to_ko から変更
  
## Parameters

* `tax`
  * default: 9606
  
## endpoint

{{SPARQLIST_EP}}

## `up_ko`
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX up: <http://purl.uniprot.org/uniprot/>
PREFIX uniprot: <http://purl.uniprot.org/core/>
SELECT DISTINCT ?up ?gene
FROM <http://jpost.org/graph/uniprot>
WHERE {
  ?up uniprot:organism <http://purl.uniprot.org/taxonomy/{{tax}}> ;
      rdfs:seeAlso ?gene .
  ?gene uniprot:database <http://purl.uniprot.org/database/KEGG> .
}
```