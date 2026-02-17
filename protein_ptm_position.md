# protein PTM position (for Stanza 'protein_browser')

## Parameters

* `uniprot` Uniprot ID (Req.)
  * default: Q9NYF8
* `unimod` Unimod ID (Req.)
  * default: 21
* `dataset` (Opt.)
  * example: DS205_1 DS206_1


## `filter`

```javascript
({dataset}) => {
  var value = "";
  var code = "";
  if(dataset){
      value += "  VALUES ?dataset { :" + decodeURIComponent(dataset).split(/[ ,]/).join(" :") + " }\n";
      code += "  ?dataset jpo:hasProtein ?protein .\n";
  }
  return {value: value, code: code};
};
```

## Endpoint

{{SPARQLIST_EP}}

## `get_psms`

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX up: <http://purl.uniprot.org/uniprot/>
PREFIX faldo: <http://biohackathon.org/resource/faldo#>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT ?seq ?begin ?end (COUNT (DISTINCT ?psm) AS ?count)
WHERE {
{{filter.value}}
{{filter.code}}
  ?protein jpo:hasDatabaseSequence up:{{uniprot}} ;
           jpo:hasPeptideEvidence ?pepevi .
  ?pepevi faldo:location/faldo:begin/faldo:position ?begin ;
          faldo:location/faldo:end/faldo:position ?end ;
          jpo:hasPeptide ?pep .
  ?pep jpo:hasSequence [ a obo:MS_1001344 ;
                           rdf:value ?seq ] ;
       jpo:hasPsm ?psm .
}
ORDER BY ?begin ?end
```

## Endpoint

{{SPARQLIST_EP}}

## `get_ptms`

```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX up: <http://purl.uniprot.org/uniprot/>
PREFIX faldo: <http://biohackathon.org/resource/faldo#>
PREFIX unimod: <http://www.unimod.org/obo/unimod.obo#>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT ?mods ?mod_label ?site ?pos (COUNT (DISTINCT ?psm) AS ?count)
WHERE {
  VALUES ?mods { unimod:UNIMOD_{{unimod}} }
{{filter.value}}
  #?mods <http://purl.obolibrary.org/obo/IAO_0000115> ?mod_label .
  ?mods rdfs:label ?mod_label .
{{filter.code}}
  ?protein jpo:hasDatabaseSequence up:{{uniprot}} ;
           jpo:hasPeptideEvidence ?pepevi .
  ?pepevi faldo:location [faldo:begin/faldo:position ?begin ] ;
          jpo:hasPeptide/jpo:hasPsm ?psm .
  ?psm jpo:hasModification ?blank .
  ?blank a ?mods ;
         faldo:location/faldo:position ?position ;
         jpo:modificationSite ?site .
  BIND (?position + ?begin -1 AS ?pos)
}
ORDER BY ?pos
```

## `return`

```javascript
({unimod, get_psms, get_ptms}) => {
  var list = get_psms.results.bindings;
  var posFreq = [];
  var ptm = [];
  for(var i = 0; i < list.length; i++){
    for(var j = parseInt(list[i].begin.value); j <= parseInt(list[i].end.value); j++){
      if(!posFreq[j]) posFreq[j] = 0;
      posFreq[j] += list[i].count.value - 0;
    }
  }
  var list = get_ptms.results.bindings;
  var max = 0;
  for(var i = 0; i < list.length; i++){
    var pos = list[i].pos.value - 0;
    var count = list[i].count.value - 0;
    var site = list[i].site.value;
    var color = "#936a76";
    if(unimod == 21){
    	if(site == "S") color = "#ffc31e";
        else if(site == "T") color = "#ff831e";
        else if(site == "Y") color ="#ff5a1e";
    }
    ptm.push({
      position: pos,
      position_count: posFreq[pos],
      count: count,
      site: site,
      color: color
    });
    if(max < count) max = count;
  }
  return {name: get_ptms.results.bindings[0].mod_label.value, mod_id: unimod, max_count: max, list: ptm};
};
```