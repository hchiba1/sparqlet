# Number of ortholog groups each gene belogs to（千葉）

## Endpoint

https://orth.dbcls.jp/sparql-dev

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

## `hop`

```sparql
PREFIX orth: <http://purl.org/net/orth#>
PREFIX ncbigene: <http://identifiers.org/ncbigene/>

SELECT ?gene (COUNT(DISTINCT ?grp) AS ?group_count)
WHERE {
  {
    SELECT ?gene ?grp
    WHERE {
      {{#if gene_list}}
      VALUES ?gene { {{#each gene_list}} ncbigene:{{this}} {{/each}} }
      {{/if}}
      ?grp orth:inDataset <http://purl.org/net/orthordf/hOP/> ;
          orth:hasHomologousMember ?gene .
    }
  }
}
ORDER BY DESC(?group_count)
```

## `return`

```javascript
({hop})=>{
  let r = [];
  
  hop.results.bindings.forEach(elem => {
    r.push({
      gene: elem.gene.value,
      group_count: elem.group_count.value});
  });
  
  return r;
};
```