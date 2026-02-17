# statistics between slices (for Stanza 'slice_comparison')

- ２つのデータセットリスト間で、タンパク質、ペプチドの共有統計情報を取得

## Parameters

* `dataset1`
  * default: DS1631_1 DS1632_1 DS1633_1 DS1634_1 DS1631_2 DS1631_3
* `dataset2`
  * default: DS1637_1 DS1635_1 DS1636_1 DS1635_2


## `filter`

```javascript
({dataset1, dataset2})=>{
  var code1 = "";
  var code2 = "";
  if(dataset1.match(/DS/)) code1 += "  VALUES ?dataset1 { :" + dataset1.replace(/%20/g, " ").split(" ").join(" :") + " }\n";
  if(dataset2.match(/DS/)) code2 += "  VALUES ?dataset2 { :" + dataset2.replace(/%20/g, " ").split(" ").join(" :") + " }\n";
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
      ?dataset1 jpo:hasProtein/jpo:hasDatabaseSequence ?protein .
    }
  }
  {
    SELECT (COUNT (DISTINCT ?protein) AS ?prot2)
    WHERE {
      {{filter.code2}}
      ?dataset2 jpo:hasProtein/jpo:hasDatabaseSequence ?protein .
    }
  }
  {
    SELECT (COUNT (DISTINCT ?protein) AS ?prot_share)
    WHERE {
      {{filter.code1}}
      {{filter.code2}}
      ?dataset1 jpo:hasProtein/jpo:hasDatabaseSequence ?protein .
      ?dataset2 jpo:hasProtein/jpo:hasDatabaseSequence ?protein .
    }
  }
  {
    SELECT (COUNT (DISTINCT ?peptide) AS ?pep1)
    WHERE {
      {{filter.code1}}
      ?dataset1 jpo:hasPeptide/jpo:hasSequence [a obo:MS_1001344 ;
                                                  rdf:value ?peptide ].
    }
  }
  {
    SELECT (COUNT (DISTINCT ?peptide) AS ?pep2)
    WHERE {
      {{filter.code2}}
      ?dataset2 jpo:hasPeptide/jpo:hasSequence [a obo:MS_1001344 ;
                                                  rdf:value ?peptide ].
    }
  }
  {
    SELECT (COUNT (DISTINCT ?peptide) AS ?pep_share)
    WHERE {
      {{filter.code1}}
      {{filter.code2}}
      ?dataset1 jpo:hasPeptide/jpo:hasSequence [a obo:MS_1001344 ;
                                                  rdf:value ?peptide ].
      ?dataset2 jpo:hasPeptide/jpo:hasSequence [a obo:MS_1001344 ;
                                                  rdf:value ?peptide ].
    }
  }
}
```

## `return`

```javascript
({protein})=>{
  var data = protein.results.bindings[0];
  return {
    prot1: data.prot1.value,
    prot2: data.prot2.value, 
    prot_share: data.prot_share.value, 
    prot1_uniq: data.prot1.value - data.prot_share.value,
    prot2_uniq: data.prot2.value - data.prot_share.value, 
    pep1: data.pep1.value,
    pep2: data.pep2.value, 
    pep_share: data.pep_share.value, 
    pep1_uniq: data.pep1.value - data.pep_share.value,
    pep2_uniq: data.pep2.value - data.pep_share.value
         }
};
```
