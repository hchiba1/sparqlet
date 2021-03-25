# List organisms in HomoloGene

## Endpoint

https://orth.dbcls.jp/sparql

## `homologene`

```sparql
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX orth: <http://purl.org/net/orth#>
PREFIX homologene: <https://ncbi.nlm.nih.gov/homologene/>

SELECT DISTINCT ?taxid ?id ?label
WHERE {
  GRAPH homologene: {
    ?s a orth:Gene .
    ?s orth:taxon ?taxid .
  }
  ?taxid rdfs:label ?label .
  ?taxid dct:identifier ?id .
}
```

## `return`

```javascript
({homologene})=>{
  let r = [];
  
  homologene.results.bindings.forEach(elem => {
    r.push({taxid: elem.taxid.value,
            id: elem.id.value,
            label: elem.label.value});
  });
  
  return r;
};
```
