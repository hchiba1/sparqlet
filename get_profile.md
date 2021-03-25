# Get ortholog profile of a human gene

## Parameters
* `gene` Gene ID
  * default: TTC21B
  * examples: TTC21B

## Endpoint
https://sparql.orth.dbcls.jp/sparql

## `result` Image URLs
``` sparql
PREFIX orth: <http://purl.jp/bio/11/orth#>
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX hop: <http://purl.org/net/orthordf/hOP/ontology#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

select distinct ?gene_label ?common_id ?scientific_name ?common_name ?comment ?time where {
    values ?gene_label { '{{gene}}' } .
    ?gene rdfs:label ?gene_label .
    ?orthogroup a orth:OrthologGroup ;
           dct:identifier ?orthogroup_id ;
           orth:member ?gene ;
           orth:organism ?organism .
    ?organism rdfs:label ?scientific_name ;
           dct:identifier ?common_id ;
           dct:description ?common_name ;
           rdfs:comment ?comment ;
           hop:branchTimeMya ?time .
}
order by ?common_id
```

## Output
``` javascript
({
    json( { result } ) {
        var array = [];
        result.results.bindings.forEach(
            function( element ) {
                var object = {
                    gene_label: element.gene_label.value,
                    common_id: parseInt( element.common_id.value ),
                    scientific_name: element.scientific_name.value,
                    common_name: element.common_name.value,
                    time: parseFloat( element.time.value )
                };
                array.push( object );
            }
        );
        return array;
    }
})
```