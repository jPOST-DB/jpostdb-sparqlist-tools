# protein togovar (for Stnza 'protein_browser')

## Parameters

* `uniprot`
  * default: O00468
* `togovar_endpoint`
  * default: https://grch38.togovar.org/proxy/sparql
  * example: https://grch38.togovar.org/sparql

## Endpoint

{{SPARQLIST_EP}}

## `symbol`

```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX core: <http://purl.uniprot.org/core/>
PREFIX up: <http://purl.uniprot.org/uniprot/>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
SELECT ?symbol ?ensp ?type
FROM <http://jpost.org/graph/uniprot>
WHERE {
  up:{{uniprot}} core:encodedBy/skos:prefLabel ?symbol ;
                                           rdfs:seeAlso ?enst .
  ?enst core:translatedTo ?ensp .
  OPTIONAL { ?enst rdfs:seeAlso/rdf:type ?type .}
  FILTER (REGEX (?ensp, "ENSP"))
}
```

## `string`

```javascript
({symbol})=>{
  let obj = { symbol: "--------", ensp: "ENSP"};
  if(symbol.results.bindings[0]){
    let flag = 0;
    if(symbol.results.bindings[1]){
      for(let i = 0; i < symbol.results.bindings.length; i++){
        if(symbol.results.bindings[i].type && symbol.results.bindings[i].type.value.match("Simple_Sequence")){
          obj.symbol = symbol.results.bindings[i].symbol.value;
          obj.ensp = symbol.results.bindings[i].ensp.value.replace("http://rdf.ebi.ac.uk/resource/ensembl.protein/", "");
          flag = 1;
          break;
        }
      }
    }
    if(flag == 0){
      obj.symbol = symbol.results.bindings[0].symbol.value;
      obj.ensp = symbol.results.bindings[0].ensp.value.replace("http://rdf.ebi.ac.uk/resource/ensembl.protein/", "");
    }
  }
  return obj;
}
```

## Endpoint

{{togovar_endpoint}}

## `get_var`

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX tgdb: <http://togodb.org/ontology/proteome_ips_test#>
PREFIX cvo:  <http://purl.jp/bio/10/clinvar/>
PREFIX tgvo: <http://togovar.biosciencedbc.jp/vocabulary/>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX dct: <http://purl.org/dc/terms/>
SELECT DISTINCT ?tgv ?hgvs ?cons_type ?interpretation
WHERE {
  ?var tgvo:hasConsequence ?cons .
  ?cons a ?cons_type;
        tgvo:hgvsp ?hgvs ;
        tgvo:gene/rdfs:label "{{string.symbol}}" .
  FILTER (REGEX (?hgvs, "{{string.ensp}}"))
  FILTER (!REGEX (?hgvs, "="))
  GRAPH <http://togovar.org/variant> {
    ?var dct:identifier ?tgv .
  }
  OPTIONAL {
    GRAPH <http://togovar.org/variant/annotation/clinvar> {
      ?var dct:identifier ?var_id .
    }
    BIND(IRI(CONCAT("http://ncbi.nlm.nih.gov/clinvar/variation/", ?var_id)) AS ?clinvar)
    ?clinvar cvo:classified_record/cvo:rcv_list/cvo:rcv_accession/cvo:rcv_classifications/cvo:germline_classification/cvo:description/cvo:description ?interpretation .
  }
}
ORDER BY ?tgv
```

## `return`

```javascript
({get_var})=>{
  var list = get_var.results.bindings;
  var res = [];
  var aa = {Gly: "G", Ala: "A", Leu: "L", Met: "M", Phe: "F", Trp: "W", Lys: "K", Gln: "Q", Glu: "E", Ser: "S", 
            Pro: "P", Val: "V", Ile: "I", Cys: "C", Tyr: "Y", His: "H", Arg: "R", Asn: "N", Asp: "D", Thr: "T", Ter: "X"};
  var clinvar_sig = {'Pathogenic': {sig: 1, color: "#ff5a54"}, 'Likely pathogenic': {sig: 2, color: "#ffae00"}, 'Uncertain significance': {sig: 3, color: "#c7bca5"},
                     'Likely benign': {sig: 4, color: "#9dcf3a"}, 'Benign': {sig: 5, color: "#27d639"}, 'Conflicting interpretations of pathogenicity': {sig: 6, color: "#c654ff"},
                     'drug response': {sig: 7, color: "#9c8202"}, 'association': {sig: 8, color: "#9c8202"}, 'risk factor': {sig: 9, color: "#9c8202"}, 
                     'protective': {sig: 10, color: "#9c8202"}, 'affects': {sig: 11, color: "#9c8202"}, 'other': {sig: 12, color: "#9c8202"}, 'not provided': {sig: 13, color: "#999999"}};
  for(let i = 0; i < list.length; i++){
    let id = list[i].hgvs.value.match(/:p\.(.+)$/)[1];
    let array = id.match(/^([A-z]+)(\d+)(.+)$/);
    let var_letter = aa[array[3]];
    if(array[3].match(/^[A-Z][a-z]{2}/)) var_letter = aa[array[3].match(/^([A-Z][a-z]{2})/)[1]];
    else if(array[3].match(/^_[A-Z][a-z]{2}/)) var_letter = aa[array[3].match(/^_([A-Z][a-z]{2})/)[1]];
    else if(array[3].match(/^_.+ins[A-Z][a-z]{2}/)) var_letter = aa[array[3].match(/^_.+ins([A-Z][a-z]{2})/)[1]];
    else if(array[3].match(/^delins[A-Z][a-z]{2}/)) var_letter = aa[array[3].match(/^delins([A-Z][a-z]{2})/)[1]];
    else if(array[3].match(/del$/)) var_letter = "-";
    else if(array[3].match(/^dup/)) var_letter = aa[array[1]];
    else if(array[3] == "?") var_letter = "?";
    else if(array[3] == "=" || array[3] == "%3D") continue;
    else console.log(array[3]);
    let obj = {reference: aa[array[1]], 
               variant: var_letter, 
               position: array[2], 
               id: id,
               tgv: list[i].tgv.value, 
               color: "#777777", 
               sig: 99};
    if(list[i].interpretation){
      obj.interpretation = list[i].interpretation.value;
	  let inter_array = list[i].interpretation.value.split(/\//);
      obj.color = clinvar_sig[inter_array[0]].color;
      obj.sig = clinvar_sig[inter_array[0]].sig;
    }
    if (obj.variant && obj.reference) {
      res.push(obj);
    }
  }
  res.sort(function(a,b){
	if( parseInt(a.position) < parseInt(b.position) ) return -1;
	if( parseInt(a.position) > parseInt(b.position) ) return 1;
	if( a.sig < b.sig ) return 1;
	if( a.sig > b.sig ) return -1; 
	return 0;
  });
  let pre = 0;
  let y = 1;
  let respons = [];
  let chk = {};
  let max_y = 0;
  for(let i = 0; i < res.length; i++){
    if(!chk[res[i].tgv]){
      chk[res[i].tgv] = 1;
      if (pre == parseInt(res[i].position)) {
        y++;
        if (max_y < y) max_y = y;
      } else {
        y = 1;
        pre = parseInt(res[i].position);
      }
      res[i].y = y;
      respons.push(res[i]);
    }
  }
  for (let i = 0; i < respons.length; i++) {
    respons[i].y = max_y - respons[i].y + 1;
  }
  return respons;
};
```

