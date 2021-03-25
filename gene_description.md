# Ortholog Genes

## Endpoint
https://sparql.orth.dbcls.jp/sparql

## `result` Image URLs
```sparql
PREFIX orth: <http://purl.jp/bio/11/orth#> 
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
SELECT distinct ?label from <http://purl.org/net/orthordf/hOP> where {
    ?gene_id a orth:Gene ;
        rdfs:label ?label .
}
```

## Output
```javascript
({
   json({result}) {
       var array = [];
       result.results.bindings.forEach(
           function( element ) {
               array.push( element.label.value );
           }
       );
       return array;
   }
})
```