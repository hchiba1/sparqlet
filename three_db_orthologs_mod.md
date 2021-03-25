# get ortholog genes from 3 ortholog DBs mod.

## Parameters

* `uniprot`
  * default: Q96C23


## Endpoint

https://www.genome.jp/oc/proxy/sparql

## `oc`

```sparql
PREFIX orth: <http://purl.org/net/orth#>
PREFIX sio: <http://semanticscience.org/resource/>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX idup: <http://identifiers.org/uniprot/>
PREFIX idtax: <http://identifiers.org/taxonomy/>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
SELECT DISTINCT ?idup ?idtax ?label
WHERE {
  ?cluster orth:hasHomologous*/sio:SIO_010078 idup:{{uniprot}} ;
      orth:hasHomologous* ?gene .
  ?gene a orth:Gene ;
        sio:SIO_010078 ?idup ;
        obo:RO_0002162 ?idtax .
  ?idtax rdfs:seeAlso/rdfs:label ?label .
  FILTER (REGEX (STR (?idup), idup:))
}
```

## Endpoint

https://sparql.omabrowser.org/lode/sparql

## `oma`

```sparql
PREFIX lscr: <http://purl.org/lscr#>
PREFIX orth: <http://purl.org/net/orth#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX up: <http://purl.uniprot.org/uniprot/>
PREFIX uptax: <http://purl.uniprot.org/taxonomy/>
PREFIX upcore: <http://purl.uniprot.org/core/>
SELECT DISTINCT ?up ?uptax ?label 
WHERE {
  ?cluster orth:hasHomologousMember*/lscr:xrefUniprot up:{{uniprot}} ;
           orth:hasHomologous*/orth:hasHomologousMember ?protein .
  ?protein a orth:Protein ;
           lscr:xrefUniprot ?up ;
           orth:organism/obo:RO_0002162 ?uptax .
  ?uptax upcore:scientificName ?label .
}
```

## Endpoint

https://sparql.orthodb.org/sparql

## `odb`

```sparql
PREFIX orthodb: <http://purl.orthodb.org/>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX up: <http://purl.uniprot.org/uniprot/>
PREFIX upcore: <http://purl.uniprot.org/core/>
PREFIX uptax: <http://purl.uniprot.org/taxonomy/>
SELECT DISTINCT ?up ?uptax ?label
WHERE {
  ?cluste orthodb:hasMember/orthodb:xref/orthodb:xrefResource up:{{uniprot}} ;
          orthodb:hasMember ?gene .
  ?gene a orthodb:Gene ;
        orthodb:xref/orthodb:xrefResource ?up ;
        upcore:organism/obo:RO_0002162 ?uptax .
  ?uptax upcore:scientificName ?label .
  FILTER (REGEX (STR (?up), up:))
}
```

## `return`

```javascript
({oc, oma, odb})=>{
  let up = {};
  let tax = {};
  let h_oc = {};
  let h_oma = {};
  let h_odb = {};
  for(let elm of oc.results.bindings){
    let acc = elm.idup.value.replace("http://identifiers.org/uniprot/", "");
   	let tnum = elm.idtax.value.replace("http://identifiers.org/taxonomy/", "");
    h_oc[acc] = 1;
    if(!up[acc]) up[acc] = tnum;
	if(!tax[tnum]) tax[tnum] = elm.label.value;
  }
  for(let elm of oma.results.bindings){
    let acc = elm.up.value.replace("http://purl.uniprot.org/uniprot/", "");
   	let tnum = elm.uptax.value.replace("http://purl.uniprot.org/taxonomy/", "");
    h_oma[acc] = 1;
    if(!up[acc]) up[acc] = tnum;
    if(!tax[tnum]) tax[tnum] = elm.label.value;
  }
  for(let elm of odb.results.bindings){
    let acc = elm.up.value.replace("http://purl.uniprot.org/uniprot/", "");
   	let tnum = elm.uptax.value.replace("http://purl.uniprot.org/taxonomy/", "");
    h_odb[acc] = 1;
    if(!up[acc]) up[acc] = tnum;
    if(!tax[tnum]) tax[tnum] = elm.label.value;
  }
  let array = Object.keys(up);
  let r = [];
  for(let i = 0; i < array.length; i++){
	  let obj = { uniprot: array[i],
               taxonomy: up[array[i]],
                 label: tax[up[array[i]]],
        	database: {}
  	  };
 	 if(h_oc[array[i]]) obj.database.oc = 1;
 	 if(h_oma[array[i]]) obj.database.oma = 1;
 	 if(h_odb[array[i]]) obj.database.orthodb = 1;
 	 r.push(obj);
  }
  return r;
};
```
