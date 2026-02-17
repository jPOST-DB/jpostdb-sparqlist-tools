# protein list with evidence level (for Stanza 'protein_evidence')

## Parameters

* `dataset`
  * default: DS810_1
* `evidence`
  * default: Evidence_At_Transcript_Level
* `level`
  * default: 2
  * eaxmple: 1 2 3 # 1: Protein, 2: Leading protein, 3: Leading protein with unique peptide


## `filter`

```javascript
({dataset, evidence, level}) => {
  var array = decodeURIComponent(dataset).split(" ");
  var type = "obo:MS_1002401";
  var code = "";
  if(level == 1){
      type = "jpo:Protein";
  }else if(level == 3){
    code = "jpo:hasPeptideEvidence/jpo:hasPeptide/a obo:MS_1001363 ;"
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
({evidence, get_tax}) => {
  var code = "?up uniprot:existence uniprot:" + evidence + "_Existence .";
  if(get_tax.results.bindings[0].tax.value.match(/identifiers\.org\/taxonomy\/9606$/)){
    code = "?next skos:exactMatch ?up;\nnext:existence next:" + evidence.charAt(0).toUpperCase() + evidence.slice(1).toLowerCase() + " .";
  }
  return  {code: code};
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
  VALUES ?dataset { {{filter.values}} }
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
  return {id: max_prot};
};
```

## Endpoint

{{SPARQLIST_EP}}

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
  ?protein a {{filter.type}} ;
           {{filter.code}}
           rdfs:label ?upid ;
           jpo:hasDatabaseSequence ?up .
  {{filter2.code}}
  ?protein  jpo:hasPeptideEvidence/jpo:hasPeptide ?pep .
  ?pep jpo:hasPsm ?psm .
  ?up  uniprot:mnemonic ?symbol .
  ?up uniprot:proteome ?chr .
  FILTER(REGEX(STR(?chr), "{{proteome.id}}"))
  OPTIONAL { ?up (uniprot:recommendedName|uniprot:submittedName)/uniprot:fullName ?name . }
}
ORDER BY DESC (?pep_count) ?name
```

## Endpoint

{{SPARQLIST_EP}}

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
  ?protein a {{filter.type}} ;
           {{filter.code}}
           rdfs:label ?upid ;
           jpo:hasDatabaseSequence ?up .
  {{filter2.code}}
  ?protein  jpo:hasPeptideEvidence/jpo:hasPeptide ?pep .
  ?pep jpo:hasSequence [ a obo:MS_1001344 ;
                       rdf:value ?pepseq ] ;
       a jpo:UniquePeptideAtMsLevel .
}
ORDER BY ?name DESC (?pep_count)
```

## `return`

```javascript
({evidence, get_list, chk_unique}) =>{
  var list = chk_unique.results.bindings;
  var uniq_up = {};
  for(var i = 0; i < list.length; i++){
    uniq_up[list[i].upid.value] = list[i].uniq_count.value
  }
  var list = get_list.results.bindings;
  var head = ["Protein Name", "Accession", "ID", "Location", "# Peptides", "# PSMs", "# Unique Peptides"];
  var arg = ["name", "uniprot", "symbol", "chromosome", "pep_count", "psm_count", "uniq_count"];
  var align = [ 0, 0, 0, 0, 1, 1, 1];
  var data = [];
  for(var i = 0; i < list.length; i++){
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
      _alink_name: "./protein.php?id=" + list[i].upid.value,
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
