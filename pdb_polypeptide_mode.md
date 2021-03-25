# PDBエントリをポリペプチドの数で分類(mode対応版)（井手）

## Parameters

* `categoryId` -(type:エントリーに含まれるポリペプチドの数)
  * example: 1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36,37,38,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,58,60,61,62,63,64,66,67,68,70,71,73,74,75,76,77,78,79,80,81,82,125
* `queryIds` -(type: PDB)
  * example: 6TIW,6E7C
* `mode` 
  * example: idList, objectList

## Endpoint

https://integbio.jp/rdf/sparql

## `input_numbers`
```javascript
({ categoryId }) => {
  categoryId = categoryId.replace(/,/g, " ")
  if (categoryId.match(/\S/)) {
    return categoryId.split(/\s+/);
  }
}
```

## `input_pdb_entries`
```javascript
({ queryIds }) => {
  queryIds = queryIds.replace(/,/g, " ");
  if (queryIds.match(/\S/)) {
    return queryIds.split(/\s+/);
  }
}
```

## `main`

```sparql
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX pdbo: <https://rdf.wwpdb.org/schema/pdbx-v50.owl#>
PREFIX pdbr: <https://rdf.wwpdb.org/pdb/>

{{#if mode}}
SELECT DISTINCT ?peptides_count ?PDBentry ?title
{{else}}
SELECT ?peptides_count (COUNT(?peptides_count) AS ?count)
{{/if}}
WHERE {
  {{#if input_numbers}}
  VALUES ?peptides_count { {{#each input_numbers}} {{this}} {{/each}} }
  {{/if}}
  {
    SELECT (COUNT(?polypeptide) AS ?peptides_count) ?PDBentry {{#if mode}} ?title {{/if}}
    WHERE {
      {{#if input_pdb_entries}}
      VALUES ?PDBentry { {{#each input_pdb_entries}} pdbr:{{this}} {{/each}} }
      {{/if}}
      ?PDBentry a pdbo:datablock ;
          dc:title ?title ;
          pdbo:has_entity_polyCategory ?polypeptideEntity .
      ?polypeptideEntity pdbo:has_entity_poly ?polypeptide .
      ?polypeptide rdfs:seeAlso ?uniprot_link . #DNAのentryを排除
    }
  }
} 
{{#unless mode}}
ORDER BY (?peptides_count)  
{{/unless}}
```

## `return`

```javascript
({ main, mode }) => {
  if (mode == "idList") {
    return Array.from(new Set(
      main.results.bindings.map((d) => d.PDBentry.value.replace("https://rdf.wwpdb.org/pdb/", ""))
    ));
  } else if (mode == "objectList") {
    return main.results.bindings.map((d) => ({
      id: d.PDBentry.value.replace("https://rdf.wwpdb.org/pdb/", ""), 
      attribute: {
        categoryId: d.peptides_count.value, 
        label: makeLabel(d.peptides_count.value)
      }
    }));
  } else {
    return main.results.bindings.map((d) => ({
      categoryId: d.peptides_count.value,
      label: makeLabel(d.peptides_count.value),
      count: d.count.value
    }));
  }

  function makeLabel(number) {
    if (number == 1) {
      return `${number} peptide`;
    } else if (number >= 2) {
      return `${number} peptides`;
    } else {
      return number;
    }
  }
}
```
