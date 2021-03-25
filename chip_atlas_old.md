# Genes with hypothetical upstream TFs in ChIP-Atlas（大田・池田・小野・千葉）

## Endpoint

https://orth.dbcls.jp/sparql-dev

## Parameters
* `queryIds` (type: ensemble gene)
  * example: ENSG00000000005,ENSG00000002587,ENSG00000115942

## `gene_list`
```javascript
({ queryIds }) => {
  queryIds = queryIds.replace(/,/g, " ");
  if (queryIds.match(/[^\s]/)) {
    return queryIds.split(/\s+/);
  } else {
    return false;
  }
};
```

## `main`

```sparql
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX refexo: <http://purl.jp/bio/01/refexo#>
PREFIX ensembl: <http://identifiers.org/ensembl/>

SELECT ?downstream_count ?remaining_count
WHERE {
  {
    SELECT (COUNT(DISTINCT ?ensg) AS ?downstream_count)
    WHERE {
      {{#if gene_list}}
      VALUES ?ensg { {{#each gene_list}} ensembl:{{this}} {{/each}} }
      {{/if}}
      ?upstream obo:RO_0002428 ?ensg .
    }
  }
  {
    SELECT (COUNT(DISTINCT ?ensg) AS ?remaining_count)
    WHERE {
      {{#if gene_list}}
      VALUES ?ensg { {{#each gene_list}} ensembl:{{this}} {{/each}} }
      {{/if}}
      ?ensg refexo:ncbigene ?ncbigene .
      FILTER NOT EXISTS {
        ?upstream obo:RO_0002428 ?ensg .
      }
    }
  }
}
```

## `return`

```javascript
({ main })=>{
  const result = main.results.bindings[0]
  return [
    {
      categoryId: "1",
      label: "with hypothetical upstream TF",
      count: result.downstream_count.value
    },
    {
      categoryId: "2",
      label: "without hypothetical upstream TF",
      count: result.remaining_count.value
    }
  ];
};
```