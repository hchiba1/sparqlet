# Number of human genes in each ortholog group（千葉）

See also [homologene_human_paralog](homologene_human_paralog) to get the list of HomoloGene Group IDs.

## Parameters
* `qureyIds` (type: ncbigene)
  * example: 100,101,102

## `gene_list`
```javascript
({ qureyIds }) => {
  qureyIds = qureyIds.replace(/,/g, " ");
  if (qureyIds.match(/[^\s]/)) {
    return qureyIds.split(/\s+/);
  } else {
    return false;
  }
};
```

## Endpoint

https://orth.dbcls.jp/sparql-dev

## `homologene`

```sparql
PREFIX orth: <http://purl.org/net/orth#>
PREFIX homologene: <https://ncbi.nlm.nih.gov/homologene/>
PREFIX taxid: <http://identifiers.org/taxonomy/>
PREFIX ncbigene: <http://identifiers.org/ncbigene/>
SELECT ?gene_count (COUNT(?grp) AS ?grp_count)
WHERE {
  {
    SELECT ?grp (COUNT(DISTINCT ?human_gene) AS ?gene_count)
    WHERE {
      {{#if gene_list}}
      VALUES ?human_gene { {{#each gene_list}} ncbigene:{{this}} {{/each}} }
      {{/if}}
      ?grp orth:inDataset homologene: ;
          orth:hasHomologousMember ?human_gene .
      ?human_gene orth:taxon taxid:9606 .
    }
  }
}
ORDER BY DESC(?gene_count)
```

## `return`

```javascript
({homologene})=>{
  let r = [];
  
  homologene.results.bindings.forEach(elem => {
    r.push({
      categoryId : elem.gene_count.value,
      label : elem.gene_count.value,
      count: elem.grp_count.value});
  });
  
  return r;
};
```