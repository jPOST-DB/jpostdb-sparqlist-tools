# check endpoint and post get_exon_info_main (for Stanza 'proteoform_brpwser')

## Parameters

* `uniprot`
  * default: P01112
* `isoform`
  * default: 0

## `return`

```javascript
async ({uniprot, isoform})=>{
  var status = await fetch('https://integbio.jp/rdf/ebi/sparql').then(res=>res.status);
  console.log(status);
  if(status == 200){
     const options = {
      method: 'POST',
      body: "uniprot=" + encodeURIComponent(uniprot) + "&isoform=" + isoform,
      headers: {
        'Accept': 'application/sparql-results+json',
        'Content-Type': 'application/x-www-form-urlencoded'
      }
    };

    try{
      var res = await fetch('https://db-dev.jpostdb.org/rest/api/dev_get_exon_info_main', options).then(res=>res.json());
      return res;
    }catch(error){
      console.log(error);
    }
  }else{
    return {uniprot: uniprot, isoform: []};
  }
};
```