# Develop: protein table for slice w/ protein estimation (for DB interface) (original: dbi_protein_table)

## Parameters

* `datasets`
  * default: DS1705_1,DS1705_2,DS1705_3
* `order` (Opt.)
  * default: mnemonic
* `desc` (Opt.)
  * example: 1
* `limit` (Opt.)
  * default: 10
* `offset` (Opt.)
  * default: 0
* `line_count`
  * example 1


## `json`

```javascript
async ({datasets})=>{
  const options = {
    method: 'get',
    headers: {
      'Accept': 'application/json',
      'Content-Type': 'application/x-www-form-urlencoded'
    }
  };
  let api = "https://tools.jpostdb.org/proteins_by_datasets?accession=true&dataset=" + encodeURIComponent(dataset.replace(/[, ]+/g, ","));
  try{
    const res = await fetch(api, options).then(res=>res.json());
    return res.proteins;
  }catch(error){
    console.log(error);
  }
};
```

## Endpoint

{{SPARQLIST_EP}}

## `taxonomy`

```sparql
PREFIX uniprot: <http://purl.uniprot.org/core/>
SELECT ?tax
WHERE {
  <http://purl.uniprot.org/uniprot/{{json.[0]}}>  uniprot:organism ?tax .
}
```

## Endpoint

{{SPARQLIST_EP}}

## `protein_items`

```sparql
#DEFINE sql:select-option "order"
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX uniprot: <http://purl.uniprot.org/core/>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT DISTINCT ?protein ?accession ?mnemonic ?full_name (STRLEN (?sequence) AS ?length) ?sequence
WHERE {
  ?protein ^jpo:hasDatabaseSequence/rdfs:label ?accession .
#  GRAPH <http://jpost.org/graph/uniprot> {
    ?protein a uniprot:Protein ;
             uniprot:organism <{{taxonomy.results.bindings.[0].tax.value}}> ;
             uniprot:mnemonic ?mnemonic ;
             uniprot:sequence ?seqEnt ;
             (uniprot:recommendedName/uniprot:fullName)|(uniprot:submittedName/uniprot:fullName) ?full_name . 
    FILTER (REGEX (STR (?seqEnt), ?accession))
    ?seqEnt a uniprot:Simple_Sequence ;
            rdf:value ?sequence .
#  }
}
```

## `format`

```javascript
({json, protein_items, order, desc, limit, offset, line_count})=>{
  let filter = {};
  let f = {};
  for (let acc of json) {
    filter[acc] = true;
    f[acc] = true;
  }
  let new_bindings = [];
  let t = 0;
  for (let i = 0; i < protein_items.results.bindings.length; i++) {
    if (filter[protein_items.results.bindings[i].accession.value]) {
      new_bindings.push(protein_items.results.bindings[i]);
      t++;
      delete(f[protein_items.results.bindings[i].accession.value]);
    }
  }
  console.log(json.length);
  console.log(t);
  console.log(Object.keys(f).join(","));

  if (line_count) {
    return {
      head: {
       link: [],
       vars: [
         "line_count",
       ]
      },
      results: {
        distinct: false,
        ordered: true,
        bindings: [
          {
            line_count: {
              type: "typed-literal",
              datatype: "http://www.w3.org/2001/XMLSchema#integer",
              value: new_bindings.length.toString()
            }
          }
        ]
      }
    };
  }
  if (order) {
    if (desc == "1") {
      new_bindings = new_bindings.sort((a, b) => {
        if( a[order].value < b[order].value ) return 1;
        if( a[order].value > b[order].value ) return -1;
        return 0;
      });
    } else {
      new_bindings = new_bindings.sort((a, b) => {
        if( a[order].value < b[order].value ) return -1;
        if( a[order].value > b[order].value ) return 1;
        return 0;
      });
    }
  }
  if (limit) {
    let limit_n = parseInt(limit);
    let offset_n = 0;
    if (offset) offset_n = parseInt(offset);
    new_bindings = new_bindings.splice(offset_n + 1, limit_n);
  }
  protein_items.results.bindings = new_bindings;
  return protein_items;
}
```
