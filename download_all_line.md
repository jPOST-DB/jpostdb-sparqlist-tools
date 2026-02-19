# download all line

データサイズが大きくなるとエラーになるので、perl で書かれた API の 'http://tools.jpostdb.org/api/marge' を使う。
パラメータは同じで、'format=text' を足す

## Parameters

* `dataset`
  * default: DS1631_1
* `limit`
  * default: 100000
* `sparqlist`
  * default: download_protein_sparql

## `loop`

```javascript
async ({dataset, limit, sparqlist})=>{
  let options = {
    method: 'POST',
    body: "count=1&dataset=" + dataset,
    headers: {
        'Content-Type': 'application/x-www-form-urlencoded',
        'Accept': 'application/json'
    }
  }

  const count = await fetch(sparqlist, options).then(res=>res.json()).then(res=>res.results.bindings[0].count.value);

  const loop = Math.ceil(parseInt(count) / parseInt(limit));
  let res = [];

  for (let i = 0; i < loop; i++){
    const offset = parseInt(limit) * i;
    options.body = "dataset=" + dataset +"&limit=" + limit + "&offset=" + offset;
    res[i] = fetch(sparqlist, options).then(res=>res.json());
  }
  const data = Promise.all(res);
  return data.then(function(res){
    const sum = res[0];
    for(let i = 1; i < loop; i++){   
      Array.prototype.push.apply(sum.results.bindings, res[i].results.bindings);
    }
    return sum;
  });
};
```

## `return`

```javascript
({
  json({loop}){
    return loop;
  },
  text({loop}){
    const vars = loop.head.vars;
    const list = loop.results.bindings;
    let text = vars.join("\t") + "\n";
    for(let i = 0; i < list.length; i++){
      let values = [];
      for(let j = 0; j < vars.length; j++){
        let val = ""; 
        if(list[i][vars[j]]) val = list[i][vars[j]].value;
        if(val.match(/^\".+\"$/)) val = val.match(/^\"(.+)\"$/)[1];
        values.push(val);
      }
      text += values.join("\t") + "\n";
    }
    return text;
  }
})
```
