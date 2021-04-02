# NCBI Gene Summary（千葉）

## Endpoint

https://orth.dbcls.jp/sparql-dev

## Parameters
* `ncbigene`
  * default: 1
  * example: 1

## `gene_list`
```javascript
({ ncbigene }) => {
  ncbigene = ncbigene.replace(/\s/g, "");
  if (ncbigene) {
    return ncbigene.split(",");
  } else {
    return false;
  }
};
```

## `main`

```sparql
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX ncbigene: <http://identifiers.org/ncbigene/>
PREFIX nuc: <http://ddbj.nig.ac.jp/ontologies/nucleotide/>
PREFIX hop: <http://purl.org/net/orthordf/hOP/ontology#>

SELECT ?id ?hgnc_symbol ?description ?type_of_gene ?chromosome ?map_location (GROUP_CONCAT(DISTINCT ?synonym; separator="|") AS ?synonyms) (GROUP_CONCAT(DISTINCT ?others; separator="|") AS ?other_descriptions)
WHERE {
  {{#if gene_list}}
  VALUES ?human_gene { {{#each gene_list}} ncbigene:{{this}} {{/each}} }
  {{/if}}
  ?human_gene dct:description ?description ;
      dct:identifier ?id ;
      hop:typeOfGene ?type_of_gene ;
      nuc:chromosome ?chromosome ;
      nuc:map ?map_location .
  OPTIONAL {
    ?human_gene nuc:standard_name ?hgnc_symbol .
  }
  OPTIONAL {
    ?human_gene nuc:gene_synonym ?synonym .
  }
  OPTIONAL {
    ?human_gene dct:alternative ?others .
  }
}
```

## `return`

```javascript
({ main }) => {
  return main.results.bindings.map((elem) => ({
    id: elem.id.value,
    type_of_gene: elem.type_of_gene.value,
    hgnc_symbol: elem.hgnc_symbol.value,
    synonyms: elem.synonyms.value,
    description: elem.description.value,
    other_descriptions: elem.other_descriptions.value,
    chromosome: elem.chromosome.value,
    map_location: elem.map_location.value,
  }));
};
```
