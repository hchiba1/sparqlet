# get ortholog genes from 2 ortholog DBs

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
SELECT DISTINCT ?idup ?idtax
WHERE {
  ?cluster a orth:OrthologsCluster ;
      orth:hasHomologous ?node1 ;
      orth:hasHomologous ?node2 .
  ?node1 orth:hasHomologous* ?query .
  ?node2 orth:hasHomologous* ?ortholog .
  ?query sio:SIO_010078 idup:{{uniprot}} .
  ?ortholog a orth:Gene ;
      sio:SIO_010078 ?idup ;
      obo:RO_0002162 ?idtax .
  FILTER (REGEX (STR (?idup), idup:))
  FILTER(?node1 != ?node2) 
}
```

## Endpoint

https://sparql.omabrowser.org/sparql

## `oma`

```sparql
PREFIX lscr: <http://purl.org/lscr#>
PREFIX orth: <http://purl.org/net/orth#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX up: <http://purl.uniprot.org/uniprot/>
PREFIX uptax: <http://purl.uniprot.org/taxonomy/>

SELECT DISTINCT ?up ?uptax
WHERE {
  ?cluster a orth:OrthologsCluster ;
      orth:hasHomologousMember ?node1 ;
      orth:hasHomologousMember ?node2 .
  ?node1 orth:hasHomologousMember* ?query .
  ?node2 orth:hasHomologousMember* ?ortholog .
  ?query a orth:Protein ;
      lscr:xrefUniprot up:{{uniprot}} .
  ?ortholog a orth:Protein ;
      lscr:xrefUniprot ?up ;
      orth:organism/obo:RO_0002162 ?uptax .
  FILTER(?node1 != ?node2) 
}
```

## `return`

```javascript
({oc, oma})=>{
  let up = {};
  let h_oc = {};
  let h_oma = {};
  let list = oc.results.bindings;
  for(let i = 0; i < list.length; i++){
    h_oc[list[i].idup.value.replace("http://identifiers.org/uniprot/", "")] = 1;
    up[list[i].idup.value.replace("http://identifiers.org/uniprot/", "")] = list[i].idtax.value.replace("http://identifiers.org/taxonomy/", "");
  }
  list = oma.results.bindings;
  for(let i = 0; i < list.length; i++){
    h_oma[list[i].up.value.replace("http://purl.uniprot.org/uniprot/", "")] = 1;
    up[list[i].up.value.replace("http://purl.uniprot.org/uniprot/", "")] = list[i].uptax.value.replace("http://purl.uniprot.org/taxonomy/", "");
  }
  let array = Object.keys(up);
  let r = [];
  for(let i = 0; i < array.length; i++){
	  let obj = { uniprot: array[i],
               taxonomy: up[array[i]],
        	database: {}
  	  };
 	 if(h_oc[array[i]]) obj.database.oc = 1;
 	 if(h_oma[array[i]]) obj.database.oma = 1;
 	 r.push(obj);
  }
  return r;
};
```


