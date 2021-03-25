# Genes expressed in a specific tissue in RefEx ver1
This query focuses on genes with one tissue flag.

For the other version of query that allows multiple tissue flags for each genes, see the other query [refex](refex).

## Endpoint

https://orth.dbcls.jp/sparql-dev

## `refex`

```sparql
PREFIX refexo: <http://purl.jp/bio/01/refexo#>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>

SELECT ?specific_tissue ?label ?gene_count
WHERE {
  ?specific_tissue a refexo:AnatomicalClassification40 ;
     skos:prefLabel ?label .
  {
    SELECT ?specific_tissue (COUNT(?gene) AS ?gene_count)
    WHERE {
      {
        SELECT DISTINCT ?specific_tissue ?gene (GROUP_CONCAT(DISTINCT ?tissue; separator=",") AS ?tissues)
        WHERE {
          ?gene refexo:affyProbeset/refexo:isPositivelySpecificTo ?specific_tissue , ?tissue .
        }
      }
      FILTER (?tissues = str(?specific_tissue))
    }
  }
}
ORDER BY ?specific_tissue
```

## `return`

```javascript
({refex})=>{
  let r = [];
  
  refex.results.bindings.forEach(elem => {
    r.push({
      refex_tissue : elem.specific_tissue.value,
      label : elem.label.value,
      gene_count: elem.gene_count.value});
  });
  
  return r;
};
```