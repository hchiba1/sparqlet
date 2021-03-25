# Get genes that co-express with gene (NCBI Gene ID)

## Parameters
* `arg1`
  * default: 10001

## Endpoint
http://coxpresdb.jp/sparql

## `result`

```sparql
PREFIX geneid: <http://identifiers.org/ncbigene/>
PREFIX coxpresdb: http://coxpresdb.jp/rdf/def/0.1/

SELECT ?gene ?rank ?cor
WHERE {
    <http://coxpresdb.jp/rdf/db/Hsa.v12>
          coxpresdb:gene_pair ?pair .
    ?pair coxpresdb:gene_id_1 <http://bio2rdf.org/geneid:{{arg1}}> ;
          coxpresdb:gene_id_2 ?gene ;
          coxpresdb:mutual_rank ?rank ;
          coxpresdb:pcc ?cor .
}
ORDER BY ?rank


```