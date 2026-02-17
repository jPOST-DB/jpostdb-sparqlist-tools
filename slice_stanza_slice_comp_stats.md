# Develop: statistics for slice comparison w/ protein estimation (for Stanza 'slice_comp_stats')

## Parameters


* `dataset1`
  * default: DS1631_1 DS1631_2 DS1631_3
* `dataset2`
  * default: DS1637_1 DS1637_2 DS1637_3
* `slice1` (Opt.)
* `slice2` (Opt.)

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

## `filter`

```javascript
({dataset1, dataset2})=>{
  var code1 = "";
  var code2 = "";
  if(dataset1) code1 += "  VALUES ?dataset { :" + decodeURIComponent(dataset1).split(/[ ,]+/).join(" :") + " }\n";
  if(dataset2) code2 += "  VALUES ?dataset { :" + decodeURIComponent(dataset2).split(/[ ,]+/).join(" :") + " }\n";
  return {code1: code1, code2: code2}
};
```

## Endpoint

{{SPARQLIST_EP}}

## `peptide1`

```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT DISTINCT ?upid ?peptide
WHERE {
  {{filter.code1}}
  ?dataset jpo:hasProtein ?protein .
  ?protein rdfs:label ?upid ;
           jpo:hasPeptideEvidence/jpo:hasPeptide/jpo:hasSequence [ a obo:MS_1001344 ;
                rdf:value ?peptide ] .
}
```

## `peptide2`

```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT DISTINCT ?upid ?peptide
WHERE {
  {{filter.code2}}
  ?dataset jpo:hasProtein ?protein .
  ?protein rdfs:label ?upid ;
           jpo:hasPeptideEvidence/jpo:hasPeptide/jpo:hasSequence [ a obo:MS_1001344 ;
                rdf:value ?peptide ] .
}
```


## `return`

```javascript
({json1, json2, dataset1, dataset2, slice1, slice2, peptide1, peptide2})=>{
  let filter1 = {};
  let filter2 = {};
  let prot_share = 0;
  for (let acc of json1) {
    filter1[acc] = true;
  }
  for (let acc of json2) {
    filter2[acc] = true;
    if (filter1[acc]) prot_share++;
  }
  let prot1_uniq = json1.length - prot_share;
  let prot2_uniq = json2.length - prot_share;
  
  let ds1 = decodeURIComponent(dataset1).split(/[ ,]+/);
  let ds2 = decodeURIComponent(dataset2).split(/[ ,]+/);
  let ds_share = 0;
  for (let i = 0; i < ds1.length; i++) {
    if(dataset2.match(ds1[i])) ds_share++;
  }
  let s1 = "slice: 1";
  let s2 = "slice: 2";
  if (slice1) s1 = "slice: " + slice1;
  if (slice2) s2 = "slice: " + slice2;
    
  let pep_filter1 = {};
  let pep_filter2 = {};
  let pep_share = 0;
  for (let list of peptide1.results.bindings) {
    if (!filter1[list.upid.value] || pep_filter1[list.peptide.value]) continue;
    pep_filter1[list.peptide.value] = true;
  }
  for (let list of peptide2.results.bindings) {
    if (!filter2[list.upid.value] || pep_filter2[list.peptide.value]) continue;
    pep_filter2[list.peptide.value] = true;
    if (pep_filter1[list.peptide.value]) pep_share++;
  }
  let pep1_uniq = Object.keys(pep_filter1).length - pep_share;
  let pep2_uniq = Object.keys(pep_filter2).length - pep_share;
  return {
    slice1: s1,
    slice2: s2,
    prot1: json1.length.toString().replace(/(\d)(?=(\d{3})+$)/g , '$1,'),
    prot2: json2.length.toString().replace(/(\d)(?=(\d{3})+$)/g , '$1,'), 
    prot_share: prot_share.toString().replace(/(\d)(?=(\d{3})+$)/g , '$1,'), 
    prot1_uniq: prot1_uniq.toString().replace(/(\d)(?=(\d{3})+$)/g , '$1,'),
    prot2_uniq: prot2_uniq.toString().replace(/(\d)(?=(\d{3})+$)/g , '$1,'), 
    pep1: Object.keys(pep_filter1).length.toString().replace(/(\d)(?=(\d{3})+$)/g , '$1,'),
    pep2: Object.keys(pep_filter2).length.toString().replace(/(\d)(?=(\d{3})+$)/g , '$1,'), 
    pep_share: pep_share.toString().replace(/(\d)(?=(\d{3})+$)/g , '$1,'), 
    pep1_uniq: pep1_uniq.toString().replace(/(\d)(?=(\d{3})+$)/g , '$1,'),
    pep2_uniq: pep2_uniq.toString().replace(/(\d)(?=(\d{3})+$)/g , '$1,'),
    ds1: ds1.length,
    ds2: ds2.length,
    ds_share: ds_share,
    ds1_uniq: ds1.length - ds_share,
    ds2_uniq: ds2.length - ds_share
         }
};
```
