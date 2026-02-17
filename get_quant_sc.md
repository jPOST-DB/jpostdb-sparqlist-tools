# quantitative values (spectral counting) (for Stanza 'slice_comparison')

- タンパク質毎の発現量（spectrul count の割合）を取得
  - qunst_test API (perl) から利用
  
## Parameters

* `dataset1`
  * default: DS1631_1 DS1632_1 DS1633_1 DS1634_1 DS1631_2 DS1631_3
* `dataset2`
  * default: DS1637_1 DS1635_1 DS1636_1 DS1635_2


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
  # is leading protein in any dataset
  ?dataset jpo:hasProtein ?leading .
  ?leading a obo:MS_1002401 ;
           rdfs:label ?leading_accs .
  ?protein rdfs:label ?leading_accs .
}
```

## `to_table`

```javascript
({dataset1, dataset2, spectral_counting}) => {
  var protCount = {};
  var specCount = {};
  var specUpCount = {};
  var array1 = decodeURIComponent(dataset1).split(/[ ,]/);
  var array2 = decodeURIComponent(dataset2).split(/[ ,]/);
  var ds_array = array1.concat(array2);
  var kara = {};
  var r = [];
  r[0] = []; r[0][0] = "uniprot";
  var i = 0;
  for(i = 0; i < array1.length; i++){
    r[0][i + 1] = 0;
    kara[array1[i]] = 0;
  }
  for(var j = i; j < array2.length + i; j++){
    r[0][j + 1] = 1;
    kara[array2[j - i]] = 0;
  }
  var prot = {};
  var spectrum = spectral_counting.results.bindings;
  for(var i = 0; i < spectrum.length; i++){
    var up = spectrum[i].upid.value;	
    var dsid = spectrum[i].dsid.value;
    var count = spectrum[i].count.value - 0; 
    prot[up] = 1;
    if(!protCount[dsid]) { 
      protCount[dsid] = 0; 
      specCount[dsid] = 0;
    }
    protCount[dsid]++;
    specCount[dsid] += count;
    specUpCount[up +"_" + dsid] = count; 
  }
  var array = Object.keys(prot);
  for(var i = 0; i < array.length; i++){
    var a = [];
    a.push(array[i]);
    for(var j = 0;  j < ds_array.length;  j++){
      var tmp = array[i] + "_" + ds_array[j];
      if(specUpCount[tmp]){
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
