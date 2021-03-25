# Evolutionary conservation patterns of human genes（千葉）

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

## `main`

```sparql
PREFIX orth: <http://purl.org/net/orth#>
PREFIX hop: <http://purl.org/net/orthordf/hOP/ontology#>
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX ncbigene: <http://identifiers.org/ncbigene/>

SELECT (REPLACE(?cat, "[a-m]_", "") AS ?category) (COUNT(DISTINCT ?gene) AS ?count)
WHERE {
  {
    SELECT ?gene ?cat
    WHERE {
      {
        SELECT ?grp ?gene (max(?org_id) as ?max_id)
        WHERE {
          {{#if gene_list}}
          VALUES ?gene { {{#each gene_list}} ncbigene:{{this}} {{/each}} }
          {{/if}}
          ?grp orth:hasHomologousMember ?gene ;
              orth:organism ?org .
          ?org dct:identifier ?org_id .
        }
      }
      BIND(IF(?max_id<=25, "a_M",
           IF(?max_id<=34, "b_V",
           IF(?max_id<=37, "c_LT",
           IF(?max_id<=41, "d_EH",
           IF(?max_id<=54, "e_A",
           IF(?max_id<=60, "f_N",
           IF(?max_id<=62, "g_C",
           IF(?max_id<=64, "h_SP",
           IF(?max_id<=67, "i_CF",
           IF(?max_id<=124, "j_F",
           IF(?max_id<=129, "k_AB",
           IF(?max_id<=147, "l_P",
             "m_O"))))))))))))
        AS ?cat)
    }
  }
}
ORDER BY ?cat
```

## `return`

```javascript
({ main }) => {
  return main.results.bindings.map((elem) => ({
    label: elem.category.value,
    count: elem.count.value,
  }));
};
```