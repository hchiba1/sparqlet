# Genes expressed in specific tissues in RefEx ver1

This query allows **multiple tissue flags** for each gene.

## Endpoint

https://orth.dbcls.jp/sparql-dev

## Parameters
* `categoryIds` (type: tissue)
  * example: v01_40,v02_40
* `queryIds` (type: ncbigene)
  * example: 105,107,108
* `mode`
  * example: idList, objectList

## `input_genes`
```javascript
({ queryIds }) => {
  queryIds = queryIds.replace(/,/g, " ");
  if (queryIds.match(/\S/)) {
    return queryIds.split(/\s+/);
  } 
};
```

## `input_tissues`
```javascript
({ categoryIds }) => {
  categoryIds = categoryIds.replace(/,/g, " ");
  if (categoryIds.match(/\S/)) {
    return categoryIds.split(/\s+/);
  }
};
```

## `main`

```sparql
PREFIX refexo: <http://purl.jp/bio/01/refexo#>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX ncbigene: <http://identifiers.org/ncbigene/>

{{#if mode}}
SELECT DISTINCT ?tissue ?label ?gene
{{else}}
SELECT ?tissue ?label (COUNT(DISTINCT ?gene) AS ?gene_count)
{{/if}}
WHERE {
  {{#if input_tissues}}
  VALUES ?tissue { {{#each input_tissues}} refexo:{{this}} {{/each}} }
  {{/if}}
  {{#if input_genes}}
  VALUES ?gene { {{#each input_genes}} ncbigene:{{this}} {{/each}} }
  {{/if}}
  ?gene refexo:affyProbeset ?seq .
  ?seq refexo:isPositivelySpecificTo ?tissue .
  ?tissue skos:prefLabel ?label .
}
{{#unless mode}}
ORDER BY ?tissue
{{/unless}}
```

## `return`

```javascript
({ main, mode })=>{
  if (mode === "idList") {
    return Array.from(new Set(
      main.results.bindings.map((elem) =>
        elem.gene.value.replace("http://identifiers.org/ncbigene/", "")
      )
    ));
  } else if (mode === "objectList") {
    return main.results.bindings.map((elem) => ({
      id: elem.gene.value.replace("http://identifiers.org/ncbigene/", ""),
      attribute: {
        categoryId: elem.tissue.value.replace("http://purl.jp/bio/01/refexo#", ""),
        uri: elem.tissue.value,
        label: elem.label.value
      }
    }));
  } else {
    return main.results.bindings.map((elem) => ({
      categoryId: elem.tissue.value.replace("http://purl.jp/bio/01/refexo#", ""),
      label: elem.label.value,
      count: elem.gene_count.value
    }));
  }
};
```