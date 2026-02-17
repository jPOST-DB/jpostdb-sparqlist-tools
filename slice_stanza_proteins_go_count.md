# Develop: protein go count for slice stanza w/ protein estimation (originsl: proteins_go_count) (for Stanza' go_count')

## Parameters

* `dataset`
  * default: DS1631_1 DS1631_2 DS1637_1 DS1637_2
* `category`
  * default: biological_process
  * example: biological_process, molecular_function, cellular_component


## `json`

```javascript
async ({dataset})=>{
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

## `go`

```javascript
({category}) => {
  var go = "GO_0008150";
  if(category == "biological_process"){
    go = "GO_0008150";
  }else if(category == "molecular_function"){
    go = "GO_0003674";
  }else if(category == "cellular_component"){
    go = "GO_0005575";
  }
  return go;
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

## `protein_go`

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX uniprot: <http://purl.uniprot.org/core/>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX jpost: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT DISTINCT ?label ?accession
WHERE {
  ?protein rdfs:label ?accession .
  ?protein jpost:hasDatabaseSequence ?up .
  ?up uniprot:organism <{{taxonomy.results.bindings.[0].tax.value}}> ;
      uniprot:classifiedWith ?go .
  FILTER (REGEX (?go, obo:GO_))
  ?go rdfs:subClassOf* ?level .
  ?level rdfs:subClassOf obo:{{go}} .
  ?level rdfs:label ?label .
  FILTER (DATATYPE(?label) = xsd:string)
}
```

## `format`

```javascript
({json, protein_go})=>{
  let filter = {};
  for (let acc of json) {
    filter[acc] = true;
  }
  let go = {};
  for (let i = 0; i < protein_go.results.bindings.length; i++) {
    if (filter[protein_go.results.bindings[i].accession.value]) {
      if (!go[protein_go.results.bindings[i].label.value]) go[protein_go.results.bindings[i].label.value] = 0;
      go[protein_go.results.bindings[i].label.value]++;
    }
  }
  let sort = Object.keys(go).sort((a, b) => {
    if( go[a] < go[b] ) return 1;
    if( go[a] > go[b] ) return -1;
    return 0;
  });

  let res = [];
  for (let key of sort) {
    res.push( {label: key, id: key, count: go[key].toString()} );
  }
  return res;
}
```
