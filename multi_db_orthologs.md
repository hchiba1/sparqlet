# get ortholog genes from 5 ortholog DBs

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
  ?cluster orth:hasHomologous*/sio:SIO_010078 idup:{{uniprot}} ;
      orth:hasHomologous* ?gene .
  ?gene a orth:Gene ;
        sio:SIO_010078 ?idup ;
        obo:RO_0002162 ?idtax .
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
SELECT DISTINCT ?up ?uptax
WHERE {
  ?cluster orth:hasHomologousMember*/lscr:xrefUniprot up:{{uniprot}} ;
           orth:hasHomologous*/orth:hasHomologousMember ?protein .
  ?protein a orth:Protein ;
           lscr:xrefUniprot ?up ;
           orth:organism/obo:RO_0002162 ?uptax .
}
```

## Endpoint

https://sparql.orthodb.org/sparql

## `odb`

```sparql
# @temp-proxy true
PREFIX orthodb: <http://purl.orthodb.org/>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX up: <http://purl.uniprot.org/uniprot/>
PREFIX upcore: <http://purl.uniprot.org/core/>
PREFIX uptax: <http://purl.uniprot.org/taxonomy/>
SELECT DISTINCT ?up ?uptax
WHERE {
  ?cluste orthodb:hasMember/orthodb:xref/orthodb:xrefResource up:{{uniprot}} ;
          orthodb:hasMember ?gene .
  ?gene a orthodb:Gene ;
        orthodb:xref/orthodb:xrefResource ?up ;
        upcore:organism/obo:RO_0002162 ?uptax .
  FILTER (REGEX (STR (?up), up:))
}
```

## Endpoint

http://mbgd.genome.ad.jp:8050/sparql

## `inpara`

```sparql
PREFIX orth: <http://purl.org/net/orth#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX idtax: <http://identifiers.org/taxonomy/>
PREFIX dcterm: <http://purl.org/dc/terms/>
PREFIX sio: <http://semanticscience.org/resource/>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
SELECT DISTINCT ?upac ?idtax
FROM <http://purl.org/net/orthordf/resource/inparanoid>
WHERE {
 ?cluster orth:hasHomologous*/sio:SIO_010078/dcterm:identifier "{{uniprot}}"^^xsd:string ;
          orth:hasHomologous* ?gene .
  ?gene a orth:Gene ;
        sio:SIO_010078/dcterm:identifier ?upac ;
        obo:RO_0002162 ?idtax .
}
```

## Endpoint

http://sparql.nibb.ac.jp/sparql

## `mbgd`

```sparql
PREFIX orth: <http://purl.org/net/orth#>
PREFIX sio: <http://semanticscience.org/resource/>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX up: <http://purl.uniprot.org/uniprot/>
PREFIX idup: <http://identifiers.org/uniprot/>
PREFIX idtax: <http://identifiers.org/taxonomy/>
SELECT DISTINCT ?up ?idtax
WHERE {
  ?cluster orth:hasHomologous*/sio:SIO_010078 up:{{uniprot}} ;
           orth:hasHomologous* ?gene .
  ?gene a orth:Gene ;
        sio:SIO_010078 ?up ;
        obo:RO_0002162 ?idtax .
  FILTER (REGEX (STR (?up), up:))
}
```

## `return`

```javascript
({oc, oma, odb, inpara, mbgd})=>{
  let up = {};
  let h_oc = {};
  let h_oma = {};
  let h_odb = {};
  let h_inpara = {};
  let h_mbgd = {};
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
  list = odb.results.bindings;
  for(let i = 0; i < list.length; i++){
    h_odb[list[i].up.value.replace("http://purl.uniprot.org/uniprot/", "")] = 1;
    up[list[i].up.value.replace("http://purl.uniprot.org/uniprot/", "")] = list[i].uptax.value.replace("http://purl.uniprot.org/taxonomy/", "");
  }
  list = inpara.results.bindings;
  for(let i = 0; i < list.length; i++){
    h_inpara[list[i].upac.value] = 1;
    up[list[i].upac.value] = list[i].idtax.value.replace("http://identifiers.org/taxonomy/", "");
  }
  list = mbgd.results.bindings;
  for(let i = 0; i < list.length; i++){
    h_mbgd[list[i].up.value.replace("http://purl.uniprot.org/uniprot/", "")] = 1;
    up[list[i].up.value.replace("http://purl.uniprot.org/uniprot/", "")] = list[i].idtax.value.replace("http://identifiers.org/taxonomy/", "");
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
 	 if(h_odb[array[i]]) obj.database.orthodb = 1;
 	 if(h_inpara[array[i]]) obj.database.inpara = 1;
 	 if(h_mbgd[array[i]]) obj.database.mbgd = 1;
 	 r.push(obj);
  }
  return r;
};
```


