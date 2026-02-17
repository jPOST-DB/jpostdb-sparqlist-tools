# get search box preset (for DB interface)

## Parameters

* `item`
  * default: organ
  * example: species, sampleType, cellLine, organ, disease, modification, instrument, (dataset, protein (for 'excluded_xxx' LIMIT 100))

## `filter`

```javascript
({item})=>{
  var code = '';
  var limit = '';
  if(item != "dataset" && item != "protein"){
    code = '  ?dataset a jpo:Dataset .\n';
    if(item == "modification"){
      code += '  ?dataset jpo:hasProfile/jpo:hasEnzymeAndModification/(jpo:variableModification|jpo:fixedModification) [ a ?object ;\n' +
           '           jpo:modificationClass jpo:JPO_022 ] .\n' +
           '  FILTER(?object != jpo:VariableModification && ?object != jpo:FixedModification)' +
           '  ?object rdfs:label ?label .\n'
    }else{
      if(item == "instrument"){
        code += '  ?dataset jpo:hasProfile/jpo:hasMsMode/jpo:' + item + ' ?object .\n';
      }else if(item == "disease"){
        code += '  ?dataset jpo:hasProfile/jpo:hasSample/jpo:' + item + 'Class ?object .\n';	
      }else{
        code += '  ?dataset jpo:hasProfile/jpo:hasSample/jpo:' + item + ' ?object .\n';	
      }
      if(item == "species"){
        code += '  ?object rdfs:seeAlso/skos:prefLabel ?label .';
      }else{
        code += '  ?object rdfs:label ?label .';
      }
      if(item == "cellLine"){
        code += '  FILTER(REGEX(STR(?object), "purl\.obolibrary\.org\/obo\/"))'
      }
    }
    limit = "ORDER BY DESC(?count)";	
  }else if(item == "dataset"){
    code =  "  ?dataset a jpo:Dataset ;\n";
    code += "          dct:identifier ?label ;\n";
    code += "          dct:identifier ?object ;\n";	
    limit = "ORDER BY ASC(?label)\nLIMIT 100";
  }else{
    code =  "  ?protein a jpo:Protein ;\n";
    code += "          jpo:hasDatabaseSequence ?up .\n";
    code += "  ?up uniprot:mnemonic ?label ;\n";
    code += "      uniprot:mnemonic ?object .\n"
    limit = "#ORDER BY ASC(?label)\nLIMIT 100";	
  }
  return {code: code, limit: limit};
};
```

## Endpoint

{{SPARQLIST_EP}}

## `get_label`

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX uniprot: <http://purl.uniprot.org/core/>
SELECT DISTINCT ?label ?object (COUNT(?object) AS ?count)
WHERE {
{{filter.code}}
}
{{filter.limit}}
```

## `return`

```javascript
({item, get_label})=>{
  for(var i = 0; i < get_label.results.bindings.length; i++){
    if(get_label.results.bindings[i].object){
      get_label.results.bindings[i].object.value = get_label.results.bindings[i].object.value
        .replace(/.+:\/\/purl\.obolibrary\.org\/obo\//, "")
        .replace(/.+:\/\/ncicb\.nci\.nih\.gov\/xml\/owl\/EVS\/Thesaurus.owl#/, "")
        .replace(/.+:\/\/identifiers\.org\/taxonomy\//, "TAX_")
        .replace(/.+:\/\/www\.unimod\.org\/obo\/unimod\.obo#/, "")
        .replace(/.+:\/\/rdf\.jpostdb\.org\/ontology\/jpost.owl#/, "");
      get_label.results.bindings[i].object.type = "typed-literal";
      get_label.results.bindings[i].object.datatype = "http://www.w3.org/2001/XMLSchema#string";
      if(item == "disease"){
        var disease_tmp = get_label.results.bindings[i].label.value;
      	get_label.results.bindings[i].label.value = disease_tmp.charAt(0).toUpperCase() + disease_tmp.slice(1);
        }
    }
  }
  return get_label;
};
```