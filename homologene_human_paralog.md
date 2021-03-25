# Number of human genes in each ortholog group

See also [homologene_human_paralog_count](homologene_human_paralog_count) to get summary table.

## Endpoint

https://orth.dbcls.jp/sparql-proxy

## `homologene`

```sparql
PREFIX orth: <http://purl.org/net/orth#>
PREFIX homologene: <https://ncbi.nlm.nih.gov/homologene/>
PREFIX taxid: <http://identifiers.org/taxonomy/>

SELECT ?grp (COUNT(DISTINCT ?human_gene) AS ?gene_count)
WHERE {
  ?grp orth:inDataset homologene: ;
      orth:hasHomologousMember ?human_gene .
  ?human_gene orth:taxon taxid:9606 .
}
ORDER BY DESC(?gene_count)
```

## `return`

```javascript
({homologene})=>{
  let r = [];
  
  homologene.results.bindings.forEach(elem => {
    r.push({
      group_id : elem.grp.value,
      gene_count: elem.gene_count.value});
  });
  
  return r;
};
```