# Develop: quantitative values w/ protein estimation (spectral counting) (for Stanza 'group_comp')

## Parameters

* `dataset1`
  * default: DS1631_1 DS1631_2 DS1631_3 DS1632_1 DS1632_2
* `dataset2`
  * default: DS1637_1 DS1638_1 DS1639_1

## `json1`

```javascript
async ({dataset1})=>{
  const options = {
    method: 'get',
    headers: {
      'Accept': 'application/json',
      'Content-Type': 'application/x-www-form-urlencoded'
    }
  };
  let api = "https://tools.jpostdb.org/proteins_by_datasets?accession=true&dataset=" + encodeURIComponent(dataset1.replace(/[, ]+/g, ","));
  try{
    const res = await fetch(api, options).then(res=>res.json());
    return res.proteins;
  }catch(error){
    console.log(error);
  }
};
```

## `json2`

```javascript
async ({dataset2})=>{
  const options = {
    method: 'get',
    headers: {
      'Accept': 'application/json',
      'Content-Type': 'application/x-www-form-urlencoded'
    }
  };

  let api = "https://tools.jpostdb.org/proteins_by_datasets?accession=true&dataset=" + encodeURIComponent(dataset2.replace(/[, ]+/g, ","));
  try{
    const res = await fetch(api, options).then(res=>res.json());
    return res.proteins;
  }catch(error){
    console.log(error);
  }
};
```

## `set_values`

```javascript
({dataset1, dataset2}) => {
  dataset = ":" + decodeURIComponent(dataset1 + " " + dataset2).split(/[ ,]/).join(" :");
  return {dataset: dataset };
};
```

## Endpoint

{{SPARQLIST_EP}}

## `spectral_counting`

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT ?dsid ?upid (COUNT (?psm) AS ?count)
WHERE {
  VALUES ?dataset { {{set_values.dataset}} }
  ?dataset dct:identifier ?dsid ;
           jpo:hasProtein ?protein .
  ?protein jpo:hasPeptideEvidence/jpo:hasPeptide/jpo:hasPsm ?psm ;
           rdfs:label ?upid .
  ?protein rdfs:label ?leading_accs .
}
```

## `to_table`

```javascript
({json1, json2, dataset1, dataset2, spectral_counting}) => {
  let filter1 = {};
  let filter2 = {};
  for (let acc of json1) {
    filter1[acc] = true;
  }
  for (let acc of json2) {
    filter2[acc] = true;
  }
  
  let protCount = {};
  let specCount = {};
  let specUpCount = {};
  let array1 = decodeURIComponent(dataset1).split(/[ ,]/);
  let array2 = decodeURIComponent(dataset2).split(/[ ,]/);
  let ds_array = array1.concat(array2);
  let r = [];
  r[0] = []; r[0][0] = "uniprot";
  let i = 0;
  for(i = 0; i < array1.length; i++){
    r[0][i + 1] = 0;
  }
  for(let j = i; j < array2.length + i; j++){
    r[0][j + 1] = 1;
  }
  let prot = {};
  let spectrum = spectral_counting.results.bindings;
  for(let i = 0; i < spectrum.length; i++){
    let up = spectrum[i].upid.value;
    if (!filter1[up] && !filter2[up]) continue;	
    let dsid = spectrum[i].dsid.value;
    let count = parseInt(spectrum[i].count.value); 
    prot[up] = 1;
    if (!protCount[dsid]) { 
      protCount[dsid] = 0; 
      specCount[dsid] = 0;
    }
    protCount[dsid]++;
    specCount[dsid] += count;
    specUpCount[up +"_" + dsid] = count; 
  }
  let array = Object.keys(prot);
  for (let i = 0; i < array.length; i++) {
    let a = [];
    a.push(array[i]);
    for (let j = 0;  j < ds_array.length;  j++) {
      let tmp = array[i] + "_" + ds_array[j];
      if (specUpCount[tmp]) {
      	let val = specUpCount[tmp] * protCount[ds_array[j]] / specCount[ds_array[j]];
      // 	let val = specUpCount[tmp] / specCount[dsid];
        a.push(val);
      }
      else a.push(0);
    }
    r.push(a);
  }
  
  /*
  //normalize
  let pseudocount = 1 / ds_array.length / ds_array.length;
  for(var i = 1; i < r.length; i++){
    let val_sum = 0;
    let zero = 0;
  	for(var j = 1; j < r[i].length; j++){
      val_sum += r[i][j];
      if(!r[i][j]) zero++;
    }
    let pseudo_sum = pseudocount  * zero;
  	for(var j = 1; j < r[i].length; j++){
      if(!r[i][j]) r[i][j] = pseudocount;
      else r[i][j] = (1 -  pseudo_sum) * r[i][j] / val_sum;
    }
  }
  console.log(pseudocount);
 */
  return r;
};
```
