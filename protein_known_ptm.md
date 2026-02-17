# protein known PTM from UniProt (for Stanza 'protein_browser')

## Parameters

* `uniprot` Uniprot ID (Req.)
  * default: Q9NYF8


## Endpoint

{{SPARQLIST_EP}}

## `get_seq`

```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX up: <http://purl.uniprot.org/uniprot/>
PREFIX uniprot: <http://purl.uniprot.org/core/>
SELECT ?seq
WHERE {
 up:{{uniprot}} uniprot:sequence ?seqEnt .
  FILTER (REGEX (STR (?seqEnt), "{{uniprot}}"))
  ?seqEnt a uniprot:Simple_Sequence ;
          rdf:value ?seq .
}
```

## Endpoint

{{SPARQLIST_EP}}

## `get_uniprot_ptm`

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX up: <http://purl.uniprot.org/uniprot/>
PREFIX faldo: <http://biohackathon.org/resource/faldo#>
PREFIX uniprot: <http://purl.uniprot.org/core/>
SELECT DISTINCT ?label ?position ?an_type
WHERE {
  VALUES ?an_type { uniprot:Modified_Residue_Annotation uniprot:Glycosylation_Annotation }
  up:{{uniprot}} uniprot:annotation ?an .
  ?an a	?an_type ; 
      rdfs:comment ?label ;
      uniprot:range/faldo:begin/faldo:position ?position .
}
ORDER BY ?position
```

## Endpoint

{{SPARQLIST_EP}}

## `get_substitution`

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX up: <http://purl.uniprot.org/uniprot/>
PREFIX faldo: <http://biohackathon.org/resource/faldo#>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX uniprot: <http://purl.uniprot.org/core/>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT DISTINCT ?substitution ?position
WHERE {
  up:{{uniprot}} uniprot:annotation ?an .
  ?an a	uniprot:Natural_Variant_Annotation ;
      uniprot:substitution ?substitution ;
      uniprot:range/faldo:begin/faldo:position ?position .
}
ORDER BY ?position
```

## `return`

```javascript
({get_seq, get_uniprot_ptm, get_substitution}) => {
  var r = [];
  var seq = get_seq.results.bindings[0].seq.value;
  var list = get_uniprot_ptm.results.bindings;
  for(var i = 0; i < list.length; i++){
    var obj = {};
    obj.label = list[i].label.value;
    obj.position = list[i].position.value - 0;
    obj.site = seq.slice(obj.position - 1, obj.position);
    if(list[i].label.value.match(/^Phospho/)){
      obj.y = 1;
      obj.symbol = "p";
      if(list[i].label.value.match(/^Phosphoserin/)) obj.color = "#ffc31e";
      else if(list[i].label.value.match(/^Phosphothreonine/)) obj.color = "#ff831e";
      else if(list[i].label.value.match(/^Phosphotyrosine/)) obj.color = "#ff5a1e";
    }else{
      obj.y = 2;
      if(list[i].label.value.match(/[- ]acetyl/)){ obj.symbol = "a"; obj.color = "#00ce1e"; }
      else if(list[i].label.value.match(/[- ]methyl/) 
         || list[i].label.value.match(/[- ]dimethyl/) 
         || list[i].label.value.match(/[- ]trimethyl/)){ obj.symbol = "m"; obj.color = "#00ce7f"; }
      else if(list[i].label.value.match(/[- ]carboxyl/)){ obj.symbol = "c"; obj.color = "#00c7ce"; }
      else if(list[i].label.value.match(/[- ]succinyl/)){ obj.symbol = "s"; obj.color = "#007bce"; }
      else if(list[i].an_type.value.match(/Glycosylation_Annotation/)){ obj.symbol = "g"; obj.color = "#90ee90"; }  
      else{ obj.symbol = "*"; obj.color = "#936a76"; }
    }
    r.push(obj);
  }
  var list = get_substitution.results.bindings;
  for(var i = 0; i < list.length; i++){
    var obj = {
    	label: seq.slice(list[i].position.value - 1, list[i].position.value - 0) + "->" + list[i].substitution.value + " Natural variant",
    	position: list[i].position.value - 0,
        site: seq.slice(list[i].position.value - 1, list[i].position.value - 0),
    	symbol: list[i].substitution.value,
        y: 3,
        color: "#777777"
    }
    r.push(obj);
  }
  return r;
};
```

