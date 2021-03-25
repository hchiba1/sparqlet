# Evolutionary conservation patterns of human genes

See also [hop_time](hop_time) to get values of divergence time.

## Endpoint

https://orth.dbcls.jp/sparql-dev

## `hop`

```sparql
PREFIX orth: <http://purl.org/net/orth#>
PREFIX hop: <http://purl.org/net/orthordf/hOP/ontology#>
PREFIX dct: <http://purl.org/dc/terms/>

SELECT ?gene (REPLACE(GROUP_CONCAT(DISTINCT ?cat; separator=","), "[a-m]_", "") AS ?categories)
WHERE {
  {
    SELECT ?grp ?gene (max(?org_id) as ?max_id)
    WHERE {
      ?grp orth:hasHomologousMember ?gene ;
          orth:organism ?org .
      ?org dct:identifier ?org_id .
    }
  }
  BIND(IF(?max_id<=25, "a_M",
       IF(?max_id<=34, "b_V",
       IF(?max_id<=37, "c_LT",
       IF(?max_id<=41, "d_EH",
       IF(?max_id<=54, "e_A",
       IF(?max_id<=60, "f_N",
       IF(?max_id<=62, "g_C",
       IF(?max_id<=64, "h_SP",
       IF(?max_id<=67, "i_CF",
       IF(?max_id<=124, "j_F",
       IF(?max_id<=129, "k_AB",
       IF(?max_id<=147, "l_P",
          "m_O"))))))))))))
       AS ?cat)
}
```

## `return`

```javascript
({hop})=>{
  let r = [];
  
  hop.results.bindings.forEach(elem => {
    r.push({
      gene : elem.gene.value,
      categories: elem.categories.value});
  });
  
  return r;
};
```