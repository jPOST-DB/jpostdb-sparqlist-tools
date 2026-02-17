# Taxon name to Taxon number (for DB interface)

## Parameters

* `string`
  * default: vertebrates,plants

## `to_code`

```javascript
({string})=>{
  return {code: "\"" + decodeURIComponent(string).toLowerCase().split(",").join("\" \"") + "\""};
};
```

## Endpoint

{{SPARQLIST_EP}}

## `search_tax_num`

```sparql
PREFIX skos: <http://www.w3.org/2004/02/skos/core#> 
PREFIX uniprot: <http://purl.uniprot.org/core/>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
SELECT DISTINCT ?taxon ?id
  WHERE {
  VALUES ?species_word { {{to_code.code}} }
  ?id (uniprot:scientificName|uniprot:commonName|uniprot:otherName) ?taxon .
  FILTER (LCASE(?taxon) = ?species_word)
}
```
