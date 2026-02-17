# protein count in each chromosome (for Stanza 'chromosome_histogram')

- Dataset 内のタンパク質の染色体分布
  - total は UniProt のカウント

## Parameters

* `dataset` (Req.)
  * default: DS1631_1
  * example: DS1631_1


## `filter`

```javascript
({dataset})=>{
      return {code: "VALUES ?dataset { :" + decodeURIComponent(dataset).split(/[ ,]/).join(" :") + " }" };
};
```

## Endpoint

{{SPARQLIST_EP}}

## `get_chromosome`

```sparql
PREFIX uniprot: <http://purl.uniprot.org/core/>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT ?chr (COUNT (DISTINCT ?up) AS ?count)
WHERE {
{{filter.code}}
  ?dataset jpo:hasProtein ?protein .
  ?protein a obo:MS_1002401 .  ### leading protein
  ?protein jpo:hasDatabaseSequence ?up .
  ?up uniprot:proteome ?chr .
}
```

## `proteome`

```javascript
({get_chromosome})=>{
  var list = get_chromosome.results.bindings;
  var chk = [];
  var total = [];
  for(var i = 0; i < list.length; i++){
    var prot = list[i].chr.value.split("#")[0].split("/")[4];
    if(!chk[prot]){
      chk[prot] = 1;
      total[prot] = 0;
    }
    total[prot] += list[i].count.value - 0
  }
  var max = 0;
  var max_prot;
  var prot = Object.keys(total);
  for(var i = 0; i < prot.length; i++){
    console.log(total[prot[i]]);
    if(max < total[prot[i]]){
      max = total[prot[i]];
      max_prot = prot[i];
    }
  }
//  var database = "GeneID";
  var database = "";
  if(max_prot == "UP000005640") database = "?up rdfs:seeAlso/uniprot:database <http://purl.uniprot.org/database/neXtProt> .";
  return {id: max_prot, database: database};
}
```

## Endpoint

{{SPARQLIST_EP}}

## `get_dataset_count`

```sparql
PREFIX uniprot: <http://purl.uniprot.org/core/>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT ?chr (COUNT (DISTINCT ?up) AS ?count)
WHERE {
{{filter.code}}
  ?dataset jpo:hasProtein ?protein .
  ?protein a obo:MS_1002401 .  ### leading protein
  ?protein jpo:hasDatabaseSequence ?up .
  ?up uniprot:proteome ?chr .
  {{proteome.database}}
}
```

## Endpoint

{{SPARQLIST_EP}}

## `get_all_count`

```sparql
PREFIX uniprot: <http://purl.uniprot.org/core/>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT ?chr (COUNT (DISTINCT ?up) AS ?count)
WHERE {
  ?up uniprot:proteome ?chr .
  {{proteome.database}}
  FILTER( REGEX( STR(?chr), "/{{proteome.id}}#"))
}
```

## `return`

```javascript
({get_chromosome, get_all_count})=>{
  var list = get_chromosome.results.bindings;
  var total_list = get_all_count.results.bindings;
  var count = [];
  var total = [];
  for(var i = 0; i < total_list.length; i++){
    var prot = total_list[i].chr.value.split("#")[0];
    var c = 0;
    for(var j = 0; j < list.length; j++){
      if(total_list[i].chr.value == list[j].chr.value) c = list[j].count.value;
    }
    var label = total_list[i].chr.value.split("#")[1];
    label = label.replace(/%20/g, " ").replace("Chromosome", "Ch.");
    count[label] = c;
    total[label] = total_list[i].count.value;
  }
  var max = 0;
  var chr1 = [];
  var chr2 = [];
  for(var i = 0; i < total_list.length; i++){
    var label = total_list[i].chr.value.split("#")[1];
    label = label.replace(/%20/g, " ").replace("Chromosome", "Ch.");
    if(label.match(/Ch. /)){
      var tmp = label.match(/Ch. (.+)/)[1];
      if(tmp.match(/^\d+$/)){
        if(tmp-0 > max) max = tmp-0	;
      }else{
        chr1.push(tmp);
      }
    }else{
      chr2.push(label);
    }
  }
  chr1 = chr1.sort();
  chr2 = chr2.sort();
  var r = [];
  for(var i = 1; i <= max; i++){		
    r.push({label: "Ch. " + i, count: count["Ch. " + i], total: total["Ch. " + i]});
  }	
  for(var i = 0; i < chr1.length; i++){
    r.push({label: "Ch. " + chr1[i], count: count["Ch. " + chr1[i]], total: total["Ch. " + chr1[i]]});
  }
  for(var i = 0; i < chr2.length; i++){
    r.push({label: chr2[i], count: count[chr2[i]], total: total[chr2[i]]});	
  }  
  return r;
};	
```
