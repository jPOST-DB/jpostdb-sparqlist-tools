# Develop
- nextprot peptide uniqueness checker wrapper やっつけ（非公開）

## Parameters

* `peptides`
  * example: LQELFLQEVR,TKMGLYYSYFK,SSRAGLQFPVGR,FELSGIPPAPRGVPQIEVTFDIDANGILNVTATDKSTGK

## `nextprot_uniqueness_checker`
```javascript
async ({peptides})=>{
  const fetchReq = async (url, body) => {
    let options = {	
      method: 'POST',
      headers: {
        'Accept': 'application/json',
        'Content-Type': 'application/json'
      }
    }
    if (body) options.body = body; 
    return await fetch(url, options).then(res=>res.json());
  }
  
  const pepArray = peptides.replace(/\s+/g, ",").split(/,/);
  const size = pepArray.length;
  const api = "https://api.nextprot.org/entries/search/peptide-post?no-variant-match=false&mode=J";
  const limit = 1000;
  
  let result1 = [];
  let result2 = [];
  let pep = {};
  for (let i = 0; limit * i <= size; i++){
    let start = i * limit;
    let end = (i + 1) * limit;
    if (end > size) end = size;
    
    let response = await fetchReq(api, JSON.stringify(pepArray.slice(start, end)));

    let id ;
    let name;
    let pe;
    let pe_label;
    
    for (let d of response) {
      id = d.uniqueName;
      console.log(d.overview.proteinNames);
	  let nameList = [];
      for (let n of d.overview.proteinNames) {
        nameList.push(n.value);
      }
      name = nameList.join(";");
      pe = d.overview.proteinExistence.level;
      pe_label = d.overview.proteinExistence.description;
      result2.push(id + "\t" + name + "\tPE" + pe + "\t" + pe_label);

      for (let p of d.annotationsByCategory["pepx-virtual-annotation"]) {
        console.log(p.cvTermName);
        if (!pep[p.cvTermName]) pep[p.cvTermName] = [p.cvTermName];
        pep[p.cvTermName].push("PE" + pe + "," + Object.keys(p.targetingIsoformsMap).join(","));
      }
    }
  }
  
  for (let d of Object.keys(pep)) {
    result1.push(pep[d].join("\t"));
  } 
  return result1.join("\n") + "\n\n" + Array.from(new Set(result2)).join("\n");
}
```

## `return`
```javascript
({
	text({nextprot_uniqueness_checker}){
      return nextprot_uniqueness_checker
    }
})
```
