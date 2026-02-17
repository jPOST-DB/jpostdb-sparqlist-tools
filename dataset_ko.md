# Discontinued

- ko list of datasets (slice) (for Stanza 'kegg_mapping_form')
  - Discontinued : UniProt から KO リンクがなくなったため
  - dataset_kegg_gene に移行
- UniProt の KO 経由で KEGG pathway にマッピングして、タンパク質に紐づくスペクトルの数で色分け

## Parameters

* `dataset` (Req.)
  * example: DS1631_1 DS1631_2 DS1632_1


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
SELECT DISTINCT # ?kegg
?ko ?uniprot_id ?spc_count ?seq
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
      rdfs:seeAlso ?ko .
 #     rdfs:seeAlso ?kegg .
  ?seqEnt a uniprot:Simple_Sequence ;
           rdf:value ?seq .
 #?kegg uniprot:database <http://purl.uniprot.org/database/KEGG> .
  ?ko uniprot:database <http://purl.uniprot.org/database/KO> .
}
```

## `return`

```javascript
async ({get_proteins}) => {
  var data = get_proteins.results.bindings;
  var prt_count = data.length;
  var r = [];
  var chk = [];
  var total = 0;
  var color = ["#6261ea", "#7e62ea", "#9b63ea", "#b764ea", "#d365ea", "#ea66e5", "#ea67ca", "#ea68b0", "#ea6995", "#ea6a7b", "#eb746b"];
  for(var i = 0; i < data.length; i++){
    var ko = data[i].ko.value.replace("http://purl.uniprot.org/ko/","");
    if(!chk[ko]) chk[ko] = 0;
    var tmp = (data[i].spc_count.value - 1) / (data[i].seq.value.length - 0);
    if(chk[ko] < tmp) chk[ko] = tmp;
  }
  var array = Object.keys(chk);
  var ko_count = array.length;
  var high_ko = array.sort(function(a,b){
        if( chk[a] > chk[b] ) return -1;
        if( chk[a] < chk[b] ) return 1;
        return 0;
  })[Math.round(array.length / 10)];
  var high = chk[high_ko];
  for(var i = 0; i < array.length; i++){        
    var tmp = Math.floor(chk[array[i]] / high * 10);
    if(tmp > 10) tmp = 10;
    r.push({ko: array[i], color: color[tmp]});        
  }
  var url = "http://www.genome.jp/kegg-bin/download_htext?htext=br08901.keg&format=json";
  var options = { method: 'GET'};
  var kegg_maps = await fetch(url, options).then(res=>res.json());
  return {ko_count: ko_count.toString().replace(/(\d)(?=(\d{3})+$)/g , '$1,'), 
  prt_count: prt_count.toString().replace(/(\d)(?=(\d{3})+$)/g , '$1,'), 
  list: r,
  kegg_maps: kegg_maps};
};
```