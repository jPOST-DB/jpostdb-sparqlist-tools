# download peptide list (for UniProt -> jPOST link)

- UniProt 側での Link 作成のため API で使用
   - Peptide seq. list (dataset 毎)
   - 全 dataset 結合は Perl 側で処理

## Parameters

* `dataset`
  * default: DS1631_1

## Endpoint

{{SPARQLIST_EP}}

## `get_up_list`

```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX sio: <http://semanticscience.org/resource/>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX tax: <http://identifiers.org/taxonomy/>
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX : <http://rdf.jpostdb.org/entry/>
SELECT ?sequence ?best_jpost_score ?pep_seq_level_global_fdr ?dataset_id ?psm_count
WHERE {
  VALUES ?dataset { :{{dataset}} }
  ?dataset dct:identifier ?dataset_id ;
                   jpo:hasPeptide ?pep .
  ?pep jpo:hasSequence/rdf:value ?sequence .
  ?pep sio:SIO_000216 [ a jpo:UniScore ;
          sio:SIO_000300 ?best_jpost_score ] .
  ?pep sio:SIO_000216 [ a obo:MS_1001364 ;
          sio:SIO_000300 ?pep_seq_level_global_fdr ] . 
  ?pep sio:SIO_000216 [ a jpo:NumOfPsms ;
	  sio:SIO_000300 ?psm_count ] .
}
```
