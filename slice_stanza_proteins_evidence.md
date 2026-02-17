# Develop: proteins evidence for slice stanza w/ protein estimation (original: proteins_evidence) (for Stanza 'proteins_evidence')

## Parameters

* `dataset`
  * default: DS1631_1 DS1631_2 DS1637_1 DS1637_2

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

## `filter`

```javascript
({taxonomy}) => {
  let code = "?protein uniprot:existence ?exist .";
  if(taxonomy.results.bindings[0].tax.value.match(/identifiers\.org\/taxonomy\/9606$/)){
    code = "?next skos:exactMatch ?protein ;\nnext:existence ?exist .";
  }
  return  {code: code};
};
```

## `evidence`

```sparql
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX uniprot: <http://purl.uniprot.org/core/>
PREFIX next: <http://nextprot.org/rdf#>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX ms: <http://purl.obolibrary.org/obo/MS_>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT ?exist ?protein
WHERE {
  ?protein a uniprot:Protein ;
           uniprot:organism <{{taxonomy.results.bindings.[0].tax.value}}> .
  {{filter.code}}
}
ORDER BY ?exist
```

## `format`

```javascript
({json, evidence}) => {
  let filter = {};
  for (let acc of json) {
    filter[acc] = true;
  }
  
  let data = evidence.results.bindings;
  n = ["Evidence at Protein Level",
      "Evidence at Transcript Level",
      "Inferred from Homology",
      "Predicted",
      "Uncertain"
      ];
  c = [0,0,0,0,0];
  r = [];
  for (let row of data){
    if (filter[row.protein.value.replace(/http:\/\/purl\.uniprot\.org\/uniprot\//, "")]) {
      if (row.exist.value.toLowerCase().match(/evidence_at_protein_level/)) c[0]++;
      else if (row.exist.value.toLowerCase().match(/evidence_at_transcript_level/)) c[1]++;
      else if (row.exist.value.toLowerCase().match(/inferred_from_homology/)) c[2]++;
 	  else if (row.exist.value.toLowerCase().match(/predicted/)) c[3]++;
      else if (row.exist.value.toLowerCase().match(/uncertain/)) c[4]++;
    }
  }
  
  for(var i = 0; i < 5; i++){
    if(n[i]) r.push({label: n[i], id: n[i].replace(/ /g, "_"), count: c[i].toString()}); 
  }
  return r;
};
```


