# uniprot to taxonomy (for Stanza 'slice_comparison' :enrich #1)

- UniProt からターゲットの種を取得

## Parameters

* `uniprot` (Req.)
  * default: Q9NYF8
  
## endpoint

{{SPARQLIST_EP}}

## `up_tax`
```sparql
PREFIX up: <http://purl.uniprot.org/uniprot/>
PREFIX uniprot: <http://purl.uniprot.org/core/>
SELECT DISTINCT ?tax
FROM <http://jpost.org/graph/uniprot>
WHERE {
  up:{{uniprot}} uniprot:organism ?tax .
}
```