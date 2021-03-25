# Evolutionary conservation of human genes

## Endpoint

https://orth.dbcls.jp/sparql-dev

## Parameters
* `ncbigene`
  * example: 100,101,102
* `flg_time`
  * example: 1
  * default: 1

## `gene_list`
```javascript
({ncbigene}) => {
  ncbigene = ncbigene.replace(/\s/g,"")
  if (ncbigene) {
    return ncbigene.split(",");
  } else {
    return false;
  }
}
```

## `main`

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX orth: <http://purl.org/net/orth#>
PREFIX homologene: <https://ncbi.nlm.nih.gov/homologene/>
PREFIX taxid: <http://identifiers.org/taxonomy/>
PREFIX ncbigene: <http://identifiers.org/ncbigene/>
PREFIX hop: <http://purl.org/net/orthordf/hOP/ontology#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

{{#if flg_time}}
SELECT ?branch_id ?branch_label ?branch_time_mya (COUNT(?human_gene) AS ?gene_count)
{{else}}
SELECT ?branch_id ?branch_label (COUNT(?human_gene) AS ?gene_count)
{{/if}}
WHERE {
  ?branch a hop:HomoloGeneBranch ;
      dct:identifier ?branch_id ;
      rdfs:label ?branch_label ;
      hop:timeMya ?branch_time_mya .
  {
    SELECT ?human_gene (max(?time) as ?branch_time_mya)
    WHERE {
      {{#if gene_list}}
      VALUES ?human_gene { {{#each gene_list}} ncbigene:{{this}} {{/each}} }
      {{/if}}
      ?grp orth:inDataset homologene: ;
          orth:hasHomologousMember ?human_gene, ?gene .
      ?human_gene orth:taxon taxid:9606 .
      ?gene orth:taxon/hop:branch/hop:timeMya ?time .
    }
  }
}
ORDER BY ?branch_id
```

## `return`

```javascript
({ flg_time, main }) => {
  return main.results.bindings.map((elem) => {
    if (flg_time) {
      return {
        id: elem.branch_id.value,
        label: elem.branch_label.value,
        time_mya: elem.branch_time_mya.value,
        count: elem.gene_count.value
      };
    } else {
      return {
        id: elem.branch_id.value,
        label: elem.branch_label.value,
        count: elem.gene_count.value
      };
    }
  });
};
```