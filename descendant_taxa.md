# get descendant taxa

## Endpoint
https://orth.dbcls.jp/sparql

## result
```
PREFIX taxon: <http://ddbj.nig.ac.jp/ontologies/taxonomy/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX taxid: <http://identifiers.org/taxonomy/>

SELECT ?order ?order_name ?family ?family_name ?species ?species_name
WHERE {
  ?species taxon:rank taxon:Species ;
      rdfs:label ?species_name ;
      rdfs:subClassOf+ taxid:40674 ;
      rdfs:subClassOf+ ?family ;
      rdfs:subClassOf+ ?order .
  ?family taxon:rank taxon:Family ;
      rdfs:label ?family_name .
  ?order taxon:rank taxon:Order ;
      rdfs:label ?order_name .
}
ORDER BY ?order_name ?family_name ?species_name
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
