# Develop: protein with go term for slice stanza w/ protein estimation (original: protein_with_go_term) (for Stanza 'go_count')

## Parameters

* `dataset`
  * default: DS1631_1 DS1631_2 DS1637_1 DS1637_2
* `category`
  * default: biological_process
  * example: biological_process, molecular_function, cellular_component
* `term`
  * default: cellular process

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

## `filter`

```javascript
({dataset, category, term}) => {
  var array = decodeURIComponent(dataset).split(/[ ,]/);
  var go = "GO_0008150";
  if(category == "biological_process"){
    go = "GO_0008150";
  }else if(category == "molecular_function"){
    go = "GO_0003674";
  }else if(category == "cellular_component"){
    go = "GO_0005575";
  }
  return  { values: ":" + array.join(" :"), go: go};
};
```

## Endpoint

{{SPARQLIST_EP}}

## `get_list`

```sparql
#DEFINE sql:select-option "order"
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX uniprot: <http://purl.uniprot.org/core/>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT DISTINCT ?upid ?name ?symbol (COUNT (DISTINCT ?pep) AS ?pep_count) (COUNT (?psm) AS ?psm_count)
WHERE {
  VALUES ?dataset { {{filter.values}} }
  ?dataset jpo:hasProtein ?protein .
  ?protein a jpo:Protein ;
            rdfs:label ?upid ;
           jpo:hasDatabaseSequence ?up .
  ?protein  jpo:hasPeptideEvidence/jpo:hasPeptide ?pep .
  ?pep jpo:hasPsm ?psm .
  ?up  uniprot:mnemonic ?symbol .
  OPTIONAL { ?up (uniprot:recommendedName|uniprot:submittedName)/uniprot:fullName ?name . }
  ?up uniprot:classifiedWith ?go .
  FILTER (REGEX (?go, obo:GO_))
  ?go rdfs:subClassOf* ?level .
  ?level rdfs:subClassOf obo:{{filter.go}} .
  ?level rdfs:label "{{term}}"^^xsd:string .
}
ORDER BY DESC (?pep_count) ?name
```

## `return`

```javascript
({json, term, get_list}) =>{
  let filter = {};
  for (let acc of json) {
    filter[acc] = true;
  }
  let list = get_list.results.bindings;
  let head = ["Protein Name", "Accession", "ID", "# Peptides", "# PSMs"];
  let arg = ["name", "uniprot", "symbol", "pep_count", "psm_count"];
  let align = [ 0, 0, 0, 1, 1];
  let data = [];
  for (let row of list) {
    if (filter[row.upid.value]) {
      let name = "";
      if(row.name) name = row.name.value;
      var obj = {
        name: name,
        _alink_name: "./protein.php?id=" + row.upid.value,
//        _alink_name: "javascript:jPost.openProtein('" + row.upid.value + "')",
        symbol: row.symbol.value,
        uniprot: row.upid.value,
        _alink_uniprot: "http://www.uniprot.org/uniprot/" + row.upid.value,
        pep_count: row.pep_count.value - 0,
        psm_count: row.psm_count.value
      };
      data.push(obj);
    }
  }
 // data = data.sort(function(a,b){
 //   if(a.sort>b.sort) return 1;
 //   if(a.sort<b.sort) return -1;
//    if(a.pep_count>b.pep_count) return -1;
//    if(a.pep_count<b.pep_count) return 1;
//    return 0;
//    });
  return {title: term, head: head, arg: arg, align: align, data: data};
};
```
