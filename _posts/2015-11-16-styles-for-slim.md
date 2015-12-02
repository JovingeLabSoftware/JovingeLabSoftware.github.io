---
layout: post
title: Code Snippet
date: 2015-12-02T00:00:00.000Z
published: true
---


(embarassingly) simply clustered code executio:

```R
cluster_go <- function() {
  cl <- makeCluster(8, type = "MPI")  
  clusterEvalQ(cl, source("geolincs.r"))
  gses <- get_ss()
  data <- clusterApplyLB(cl, gses, load_gse)
  stopCluster(cl)
  save(data, file="geo_lincs_metadata.tab")
}
```
