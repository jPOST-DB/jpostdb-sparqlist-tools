# Protein PTM linkage (for Stanza 'protein_browser')

## Parameters

* `uniprot` (Req.)
  * default: Q9NYF8
* `unimod` (Req.)
  * default: 21 
* `dataset` (Opt.)
  * example: DS205_1 DS206_1


## `set_filter`

```javascript
({dataset}) => {
  var value = "";
  var code = "";
  if(dataset){
      value += "  VALUES ?dataset { :" + decodeURIComponent(dataset).split(/[ ,]/).join(" :") + " }\n";
      code += "  ?dataset jpo:hasProtein ?protein .\n";
  }
  return { value: value, code: code };
};
```

## Endpoint

{{SPARQLIST_EP}}

## `ptm_position`

```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX up: <http://purl.uniprot.org/uniprot/>
PREFIX faldo: <http://biohackathon.org/resource/faldo#>
PREFIX unimod: <http://www.unimod.org/obo/unimod.obo#UNIMOD_>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT ?begin ?end (COUNT (?psm) AS ?count)
WHERE {
  VALUES ?mods { unimod:{{unimod}} }
{{set_filter.value}}
  ?mods rdfs:label ?mod_label .
{{set_filter.code}}
  ?protein jpo:hasDatabaseSequence up:{{uniprot}} ;
           jpo:hasPeptideEvidence ?pepevi .
  ?pepevi faldo:location [faldo:begin/faldo:position ?begin ;
                                      faldo:end/faldo:position ?end ] ;
          jpo:hasPeptide/jpo:hasPsm ?psm .
}
ORDER BY ?begin ?end
```

## Endpoint

{{SPARQLIST_EP}}

## `ptm_linkage`

```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX up: <http://purl.uniprot.org/uniprot/>
PREFIX faldo: <http://biohackathon.org/resource/faldo#>
PREFIX unimod: <http://www.unimod.org/obo/unimod.obo#UNIMOD_>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT ?pos_a ?pos_b ?site_a ?site_b ?psm (COUNT (DISTINCT ?pos_a) AS ?count)
WHERE {
  VALUES ?mods { unimod:{{unimod}} }
{{set_filter.value}}
  ?mods rdfs:label ?mod_label .
{{set_filter.code}}
  ?protein jpo:hasDatabaseSequence up:{{uniprot}} ;
           jpo:hasPeptideEvidence ?pepevi .
  ?pepevi faldo:location [faldo:begin/faldo:position ?begin ] ;
          jpo:hasPeptide/jpo:hasPsm ?psm .
  ?psm jpo:hasModification [a ?mods ;
                              faldo:location/faldo:position ?position_a ;
                              jpo:modificationSite ?site_a ] .
  ?psm jpo:hasModification [a ?mods ;
                              faldo:location/faldo:position ?position_b ;
                              jpo:modificationSite ?site_b ] .
       FILTER (?position_a < ?position_b)
  BIND (?position_a + ?begin -1 AS ?pos_a)
  BIND (?position_b + ?begin -1 AS ?pos_b)
}
ORDER BY ?pos_a ?pos_b
```

## `chk_linkage`

```javascript
({ptm_position, ptm_linkage}) => {
  var position = ptm_position.results.bindings;
  var linkage = ptm_linkage.results.bindings;
  var r = [];
  var pos = {};
  for(var i = 0; i < linkage.length; i++){
    var obj = {
      pos_a: linkage[i].pos_a.value - 0,
      pos_b: linkage[i].pos_b.value - 0,
      countLink: linkage[i].count.value - 0,
      countBase: 0
    }
    pos[obj.pos_a] = linkage[i].site_a.value;
    pos[obj.pos_b] = linkage[i].site_b.value;	
    for(var j = 0; j < position.length; j++){
      if(position[j].begin.value <= obj.pos_a && obj.pos_b <= position[j].end.value){
        obj.countBase += position[j].count.value - 0;
      }
    }
    r.push(obj)
  }
  var r2 = [];
  var array = Object.keys(pos);
  for(var i = 0; i < array.length; i++){
    var site = pos[array[i]];
    var color = "#888888";
    if(site == "S") color = "#ffc31e";
    else if(site == "T") color = "#ff831e";
    else if(site == "Y") color ="#ff5a1e";
    var obj = {
      position: array[i],
      color: color
    }
    r2.push(obj);
  }
  return {link: r, site: r2};
};
```

