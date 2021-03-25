# Genes expressed in specific tissues in RefEx ver1
This query allows **multiple tissue flags** for each gene.

For the other version of query which focuses on genes with a specific tissue flag, see the other query [refex_tissue_specific](refex_tissue_specific).

## Endpoint

https://orth.dbcls.jp/sparql-dev

## `refex`

```sparql
PREFIX refexo: <http://purl.jp/bio/01/refexo#>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>

SELECT ?tissue ?label (COUNT(DISTINCT ?gene) AS ?gene_count)
WHERE {
  ?gene refexo:affyProbeset ?seq .
  ?seq refexo:isPositivelySpecificTo ?tissue .
  ?tissue skos:prefLabel ?label .
}
ORDER BY ?tissue
```

## `return`

```javascript
({refex})=>{
  let r = [];
  
  refex.results.bindings.forEach(elem => {
    r.push({
      refex_tissue : elem.tissue.value,
      label : elem.label.value,
      gene_count: elem.gene_count.value});
  });
  
  return r;
};
```