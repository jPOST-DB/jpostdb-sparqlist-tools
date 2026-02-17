# PSM position list (for Stanza 'protein_browser')

## Parameters

* `uniprot` Uniprot ID (Req.)
  * default: O75143
* `dataset` (Opt.)
  * example: DS206_1 DS207_1 DS208_1
* `pep_len` (Opt.)


## `filter`

```javascript
({dataset, uniq}) => {
  var value = "";
  var code = "";
  if(dataset){
    value += "  VALUES ?dataset { :" + decodeURIComponent(dataset).split(/[ ,]/).join(" :") + " }\n";
    code += "  ?dataset jpo:hasProtein ?protein .\n";
  }
  return {value: value, code: code };
};
```

## Endpoint

{{SPARQLIST_EP}}

## `get_psms`

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX up: <http://purl.uniprot.org/uniprot/>
PREFIX faldo: <http://biohackathon.org/resource/faldo#>
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX : <http://rdf.jpostdb.org/entry/> 
SELECT ?begin ?end (COUNT (?begin) AS ?count) ?seq (SAMPLE(?pep_id) AS ?pep_id_sample) ?mutation ?tax #?uniq
WHERE {
{{filter.value}}
{{filter.code}}
  ?protein jpo:hasDatabaseSequence up:{{uniprot}} ;
           ^jpo:hasProtein / jpo:hasProfile / jpo:hasSample / jpo:species ?tax ;
           jpo:hasPeptideEvidence ?pepevi .
  ?pepevi jpo:hasPeptide ?pep ;
          faldo:location ?location .
  ?location faldo:begin/faldo:position ?begin ;
            faldo:end/faldo:position ?end .
  ?pep jpo:hasPsm ?psm ;
       dct:identifier ?pep_id ;
       jpo:hasSequence [ a obo:MS_1001344 ;
                         rdf:value ?seq ] .
 # OPTIONAL { ?pep a ?uniq .
 #   FILTER( ?uniq = jpo:UniquePeptideAtMsLevel) }
  OPTIONAL { ?psm jpo:hasMutation ?mutation }
}
ORDER BY ?begin ?end
```

## `return`

```javascript
({pep_len, get_psms}) => {
  var list = get_psms.results.bindings;
  psm_position = [];
  var len = 0;
  if(pep_len) len = pep_len - 0;
  var max_count = 0;
  for(var i = 0; i < list.length; i++){
    if(list[i].seq.value.length < len) continue;
    if(list[i].mutation) continue;
    var tmp = parseInt(list[i].count.value);
    if(max_count < tmp) max_count = tmp;
  }
  var toHex = function(num){
    var hex = num.toString(16);
    if(hex.length == 1) hex = "0" + hex;
    return hex;
  };
  var count2color = function(count, max){      // red ~ gray
    var tmp = ~~((count-1) / (max-1) * 255 / 2 + 128); // 128 ~ 255
    if(tmp <= 128) tmp = 128;
    if(tmp >= 255) tmp = 255;
    var r_color = toHex(tmp);
    var gb_color = toHex(255 - tmp);
    return "#" + r_color + gb_color + gb_color;
  };
  var y_axis = 1;
  var yend = [];
  var max_y = 0;
  for(var i = 0; i < list.length; i++){
    if(list[i].seq.value.length < len) continue;
   	if(list[i].mutation) continue;
    var color = count2color(list[i].count.value, max_count);
    var f = 0;
    for(var j = 1; j < yend.length; j++){
      if(yend[j] < list[i].begin.value - 1){
        y_axis = j;
        f = 1;
        break;
      }	
    }
    if(f == 0){ max_y++; y_axis = max_y; }
    yend[y_axis] = list[i].end.value;
    var uniq = 0;
    if(list[i].uniq) uniq = 1;
    psm_position.push({
      pep_id: list[i].pep_id_sample.value, 
      begin: list[i].begin.value, 
      end: list[i].end.value, 
      uniq: uniq,
      color: color, 
      y: y_axis, 
      count: list[i].count.value,    
      label: list[i].seq.value + ": " + list[i].count.value,
      seq: list[i].seq.value,
      tax: list[i].tax.value.replace("http://identifiers.org/taxonomy/","")
    });
  }
  return psm_position;
};
```

