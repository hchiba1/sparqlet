# Evolutionary conservation of human tissue specific genes

- まずRefEx ver1を使って、組織特異的に高発現している遺伝子のリストを取得する (例: v24_40 "精巣")
- 次にHomoloGeneを使って、オーソログを保持している生物を取得し、それら分岐年代で分類する

## Endpoint

https://orth.dbcls.jp/sparql-dev

## Parameters

* `specific_tissue`
  * default: v24_40
  * example: v24_40

## `homologene`

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX orth: <http://purl.org/net/orth#>
PREFIX homologene: <https://ncbi.nlm.nih.gov/homologene/>
PREFIX taxid: <http://identifiers.org/taxonomy/>
PREFIX hop: <http://purl.org/net/orthordf/hOP/ontology#>
PREFIX refexo: <http://purl.jp/bio/01/refexo#>

SELECT ?branch_label ?branch_time_mya (COUNT(?human_gene) AS ?gene_count)
WHERE {
  ?branch a hop:HomoloGeneBranch ;
      rdfs:label ?branch_label ;
      hop:timeMya ?branch_time_mya .
  {
    SELECT ?human_gene (max(?time) as ?branch_time_mya)
    WHERE {
      ?grp orth:inDataset homologene: ;
          orth:hasHomologousMember ?human_gene, ?gene .
      ?human_gene orth:taxon taxid:9606 .
      ?gene orth:taxon/hop:branch/hop:timeMya ?time .
      {
        SELECT DISTINCT ?human_gene (GROUP_CONCAT(DISTINCT ?tissue; separator=",") AS ?tissues)
        WHERE {
          ?human_gene refexo:affyProbeset/refexo:isPositivelySpecificTo refexo:{{specific_tissue}} , ?tissue .
        }
      }
      FILTER (?tissues = str(refexo:{{specific_tissue}}))
    }
  }
}
ORDER BY ?branch_time_mya
```

## `return`

```javascript
({homologene})=>{
  let r = [];
  
  homologene.results.bindings.forEach(elem => {
    r.push({
      label : elem.branch_label.value,
      time_mya : elem.branch_time_mya.value,
      gene_count: elem.gene_count.value});
  });
  
  return r;
};
```