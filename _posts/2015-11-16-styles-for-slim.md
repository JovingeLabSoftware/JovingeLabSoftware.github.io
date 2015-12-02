---
layout: post
title: Code Snippet
date: {}
published: true
---



(embarassingly) simply clustered code execution:

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
