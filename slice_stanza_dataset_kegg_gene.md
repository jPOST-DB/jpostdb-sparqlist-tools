# kegg genes list of datasets for slice stanza (original: dataset_kegg_gene) (for Stanza 'kegg_mapping_form')

## Parameters

* `dataset`
  * default: DS810_1 DS810_2 DS811_1 DS811_2

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

## `proteins`

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX uniprot: <http://purl.uniprot.org/core/>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT ?kegg ?protein ?seq (COUNT (DISTINCT ?spc) AS ?spc_count)
WHERE {
  ?protein ^jpo:hasDatabaseSequence/jpo:hasPeptideEvidence/jpo:hasPeptide/jpo:hasPsm/jpo:hasSpectrum ?spc ;
           a uniprot:Protein ;
           uniprot:organism <{{taxonomy.results.bindings.[0].tax.value}}> ;
  	       uniprot:sequence ?seqEnt ;
           rdfs:seeAlso ?kegg .
  ?seqEnt a uniprot:Simple_Sequence ;
           rdf:value ?seq .
  ?kegg uniprot:database <http://purl.uniprot.org/database/KEGG> .
}
```

## `return`

```javascript
async ({json, proteins}) => {
  let filter = {};
  for (let acc of json) {
    filter[acc] = true;
  }
  let data = proteins.results.bindings;
  let prt_count = 0;
  let r = [];
  let chk = [];
  let total = 0;
  let color = ["#6261ea", "#7e62ea", "#9b63ea", "#b764ea", "#d365ea", "#ea66e5", "#ea67ca", "#ea68b0", "#ea6995", "#ea6a7b", "#eb746b"];
  let org = false;
  for (let row of data) {
    if (filter[row.protein.value.replace("http://purl.uniprot.org/uniprot/","")]) {
      prt_count++;
   	  if (!org) org = row.kegg.value.replace("http://purl.uniprot.org/kegg/","").split(":")[0];
      let kegg = row.kegg.value.replace("http://purl.uniprot.org/kegg/" + org + ":", "");
      if (!chk[kegg]) chk[kegg] = 0;
      let tmp = (row.spc_count.value - 1) / (row.seq.value.length - 0);
      if (chk[kegg] < tmp) chk[kegg] = tmp;
    }
  }
  let array = Object.keys(chk);
  let high_kegg = array.sort(function(a,b){
        if( chk[a] > chk[b] ) return -1;
        if( chk[a] < chk[b] ) return 1;
        return 0;
  })[Math.round(array.length / 10)];
  let high = chk[high_kegg];
  for (var i = 0; i < array.length; i++) {        
    var tmp = Math.floor(chk[array[i]] / high * 10);
    if(tmp > 10) tmp = 10;
    r.push({kegg: array[i], color: color[tmp]});        
  }
  var url = "http://www.genome.jp/kegg-bin/download_htext?htext=br08901.keg&format=json";
  var options = { method: 'GET'};
  var kegg_maps = await fetch(url, options).then(res=>res.json());
  return {org: org, 
  prt_count: prt_count.toString().replace(/(\d)(?=(\d{3})+$)/g , '$1,'), 
  list: r,
  kegg_maps: kegg_maps};
};
```
