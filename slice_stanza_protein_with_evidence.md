# Develop: protein list with evidence level for slice stanza w/ protein estimation (original: protein_with_evidence) (for Stanza 'protein_evidence')

## Parameters

* `dataset`
  * default: DS1631_1 DS1631_2 DS1637_1 DS1637_2
* `evidence`
  * default: Evidence_At_Transcript_Level

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
({dataset, evidence, taxonomy}) => {
  var array = decodeURIComponent(dataset).split(" ");
  var code = "?up uniprot:existence uniprot:" + evidence + "_Existence .";
  if(taxonomy.results.bindings[0].tax.value.match(/\/taxonomy\/9606$/)){
    code = "?next skos:exactMatch ?up ;\n    next:existence next:" + evidence.charAt(0).toUpperCase() + evidence.slice(1).toLowerCase() + " .";
  }
  return  {values: ":" + array.join(" :"), code: code};
};
```

## `chromosome`

```sparql
PREFIX uniprot: <http://purl.uniprot.org/core/>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT ?chr (COUNT (DISTINCT ?up) AS ?count)
WHERE {
  VALUES ?dataset { {{filter.values}} }
  ?dataset jpo:hasProtein ?protein .
  ?protein a obo:MS_1002401 .  ### leading protein
  ?protein jpo:hasDatabaseSequence ?up .
  ?up uniprot:proteome ?chr .
}
```

## `proteome`

```javascript
({chromosome})=>{
  var list = chromosome.results.bindings;
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
  return {id: max_prot};
};
```

## `get_list`

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX uniprot: <http://purl.uniprot.org/core/>
PREFIX next: <http://nextprot.org/rdf#>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT DISTINCT ?upid ?name ?symbol (COUNT (DISTINCT ?pep) AS ?pep_count) (COUNT (?psm) AS ?psm_count) ?chr
WHERE {
  VALUES ?dataset { {{filter.values}} }
  ?dataset jpo:hasProtein ?protein .
  ?protein a jpo:Protein ;
           rdfs:label ?upid ;
           jpo:hasDatabaseSequence ?up .
  {{filter.code}}
  ?protein  jpo:hasPeptideEvidence/jpo:hasPeptide ?pep .
  ?pep jpo:hasPsm ?psm .
  ?up  uniprot:mnemonic ?symbol .
  ?up uniprot:proteome ?chr .
  FILTER(REGEX(STR(?chr), "{{proteome.id}}"))
  OPTIONAL { ?up (uniprot:recommendedName|uniprot:submittedName)/uniprot:fullName ?name . }
}
ORDER BY DESC (?pep_count) ?name
```

## `chk_unique`

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX uniprot: <http://purl.uniprot.org/core/>
PREFIX next: <http://nextprot.org/rdf#>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT DISTINCT ?upid (COUNT(DISTINCT ?pepseq) AS ?uniq_count)
WHERE {
  VALUES ?dataset { {{filter.values}} }
  ?dataset jpo:hasProtein ?protein .
  ?protein a jpo:Protein ;
           rdfs:label ?upid ;
           jpo:hasDatabaseSequence ?up .
  {{filter.code}}
  ?protein  jpo:hasPeptideEvidence/jpo:hasPeptide ?pep .
  ?pep jpo:hasSequence [ a obo:MS_1001344 ;
                       rdf:value ?pepseq ] ;
       a jpo:UniquePeptideAtMsLevel .
}
ORDER BY ?name DESC (?pep_count)
```

## `return`

```javascript
({json, evidence, get_list, chk_unique}) =>{
  let filter = {};
  for (let acc of json) {
    filter[acc] = true;
  }
  var list = chk_unique.results.bindings;
  var uniq_up = {};
  for (var i = 0; i < list.length; i++) {
    if (!filter[list[i].upid.value]) continue;
    uniq_up[list[i].upid.value] = list[i].uniq_count.value
  }
  var list = get_list.results.bindings;
  var head = ["Protein Name", "Accession", "ID", "Location", "# Peptides", "# PSMs", "# Unique Peptides"];
  var arg = ["name", "uniprot", "symbol", "chromosome", "pep_count", "psm_count", "uniq_count"];
  var align = [ 0, 0, 0, 0, 1, 1, 1];
  var data = [];
  for(var i = 0; i < list.length; i++){
    if (!filter[list[i].upid.value]) continue;
    var uniq_count = 0;
    if(uniq_up[list[i].upid.value]) uniq_count = uniq_up[list[i].upid.value];
    var name = "";
    if(list[i].name) name = list[i].name.value;
    var chr = list[i].chr.value.match(/#([^#]+)$/)[1].replace("%20", " ");
    chr = chr.replace("Chromosome ", "Ch. ");
    var sort = chr;
    if(sort.match(/^Ch. /)) sort = "0" + sort.replace("Ch. ", ""); 
    if(sort.match(/^\d+$/)){
      if(sort - 0 < 10) sort = "0" + sort;
    } 
    var obj = {
      name: name,
//      _alink_name: "./protein?id=" + list[i].upid.value,
      _alink_name: "javascript:jPost.openProtein('" + list[i].upid.value + "')",
      symbol: list[i].symbol.value,
      uniprot: list[i].upid.value,
      _alink_uniprot: "http://www.uniprot.org/uniprot/" + list[i].upid.value,
      pep_count: list[i].pep_count.value - 0,
      psm_count: list[i].psm_count.value,
      chromosome: chr,
      sort: sort,
      uniq_count: uniq_count
    };
    data.push(obj);
  }
  data = data.sort(function(a,b){
    if(a.sort>b.sort) return 1;
    if(a.sort<b.sort) return -1;
    if(a.pep_count>b.pep_count) return -1;
    if(a.pep_count<b.pep_count) return 1;
    return 0;
    });
  return {title: evidence.replace(/_/g, " "), head: head, arg: arg, align: align, data: data};
};
```
