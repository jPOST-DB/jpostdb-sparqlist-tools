# statistics for slice comparison (for Stanza 'slice_comp_stats')

## Parameters


* `dataset1` (Req.)
  * default: DS1632_1 DS1632_2 DS1632_3
* `dataset2` (Req.)
  * default: DS1637_1 DS1637_2 DS1637_3
* `slice1` (Opt.)
* `slice2` (Opt.)

## `filter`

```javascript
({dataset1, dataset2})=>{
  var code1 = "";
  var code2 = "";
  if(dataset1) code1 += "  VALUES ?dataset1 { :" + decodeURIComponent(dataset1).split(/[ ,]+/).join(" :") + " }\n";
  if(dataset2) code2 += "  VALUES ?dataset2 { :" + decodeURIComponent(dataset2).split(/[ ,]+/).join(" :") + " }\n";
  return {code1: code1, code2: code2}
};
```

## Endpoint

{{SPARQLIST_EP}}

## `protein`

```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT ?prot1 ?prot2 ?prot_share ?pep1 ?pep2 ?pep_share
WHERE {
  {
    SELECT (COUNT (DISTINCT ?protein) AS ?prot1)
    WHERE {
      {{filter.code1}}
      ?dataset1 jpo:hasProtein ?prt .
      ?prt a obo:MS_1002401 ;
           jpo:hasDatabaseSequence ?protein .
    }
  }
  {
    SELECT (COUNT (DISTINCT ?protein) AS ?prot2)
    WHERE {
      {{filter.code2}}
      ?dataset2 jpo:hasProtein ?prt .
      ?prt a obo:MS_1002401 ;
           jpo:hasDatabaseSequence ?protein .
    }
  }
  {
    SELECT (COUNT (DISTINCT ?protein) AS ?prot_share)
    WHERE {
      {{filter.code1}}
      {{filter.code2}}
      ?dataset1 jpo:hasProtein ?prt1 .
      ?prt1 a obo:MS_1002401 ;
           jpo:hasDatabaseSequence ?protein .
      ?dataset2 jpo:hasProtein ?prt2 .
      ?prt2 a obo:MS_1002401 ;
           jpo:hasDatabaseSequence ?protein .
    }
  }
  {
    SELECT (COUNT (DISTINCT ?peptide) AS ?pep1)
    WHERE {
      {{filter.code1}}
      ?dataset1 jpo:hasPeptide/jpo:hasSequence [ a obo:MS_1001344 ;
                rdf:value ?peptide ] .
    }
  }
  {
    SELECT (COUNT (DISTINCT ?peptide) AS ?pep2)
    WHERE {
      {{filter.code2}}
      ?dataset2 jpo:hasPeptide/jpo:hasSequence [ a obo:MS_1001344 ;
                rdf:value ?peptide ] .
    }
  }
  {
    SELECT (COUNT (DISTINCT ?peptide1) AS ?pep_share)
    WHERE {
      {
        SELECT DISTINCT ?peptide1
        WHERE {
          {{filter.code1}}
          ?dataset1 jpo:hasProtein ?prt1 .
          ?prt1 a obo:MS_1002401 ;
                jpo:hasPeptideEvidence/jpo:hasPeptide/jpo:hasSequence [ a obo:MS_1001344 ;
                rdf:value ?peptide1 ] .
        }
      }
      {
        SELECT DISTINCT ?peptide2
        WHERE {
          {{filter.code2}}
          ?dataset2 jpo:hasProtein ?prt2 .
          ?prt2 a obo:MS_1002401 ;
                jpo:hasPeptideEvidence/jpo:hasPeptide/jpo:hasSequence [ a obo:MS_1001344 ;
                rdf:value ?peptide2 ] .
        }
      }
      FILTER(?peptide1 = ?peptide2) 
    }
  }
}
```

## `return`

```javascript
({dataset1, dataset2, slice1, slice2, protein})=>{
  var ds1 = decodeURIComponent(dataset1).split(/[ ,]+/);
  var ds2 = decodeURIComponent(dataset2).split(/[ ,]+/);
  var ds_share = 0;
  for(var i = 0; i < ds1.length; i++){
    if(dataset2.match(ds1[i])) ds_share++;
  }
  var s1 = "slice: 1";
  var s2 = "slice: 2";
  if(slice1) s1 = "slice: " + slice1;
  if(slice2) s2 = "slice: " + slice2;
  var data = protein.results.bindings[0];
  var prot1_uniq = data.prot1.value - data.prot_share.value;
  var prot2_uniq = data.prot2.value - data.prot_share.value;
  var pep1_uniq = data.pep1.value - data.pep_share.value;
  var pep2_uniq = data.pep2.value - data.pep_share.value;
  return {
    slice1: s1,
    slice2: s2,
    prot1: data.prot1.value.replace(/(\d)(?=(\d{3})+$)/g , '$1,'),
    prot2: data.prot2.value.replace(/(\d)(?=(\d{3})+$)/g , '$1,'), 
    prot_share: data.prot_share.value.replace(/(\d)(?=(\d{3})+$)/g , '$1,'), 
    prot1_uniq: prot1_uniq.toString().replace(/(\d)(?=(\d{3})+$)/g , '$1,'),
    prot2_uniq: prot2_uniq.toString().replace(/(\d)(?=(\d{3})+$)/g , '$1,'), 
    pep1: data.pep1.value.replace(/(\d)(?=(\d{3})+$)/g , '$1,'),
    pep2: data.pep2.value.replace(/(\d)(?=(\d{3})+$)/g , '$1,'), 
    pep_share: data.pep_share.value.replace(/(\d)(?=(\d{3})+$)/g , '$1,'), 
    pep1_uniq: pep1_uniq.toString().replace(/(\d)(?=(\d{3})+$)/g , '$1,'),
    pep2_uniq: pep2_uniq.toString().replace(/(\d)(?=(\d{3})+$)/g , '$1,'),
    ds1: ds1.length,
    ds2: ds2.length,
    ds_share: ds_share,
    ds1_uniq: ds1.length - ds_share,
    ds2_uniq: ds2.length - ds_share
         }
};
```