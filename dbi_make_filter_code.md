# make filtering SPARQL code (for DB interface) req. dbi_to_taxid, dbi_to_dois

## Parameters

* `mode`
  * example: project, dataset, protein, peptide, psm
* `datasets` (Opt.)
  * example: DS1631_1,DS1631_2,DS1631_3
* `proteins`
  * example: PRT1631_1_P01111 P01111
* `peptides`
  * example: PEP1631_1_697
* `species` (Opt.)
  * default: 
* `species_s` (Opt.)
  * example: vertebrates
* `sample_type` (Opt.)
  * example: C16403
* `cell_line` (Opt.)
* `organ` (Opt.)
* `disease` (Opt.)
* `disease_s` (Opt.)
  * example: cancer
* `modification` (Opt.)
  * example: UNIMOD_21
* `instrument_mode` (Opt.)
  * example: JPO_006
* `instrument` (Opt.)
* `project_keywords` (Opt.)
  * example: CH2O,by trypsin
* `dataset_keywords` (Opt.)
  * example: CH2O,by trypsin
* `protein_keywords` (Opt.)
  *example: tyrosin,kinase
* `excluded_datasets` (Opt.)
  * example: DS1631_1,DS1631_2
* `excluded_proteins` (Opt.)
  * example: P00519,ABL2
* `order` (Opt.)
  * example: mnemonic
* `desc` (Opt.)
  * example: 1
* `limit` (Opt.)
  * example: 10
* `offset` (Opt.)
  * example: 0


## `get_ids`

```javascript
async ({species_s, disease_s}) => {
  var species = [];
  var disease = [];
  var sparqlet = (api, body) => {
    const options = {
      method: 'POST',
      body: body,
      headers: {
        'Accept': 'application/sparql-results+json',
        'Content-Type': 'application/x-www-form-urlencoded'
      }
    };
    var res = fetch(api, options).then(res=>res.json());
    return res;
  }
  if(species_s){
    var body = "string=" + encodeURIComponent(species_s);
    var res = await sparqlet("https://db-dev.jpostdb.org/rest/api/dbi_to_taxid", body);
    for(var i = 0; i < res.results.bindings.length; i++){
      if(res.results.bindings[i].id) species.push(res.results.bindings[i].id.value.replace(/.+purl\.uniprot\.org\/taxonomy\//, "tax:"));
    }
  }
  if(disease_s){
    var body = "string=" + encodeURIComponent(disease_s);
    var res = await sparqlet("https://db-dev.jpostdb.org/rest/api/dbi_to_doid", body);
    for(var i = 0; i < res.results.bindings.length; i++){
      if(res.results.bindings[i].id) disease.push(res.results.bindings[i].id.value.replace(/.+purl\.obolibrary\.org\/obo\//, "obo:"));
    }
  }
    return {species: species.join(" "), disease: disease.join(" ")}
};
```

## `filter_code`

```javascript
({mode, datasets, proteins, peptides, species, sample_type, cell_line, organ, disease, modification, 
  instrument_mode, instrument, project_keywords, dataset_keywords, protein_keywords, excluded_datasets, 
  excluded_proteins, order, desc, limit, offset, get_ids}) => {
  var code_value = "";
  var code_protein_value = "";
  var code_peptide_value = "";
  var code_up_label_value = "";
  var code_dataset = "";
  var code_protein = "";
  var code_up_label = "";
  var code_limit = "";
  var StoA = function(string, ds_id, q){
    var ids = [];
    var labels = [];
    var a = decodeURIComponent(string).split(",");
    for(var i = 0; i < a.length; i++){
      if(ds_id) a[i] = ds_id + a[i];
      if(q) a[i] = "'" + a[i] + "'";
      if(a[i].match(/^DS\d+_\d+$/)      ||
         a[i].match(/^PRT\d+_\d+_\w+$/) ||
         a[i].match(/^PEP\d+_\d+_\d+$/))  ids.push(":" + a[i]);
      else if(a[i].match(/^TAX_\d+$/))    ids.push(a[i].replace(/^TAX_/, "tax:"));
      else if(a[i].match(/^C\d+$/))       ids.push("ncit:" + a[i]); 	
      else if(a[i].match(/^UNIMOD_\d+$/)) ids.push("unimod:" + a[i]);     
      else if(a[i].match(/^JPO_\d+$/))    ids.push("jpo:" + a[i]);
      else if(a[i].match(/^CLO*_\d+$/))   ids.push("obo:" + a[i]); 
      else if(a[i].match(/^DOID_\d+$/))   ids.push("obo:" + a[i]); 	
      else if(a[i].match(/^MS_\d+$/))     ids.push("obo:" + a[i]);
      else ids.push(a[i]);
    }
    return ids;
  };
  if(datasets){
    code_value += "  VALUES ?dataset { " + StoA(datasets).join(" ") + " }\n";
  }
  if(proteins){
    if(!proteins.match(/^PRT\d+_\d+_[\w\-]+$/)){
       var ds_id = ""
       if(datasets && datasets.match(/^DS\d+_\d+$/)){
          ds_id = "PRT" + datasets.match(/^DS(\d+_\d+)$/)[1] + "_";
       }else if(peptides && peptides.match(/^PEP\d+_\d+$/)){
          ds_id = "PRT" + datasets.match(/^PEP(\d+_\d+)$/)[1] + "_";
       }
       if(ds_id){
         code_protein_value += "  VALUES ?db_protein { " + StoA(proteins, ds_id).join(" ") + " }\n";
       }else{
         code_up_label_value += "  VALUES ?up_label { " + StoA(proteins, ds_id, 1).join(" ") + " }\n"; 
         code_up_label += "  ?db_protein rdfs:label ?up_label .\n";  
       }
    }else{  
       code_protein_value += "  VALUES ?db_protein { " + StoA(proteins).join(" ") + " }\n";
    }
  }
  if(peptides){  
    code_peptide_value += "  VALUES ?peptide { " + StoA(peptides).join(" ") + " }\n";
  }
  if(excluded_datasets){
    code_dataset  += "  FILTER(?dataset NOT IN( " + StoA(excluded_datasets).join(",") + "))\n";
  }
  if(species || get_ids.species){
    code_value += "  VALUES ?species { ";
    if(species)         code_value += StoA(species).join(" ");
    if(get_ids.species) code_value += get_ids.species;
    code_value += " }\n";
   // avoid property path bug of virtuoso ( multi species + disease )
   // code_dataset  += "  ?dataset jpo:hasProfile/jpo:hasSample/jpo:species/rdfs:seeAlso/rdfs:subClassOf* ?species .\n";
    code_dataset  += "{\n    ?dataset jpo:hasProfile/jpo:hasSample/jpo:species/rdfs:seeAlso ?species .\n  } UNION {\n    ?dataset jpo:hasProfile/jpo:hasSample/jpo:species/rdfs:seeAlso/rdfs:subClassOf+ ?species .\n  }\n";
  } 
  if(sample_type){
    code_value += "  VALUES ?sample_type { " + StoA(sample_type).join(" ") + " }\n";
    code_dataset  += "  ?dataset jpo:hasProfile/jpo:hasSample/jpo:sampleType ?sample_type .\n";
  }
  if(cell_line){	
    code_value += "  VALUES ?cell_line { " + StoA(cell_line).join(" ") + " }\n";
    code_dataset  += "  ?dataset jpo:hasProfile/jpo:hasSample/jpo:cellLine ?cell_line .\n";
  }
  if(organ){	
    code_value += "  VALUES ?organ { " + StoA(organ).join(" ") + " }\n";
    code_dataset  += "  ?dataset jpo:hasProfile/jpo:hasSample/jpo:organ ?organ .\n";
  }
  if(disease || get_ids.disease){	
    code_value += "  VALUES ?disease { ";
    if(disease)         code_value += StoA(disease).join(" ");
    if(get_ids.disease) code_value += get_ids.disease;
    code_value += " }\n";
   // avoid property path bug of virtuoso ( multi species + disease )
   // code_dataset  += "  ?dataset jpo:hasProfile/jpo:hasSample/((jpo:disease/rdfs:subClassOf)|jpo:diseaseClass) ?disease .\n";
    code_dataset  += "{\n    ?dataset jpo:hasProfile/jpo:hasSample/jpo:disease/rdfs:subClassOf* ?disease .\n  } UNION {\n    ?dataset jpo:hasProfile/jpo:hasSample/jpo:diseaseClass ?disease .\n  }\n";
  } 
  if(modification){
    code_value += "  VALUES ?modification { " + StoA(modification).join(" ") + " }\n";
    code_dataset += "  ?dataset jpo:hasProfile/jpo:hasEnzymeAndModification/\n";
    code_dataset += "          (jpo:fixedModification|jpo:variableModification) [ a ?modification ;\n";
    code_dataset += "                                                              jpo:modificationClass jpo:JPO_022 ] .\n";
  }
  if(instrument_mode){	
    code_value += "  VALUES ?instrument_mode { " + StoA(instrument_mode).join(" ") + " }\n";
    code_dataset  += "  ?dataset jpo:hasProfile/jpo:hasMsMode/jpo:instrumentMode ?instrument_mode .\n";
  } 
  if(instrument){	
    code_value += "  VALUES ?instrument { " + StoA(instrument).join(" ") + " }\n";
    code_dataset  += "  ?dataset jpo:hasProfile/jpo:hasMsMode/jpo:instrument ?instrument .\n";
  }
  if (project_keywords) {
    dataset_keywords += "," + project_keywords;
  }
  if (dataset_keywords) {
    var array = StoA(dataset_keywords);
    code_dataset += "  { SELECT ?project { ?project (dct:description|dct:title|rdfs:comment|dct:identifier|(jpo:hasDataset/dct:identifier)|(jpo:hasDataset/jpo:hasProfile/jpo:hasSample/rdfs:comment)|(jpo:hasDataset/jpo:hasProfile/jpo:hasSample/jpo:species/rdfs:seeAlso/skos:prefLabel)) ?dataset_search_space .\n";	
    code_dataset += "  FILTER( true\n"
    for(var i = 0; i < array.length; i++){
      code_dataset += "      && REGEX( ?dataset_search_space, '" + array[i] + "', 'i' )\n";
    }
    code_dataset += "  ) }}\n";
  }
  if (protein_keywords) {
    var array = StoA(protein_keywords);
  //  code += "  ?protein (dct:description|dct:title|dct:identifier|(jpo:hasDataset/dct:identifier)|(jpo:hasDataset/jpo:hasProfile/jpo:hasSample/rdfs:comment)) ?protein_search_space .\n";	
    code_protein += "  FILTER( true\n"
    for(var i = 0; i < array.length; i++){
      code_protein += "      && REGEX( CONCAT(STR(?sequence),' ',STR(?accession), ' ', STR(?mnemonic), ' ', STR(?full_name), ' ', STR(?gene_name)), '" + array[i] + "', 'i' )\n";
    }
    code_protein += "  )\n";
  }
  if (excluded_proteins) {
    var array = StoA(excluded_proteins);	
    code_protein += "  FILTER( true\n"	
    for(var i = 0; i < array.length; i++){
      code_protein += "      && ?accession != '" + array[i].toUpperCase() + "'\n";
      code_protein += "      && ?mnemonic != '" + array[i].toUpperCase() + "'\n";	
      code_protein += "      && ! REGEX( ?mnemonic, '" + array[i] + "_', 'i')\n";	
    }
    code_protein += "  )\n";
  }  
  if(order || mode == "project" || mode == "dataset"){
    code_limit += "ORDER BY ";
    var scend = "ASC";
    if(desc) scend = "DESC";
    if (mode == "project" && !order) code_limit += scend + "(?project_id)";
    else if (mode == "dataset" && !order) code_limit += scend + "(?dataset_id)";
    else if (order) code_limit += scend + "(?" + order + ") ";
    code_limit += "\n";
  }
  if (mode == "project") {
    code_dataset = code_dataset.replace(/\?dataset jpo:hasProfile/g, "?project jpo:hasDataset/jpo:hasProfile");
  }
  if(parseInt(limit) > 0) code_limit += "LIMIT " + limit + "\n";
  if(parseInt(offset) > 0) code_limit += "OFFSET " + offset + "\n";
  return {
    code_value: code_value, 
    code_protein_value: code_protein_value, 
    code_peptide_value: code_peptide_value, 
    code_up_label_value: code_up_label_value, 
    code_dataset: code_dataset, 
    code_protein: code_protein, 
    code_up_label: code_up_label, 
    code_limit: code_limit
  };
};
```
