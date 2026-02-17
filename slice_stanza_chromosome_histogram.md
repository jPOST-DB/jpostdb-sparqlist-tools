# chromosome histogram for slice stanza (original: chromosome_histogram) (for Stanza 'chromosome_histogram')

## Parameters

* `dataset`
  * default: DS810_1 DS810_2 DS809_1 DS809_2

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

## `chromosome`

```sparql
PREFIX uniprot: <http://purl.uniprot.org/core/>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT ?chr (COUNT (DISTINCT ?protein) AS ?count) 
WHERE {
  ?protein a uniprot:Protein ;
           uniprot:organism <{{taxonomy.results.bindings.[0].tax.value}}> ;
           uniprot:proteome ?chr .
}
```

## `proteome`

```javascript
({chromosome})=>{
  var list = chromosome.results.bindings;
  var chk = [];
  var total = [];
  for(var i = 0; i < list.length; i++){
    var prot = list[i].chr.value.split("#")[0].split("/")[4];
    if(!chk[prot]){
      chk[prot] = 1;
      total[prot] = 0;
    }
    total[prot] += list[i].count.value - 0
  }
  var max = 0;
  var max_prot;
  var prot = Object.keys(total);
  for(var i = 0; i < prot.length; i++){
    console.log(total[prot[i]]);
    if(max < total[prot[i]]){
      max = total[prot[i]];
      max_prot = prot[i];
    }
  }
//  var database = "GeneID";
  var database = "";
  if(max_prot == "UP000005640") database = "?protein rdfs:seeAlso/uniprot:database <http://purl.uniprot.org/database/neXtProt> .";
  return {id: max_prot, database: database};
}
```

## `proteins`

```sparql
PREFIX uniprot: <http://purl.uniprot.org/core/>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT ?chr ?protein
WHERE {
  ?protein a uniprot:Protein ;
           uniprot:organism <{{taxonomy.results.bindings.[0].tax.value}}> ;
           uniprot:proteome ?chr .
  {{proteome.database}}
  FILTER( REGEX( STR(?chr), "/{{proteome.id}}#"))
}
ORDER BY ?chr
```

## `format`

```javascript
({json, proteins})=>{
  let filter = {};
  for (let acc of json) {
    filter[acc] = true;
  }
  
  let list = proteins.results.bindings;
  let dataset = {};
  let total = {};
  let label_hash = {};
  for (let row of list) {
    let label = row.chr.value.split("#")[1].replace(/%20/g, " ").replace("Chromosome", "Ch.");
    label_hash[label] = true;
    if (!total[label]) {
      total[label] = 0;
      dataset[label] = 0;
    }
    total[label]++;
    if (filter[row.protein.value.replace(/http:\/\/purl\.uniprot\.org\/uniprot\//, "")]) {
      dataset[label]++;
    }
  }
  
  let labels = Object.keys(label_hash).sort( (a, b) => {
    if (a.match(/^Ch\./) && b.match(/^Ch\./)) {
      let a_i = parseInt(a.replace(/Ch\. /, ""));
      let b_i = parseInt(b.replace(/Ch\. /, ""));
   	  if( a_i < b_i ) return -1;
      if( a_i > b_i ) return 1;
      return 0;
    } else if (a.match(/^Ch\./)) { 
      return -1;
    } else if (b.match(/^Ch\./)) {
      return 1;
    } else {
   	  if( a < b ) return -1;
      if( a > b ) return 1;
      return 0;
    }
  });
  
  let r = [];
  for(let label of labels){
    r.push({
      	label: label,
    	count: dataset[label],
        total: total[label]
    });
  }
  return r;
};	
```
