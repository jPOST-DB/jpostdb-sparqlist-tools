# Deverop ?
* exon intron data form togogenome.org (for Stanza 'proteoform_brpwser')

## Parameters

* `uniprot`
  * default: Q15149
* `isoform`
  * default: 0


## `filter`

```javascript
({isoform})=>{
  var code = "";
  if(isoform == 0) code = "?iso a uniprot:Simple_Sequence .";	
  return {code: code};
};
```

## Endpoint

{{SPARQLIST_EP}}

## `get_trans`

```sparql
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX uniprot: <http://purl.uniprot.org/core/>
PREFIX up: <http://purl.uniprot.org/uniprot/>
SELECT DISTINCT ?enst ?type ?iso ?seq
FROM <http://jpost.org/graph/uniprot>
WHERE {
  up:{{uniprot}} rdfs:seeAlso ?enst ;
            uniprot:sequence ?iso .
  ?iso rdf:value ?seq ;
       a ?type .
  {{filter.code}}
  ?enst a uniprot:Transcript_Resource .
  ?enst rdfs:seeAlso ?iso .
}
```

## `enst_list`

```javascript
({get_trans})=>{
  var code = "";
  var list = get_trans.results.bindings;
  for(var i = 0; i < list.length; i++){
    code += " <" + list[i].enst.value + ">";
  }
  return {code: code};
};
```

## Endpoint

https://integbio.jp/rdf/ebi/sparql

## `get_location`

```sparql
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX sio: <http://semanticscience.org/resource/>
PREFIX faldo: <http://biohackathon.org/resource/faldo#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
SELECT ?enst ?exon ?region
WHERE {
  VALUES ?enst { {{enst_list.code}} }
  ?enst sio:SIO_000974 ?exon .
  ?exon sio:SIO_000628	 ?node .
  ?node faldo:location ?location .
  ?location rdfs:label ?region . 
}
```

## `return`

```javascript
({get_trans, get_location})=>{
  var trans = get_trans.results.bindings;
  var loc = get_location.results.bindings;
  var r = [];
  var chk = {};
  for(var i = 0; i < trans.length; i++){
    var uniprot = trans[i].iso.value.replace("http://purl.uniprot.org/isoforms/", "");
    if(trans[i].type.value.match("Simple_Sequence")) uniprot = uniprot.replace(/-\d+$/, "");
    if(chk[uniprot]) continue;
    var enst = trans[i].enst.value;
    var location = [];
    for(var j = 0; j < loc.length; j++){	
      if(loc[j].enst.value == enst){
        var num = loc[j].exon.value.match(/_(\d+)$/)[1];
        location[num - 1] = loc[j].region.value;
      }
    }
    var comp = 0;
    var s1 = "";
    var s2 = [];
    var s3 = "";
    if(location[0].match(/\-1$/)){
      comp = 1;	
      s1 += "complement(";
      s3 += ")";
    }
    if(location.length > 1){
      s1 += "join("
      s3 += ")";
    }
    if(comp){
      for(var j = location.length; j > 0; j--){
        var pos = location[j - 1].split(":");
        s2.push(pos[1].replace("-", ".."));
      }
    }else{
      for(var j = 0; j < location.length; j++){
        var pos = location[j].split(":");	
        s2.push(pos[1].replsce("-", ".."));      
      }
    }
    var obj = {
      uniprot: uniprot,
      isoform: [	
        {	
          sequence: trans[i].seq.value,
          location: s1 + s2.join(",") + s3
        }
      ]	
    };
    r.push(obj);
    chk[uniprot] = 1;
  }
  return r;
};
```
