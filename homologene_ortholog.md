# get orthologs from HomoloGene

## Parameters

* `gene`
  * default: 34


## Endpoint

https://orth.dbcls.jp/sparql

## `homologene`

```sparql
PREFIX orth: <http://purl.org/net/orth#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX ncbigene: <http://identifiers.org/ncbigene/>

SELECT ?tax_label ?taxid ?gene ?protein
WHERE {
 GRAPH <https://ncbi.nlm.nih.gov/homologene/> {
   ?grp a orth:OrthologsCluster ;
       orth:hasHomologousMember ncbigene:{{gene}} ;
       orth:hasHomologousMember ?gene .
   ?gene orth:taxon ?taxid ;
       orth:protein ?protein .
 }
 ?taxid rdfs:label ?tax_label .
}
```

## `return`

```javascript
({homologene})=>{
  let r = [];
  
  homologene.results.bindings.forEach(elem => {
    r.push({organism: elem.tax_label.value, 
            taxid: elem.taxid.value,
            gene: elem.gene.value,
           protein: elem.protein.value});
  });
  
  return r;
};
```