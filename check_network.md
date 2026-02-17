# check network connection

- サイト(endpoint)の死活を確認する

## Parameters

* `url` (Req.)
  * example: http://togogenome.org/sparql

## `fetch`

```javascript
async ({url})=>{
return {status: await fetch(decodeURIComponent(url)).then(res=>res.status)};
};
```