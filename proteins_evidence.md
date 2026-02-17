# proteins evidence via uniprot (nextprot for human) (for Stanza 'proteins_evidence')

## Parameters

* `dataset` (Req.)
  * default: DS810_1
* `level` (Req.)
  * default: 2
  * example: 1: protein, 2: leading protein, 3: protein with uniquq-pep


## `filter`

```javascript
({dataset, level}) => {
  var array = decodeURIComponent(dataset).split(/[ ,]/);
  var type = "obo:MS_1002401";
  var code = "";
  if(level == 1){
  	type = "jpo:Protein";
  }else if(level == 3){
    code = "jpo:hasPeptideEvidence/jpo:hasPeptide/a jpo:UniquePeptideAtMsLevel ;"
  }
  return  { values: ":" + array.join(" :"), type: type, code: code};
};
```

## Endpoint

{{SPARQLIST_EP}}

## `get_tax`

```sparql
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT DISTINCT ?tax
WHERE {
  VALUES ?dataset { {{filter.values}} }
  ?dataset jpo:hasProfile/jpo:hasSample/jpo:species ?tax .
}
```

## `filter2`

```javascript
({get_tax}) => {
  var code = "?up uniprot:existence ?exist .";
  if(get_tax.results.bindings[0].tax.value.match(/identifiers\.org\/taxonomy\/9606$/)){
    code = "?next skos:exactMatch ?up ;\nnext:existence ?exist .";
  }
  return  {code: code};
};
```

## Endpoint

{{SPARQLIST_EP}}

## `get_evidence`

```sparql
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX uniprot: <http://purl.uniprot.org/core/>
PREFIX next: <http://nextprot.org/rdf#>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX ms: <http://purl.obolibrary.org/obo/MS_>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT ?exist (COUNT (DISTINCT ?up) AS ?count)
WHERE {
  VALUES ?dataset { {{filter.values}} }
  ?dataset jpo:hasProtein ?protein .
  ?protein a {{filter.type}} ;
           {{filter.code}}
           jpo:hasDatabaseSequence ?up .
  {{filter2.code}}
}
```

## `return`

```javascript
({get_evidence}) => {
  var data = get_evidence.results.bindings;
  n = [];
  c = [];
  r = [];
  for(var i = 0; i < data.length; i++){
    if(data[i].exist.value.toLowerCase().match(/evidence_at_protein_level/)){
      n[0] = "Evidence at Protein Level";
      c[0] = data[i].count.value;
    }else if(data[i].exist.value.toLowerCase().match(/evidence_at_transcript_level/)){
      n[1] = "Evidence at Transcript Level";
      c[1] = data[i].count.value;
    }else if(data[i].exist.value.toLowerCase().match(/inferred_from_homology/)){
      n[2] = "Inferred from Homology";
      c[2] = data[i].count.value;
    }else if(data[i].exist.value.toLowerCase().match(/predicted/)){
      n[3] = "Predicted";
      c[3] = data[i].count.value;
    }else if(data[i].exist.value.toLowerCase().match(/uncertain/)){
      n[4] = "Uncertain";
      c[4] = data[i].count.value;
    }
  }
  for(var i = 0; i < 5; i++){
    if(n[i]) r.push({label: n[i], id: n[i].replace(/ /g, "_"), count: c[i]}); 
  }
  return r
};
```

