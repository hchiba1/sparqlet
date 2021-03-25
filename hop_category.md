# get phylogenetic profile category from hOP

## Parameters

* `gene`
  * default: 100288485


## Endpoint

https://orth.dbcls.jp/sparql

## `hop`

```sparql
PREFIX hop: <http://purl.org/net/orthordf/hOP/ontology#>
PREFIX orth: <http://purl.jp/bio/11/orth#>
PREFIX ncbigene: <http://identifiers.org/ncbigene/>

SELECT ?grp ?max ?category
WHERE {
  {
    SELECT ?grp (max(?time) as ?max)
    WHERE {
      ?grp orth:member ncbigene:{{gene}} .
      ?grp orth:organism ?org .
      ?org hop:branchTimeMya ?time .
    }
  }
  BIND(IF(?max<180, "M",
       IF(?max<430, "V",
       IF(?max<735, "LT",
       IF(?max<750, "EH",
       IF(?max<850, "A",
       IF(?max<940, "C",
       IF(?max<1000, "CF",
       IF(?max<1400, "F",
       IF(?max<1520, "P",
       IF(?max<1660, "AB",
          "O"))))))))))
       AS ?category)
}
```

## `return`

```javascript
({hop})=>{
  let r = [];
  
  hop.results.bindings.forEach(elem => {
    r.push({grp: elem.grp.value, 
            max: elem.max.value,
           category: elem.category.value});
  });
  
  return r;
};
```