# Ensembl transcript description（井手）

## Endpoint

https://integbio.jp/rdf/ebi/sparql

## Parameters

* `categoryId` -(type:種別/機能)
  * example: protein_coding, protein, lncRNA, retained_intron, processed_transcript, processed_pseudogene, nonsense_mediated_decay, unprocessed_pseudogene, misc_RNA, snRNA, miRNA, TEC, transcribed_unprocessed_pseudogene, snoRNA, LRG_gene, transcribed_processed_pseudogene, rRNA_pseudogene, IG_V_pseudogene, IG_V_gene, TR_V_gene, transcribed_unitary_pseudogene, unitary_pseudogene, TR_J_gene, polymorphic_pseudogene, rRNA, IG_D_gene, scaRNA, non_stop_decay, pseudogene, TR_V_pseudogene, IG_C_gene, IG_J_gene, Mt_tRNA, IG_C_pseudogene, TR_C_gene, ribozyme, IG_J_pseudogene, sRNA, TR_D_gene, TR_J_pseudogene, translated_processed_pseudogene, Mt_rRNA, IG_pseudogene, vault_RNA, scRNA, translated_unprocessed_pseudogene
* `queryIds` -(type: Ensembl)
  * example: ENST00000460472
* `mode` 
  * example: idList, objectList

## `gene_list`
```javascript
({ queryIds }) => {
  queryIds = queryIds.replace(/\s/g, "");
  if (queryIds) {
    return queryIds.split(",");
  } else {
    return false;
  }
};
```

## `category_list`
- categoryId を配列に
```javascript
({categoryId}) => {
  categoryId = categoryId.replace(/,/g," ")
  if (categoryId.match(/[^\s]/)) return categoryId.split(/\s+/);
  return false;
}
```

## `main`

```sparql
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX taxon: <http://identifiers.org/taxonomy/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX enst: <http://rdf.ebi.ac.uk/resource/ensembl.transcript/>
PREFIX dc: <http://purl.org/dc/elements/1.1/>

{{#if mode}}
SELECT DISTINCT ?description ?id ?gene_name ?gene_symbol
{{else}}
SELECT DISTINCT ?description (COUNT(?des) AS ?count)
{{/if}}
WHERE {
  {{#if gene_list}}
  VALUES ?entry { {{#each gene_list}} enst:{{this}} {{/each}} }
  {{/if}}
  ?entry a ?des .
  ?entry obo:RO_0002162 taxon:9606 .  
  filter contains(STR(?des), "terms/ensembl/")
  {{#if mode}}   
  ?entry rdfs:label ?gene_symbol .
  ?entry obo:SO_transcribed_from/dc:description ?gene_name .
  {{/if}}  
  BIND(REPLACE(STR(?entry), enst:, "") AS ?id)
  BIND(REPLACE(STR(?des), "http://rdf.ebi.ac.uk/terms/ensembl/", "") AS ?description)
  {{#if category_list}}
  VALUES ?description { {{#each category_list}} "{{this}}" {{/each}} } 
 {{/if}}
}
Order by DESC(?count)
                   
```

## `return`

```javascript
({mode, main }) => {
  if (mode == "idList") {
    return Array.from(new Set(
      main.results.bindings.map((d) => d.id.value.replace("https://rdf.wwpdb.org/pdb/", ""));
    ));
  } else if (mode == "objectList") {
    return main.results.bindings.map(d=>{ 
      return {
        id: d.id.value, 
        attribute: {
          gene_name: d.gene_name.value, 
          gene_symbol: d.gene_symbol.value, 
          description: d.description.value
        }
      };
    });
  } else {
    var id_val = 0;
    return main.results.bindings.map(d=>{ 
      return {
        categoryId: id_val++, 
        description: d.description.value, 
        count: d.count.value
      };
    });
  }
};
```
