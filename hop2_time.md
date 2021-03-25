# Divergence time of human genes

See also [hop_categories](hop_categories) to get evolutionary patterns.

## Endpoint

https://orth.dbcls.jp/sparql-dev

## `hop`

```sparql
PREFIX orth: <http://purl.org/net/orth#>
PREFIX hop: <http://purl.org/net/orthordf/hOP/ontology#>
PREFIX dct: <http://purl.org/dc/terms/>

SELECT ?gene (GROUP_CONCAT(DISTINCT ?max_time; separator=",") AS ?branch_times)
WHERE {
  {
    SELECT ?grp ?gene (max(?time) AS ?max_time)
    WHERE {
      ?grp orth:inDataset <http://purl.org/net/orthordf/hOP/> ;
          orth:hasHomologousMember ?gene ;
          orth:organism ?org .
      ?org hop:branchTimeMya ?time .
    }
  }
}
```

## `return`

```javascript
({hop})=>{
  let r = [];
  
  hop.results.bindings.forEach(elem => {
    r.push({
      gene: elem.gene.value,
      branch_times: elem.branch_times.value});
  });
  
  return r;
};
```