# check endpoint and post get_exon_info_main (for Stanza 'proteoform_brpwser')

- Exon, Intron 構造を取得
  - Req. [get_exon_info_main](./get_exon_info_main)

## Parameters

* `uniprot`
  * default: P01112
* `isoform`
  * default: 0
* `endpoint`
  * default: http://togogenome.org/sparql

## `return`

```javascript
async ({uniprot, endpoint})=>{
  const status = await fetch(endpoint).then(res=>res.status);
  //console.log(status);
  if(status == 200){
     const options = {
      method: 'POST',
      body: "uniprot=" + encodeURIComponent(uniprot) + "&endpoint=" + encodeURIComponent(endpoint),
      headers: {
        'Accept': 'application/sparql-results+json',
        'Content-Type': 'application/x-www-form-urlencoded'
      }
    };

    try{
      return await fetch('get_exon_info_main', options).then(res=>res.json());
    }catch(error){
      console.log(error);
    }
  }else{
    return {uniprot: uniprot, isoform: []};
  }
};
```