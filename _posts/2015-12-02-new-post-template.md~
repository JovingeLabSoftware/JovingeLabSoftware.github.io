---
layout: post
title:  "Code Snippet: Clustering"
date:   2015-12-02
---

Simple example of clustered analysis using snow & MPI.

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

