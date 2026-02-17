# Mutated peptide list  (for Stanza 'protein_browser')

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

## `get_mt_seq`

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX up: <http://purl.uniprot.org/uniprot/>
PREFIX faldo: <http://biohackathon.org/resource/faldo#>
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX : <http://rdf.jpostdb.org/entry/> 
SELECT ?begin ?end (COUNT (?begin) AS ?count) ?seq (SAMPLE(?pep_id) AS ?pep_id_sample) ?type ?shift ?ref ?mt ?mt_begin ?mt_end
WHERE {
{{filter.value}}
{{filter.code}}
  ?protein jpo:hasDatabaseSequence up:{{uniprot}} ;
           jpo:hasPeptideEvidence ?pepevi .
  ?pepevi jpo:hasPeptide ?pep ;
          faldo:location ?location .
  ?location faldo:begin/faldo:position ?begin ;
            faldo:end/faldo:position ?end .
  ?pep jpo:hasPsm ?psm ;
       dct:identifier ?pep_id ;
       jpo:hasSequence [ a obo:MS_1001344 ;
                         rdf:value ?seq ] .
#  OPTIONAL { ?pep a ?uniq .
#    FILTER( ?uniq = jpo:UniquePeptideAtMsLevel) }
  ?psm jpo:hasMutation [ a ?type ;
                         jpo:shift ?shift ;
                         jpo:referenceResidues ?ref ;
                         jpo:mutatedResidues ?mt ;
                       faldo:location/faldo:begin/faldo:position ?mt_begin ;
                       faldo:location/faldo:end/faldo:position ?mt_end ] .
}
ORDER BY ?begin ?end
```

## `return`

```javascript
({pep_len, get_mt_seq}) => {
  var list = get_mt_seq.results.bindings;
  psm_position = [];
  var len = 0;
  if(pep_len) len = pep_len - 0;
  var max_count = 0;
  for(var i = 0; i < list.length; i++){
    if(list[i].seq.value.length < len) continue;
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
    var label = list[i].seq.value;
    if(list[i].type.value.replace(/.+#/, "") == "InFrameInsertion"){
  //    label = label.substr(0, Number(list[i].mt_begin.value) - 1) + label.substr(Number(list[i].mt_begin.value), label.length - Number(list[i].mt_end.value) );
    }else if(list[i].type.value.replace(/.+#/, "") == "InFrameDeletion"){
	  label = label.substr(0, Number(list[i].mt_begin.value) - 1) + "-" + label.substr(Number(list[i].mt_begin.value) - 1, label.length - Number(list[i].mt_end.value) + 1);
    }
    psm_position.push({
      pep_id: list[i].pep_id_sample.value, 
      begin: list[i].begin.value, 
      end: list[i].end.value,
      shift: list[i].shift.value,
      mt_begin: list[i].mt_begin.value,
      mt_end: list[i].mt_end.value,
      ref_aa: list[i].ref.value,
      mt_aa: list[i].mt.value,
      type: list[i].type.value.replace(/.+#/, ""),
    //  uniq: uniq,
      color: color, 
      y: y_axis, 
      count: list[i].count.value,    
      sequence: list[i].seq.value,
      label: label
    });
  }
  return psm_position;
};
```

