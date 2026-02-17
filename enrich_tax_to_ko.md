# Discontinued
- uniprot ko list from taxonomy (for Stanza 'slice_comparison' :enrich #2 KO)

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
SELECT DISTINCT ?up ?ko
FROM <http://jpost.org/graph/uniprot>
WHERE {
  ?up uniprot:organism <http://purl.uniprot.org/taxonomy/{{tax}}> ;
      rdfs:seeAlso ?ko .
  ?ko uniprot:database	<http://purl.uniprot.org/database/KO> .
}
```