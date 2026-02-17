# kegg gene list of datasets (slice) (for Stanza 'kegg_mapping_form')

- UniProt の KEGG genes 経由で KEGG pathway にマッピングして、タンパク質に紐づくスペクトルの数で色分け

## Parameters

* `dataset` (Rea.)
  * example: DS1631_1 DS1631_2 DS1631_1


## `filter`

```javascript
({dataset})=>{
      return {code: "VALUES ?dataset { :" + decodeURIComponent(dataset).split(/[ ,]/).join(" :") + " }" };
};
```

## Endpoint

{{SPARQLIST_EP}}

## `get_proteins`

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX uniprot: <http://purl.uniprot.org/core/>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT DISTINCT ?kegg ?uniprot_id ?spc_count ?seq
WHERE {
  {
    SELECT DISTINCT ?uniprot_id ?up (COUNT (DISTINCT ?spc) AS ?spc_count)
    WHERE {
{{filter.code}}
      ?dataset jpo:hasProtein ?protein .
      ?protein a obo:MS_1002401 ;
               rdfs:label ?uniprot_id ;
               jpo:hasPeptideEvidence/jpo:hasPeptide/jpo:hasPsm/jpo:hasSpectrum ?spc ;
               jpo:hasDatabaseSequence ?up .
      }
    }
  ?up uniprot:sequence ?seqEnt ;
      rdfs:seeAlso ?kegg .
  ?seqEnt a uniprot:Simple_Sequence ;
           rdf:value ?seq .
  ?kegg uniprot:database <http://purl.uniprot.org/database/KEGG> .
}
```

## `return`

```javascript
async ({get_proteins}) => {
  let data = get_proteins.results.bindings;
  let prt_count = data.length;
  let r = [];
  let chk = [];
  let total = 0;
  let color = ["#6261ea", "#7e62ea", "#9b63ea", "#b764ea", "#d365ea", "#ea66e5", "#ea67ca", "#ea68b0", "#ea6995", "#ea6a7b", "#eb746b"];
  let org = false;
  for(var i = 0; i < data.length; i++){
 	if (!org) org = data[i].kegg.value.replace("http://purl.uniprot.org/kegg/","").split(":")[0];
    let kegg = data[i].kegg.value.replace("http://purl.uniprot.org/kegg/" + org + ":", "");
    if(!chk[kegg]) chk[kegg] = 0;
    let tmp = (data[i].spc_count.value - 1) / (data[i].seq.value.length - 0);
    if(chk[kegg] < tmp) chk[kegg] = tmp;
  }
  let array = Object.keys(chk);
  let high_kegg = array.sort(function(a,b){
        if( chk[a] > chk[b] ) return -1;
        if( chk[a] < chk[b] ) return 1;
        return 0;
  })[Math.round(array.length / 10)];
  let high = chk[high_kegg];
  for(var i = 0; i < array.length; i++){        
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