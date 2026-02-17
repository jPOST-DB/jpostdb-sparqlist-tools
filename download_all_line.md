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
  var options = {
    method: 'POST',
    body: "count=1&dataset=" + dataset,
    headers: {
        'Content-Type': 'application/x-www-form-urlencoded',
        'Accept': 'application/json'
    }
  }
  var url = 'https://db-dev.jpostdb.org/rest/api/' + sparqlist;
  var count = await fetch(url, options).then(res=>res.json()).then(res=>res.results.bindings[0].count.value);
  console.log("count: " + count);
  var loop = Math.ceil(parseInt(count) / parseInt(limit));
  var res = [];
  console.log("loop: " + loop);
  for(var i = 0; i < loop; i++){
    var offset = parseInt(limit) * i;
    options.body = "dataset=" + dataset +"&limit=" + limit + "&offset=" + offset;
    res[i] = fetch(url, options).then(res=>res.json());
  }
  var data = Promise.all(res);
  return data.then(function(res){
    var sum = res[0];
    for(var i = 1; i < loop; i++){   
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
    var vars = loop.head.vars;
    var list = loop.results.bindings;
    var text = vars.join("\t") + "\n";
    for(var i = 0; i < list.length; i++){
      var values = [];
      for(var j = 0; j < vars.length; j++){
        var val = ""; 
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
