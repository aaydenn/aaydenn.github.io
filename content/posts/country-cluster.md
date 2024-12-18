+++
title = 'Can we cluster countries based on trend lines through time?'
tags = ['r', 'python', 'worlddata', 'distance']
draft = true
date = '2024-11-21T18:31:11+01:00'
+++

```{r}
library(ggplot2)

df <- data.frame(id = rep(c("X","Y","Z","P","Q"), each = 4),
                 value = c(1, 2, 3, 5, 3, 6, 9, 14, 2, 4, 8, 13, 15, 12, 9, 5, 4, 5, 6, 4),
                 step = rep(c(1,2,3,4), times = 5))

(mocktrend <- ggplot(df, aes(step, value, group = id)) +
    geom_line(aes(color = id)) +
    geom_point(aes(shape = id, color = id)) +
    theme_bw())
# X <- c(1, 2, 3, 5)
# Y <- c(3, 6, 9, 14)
# Z <- c(2, 4, 8, 13)
# P <- c(15, 12, 9, 5)
# Q <- c(4, 5, 6, 4)
```

```{r}
A <- letters[X]
B <- letters[Y]
C <- letters[Z]
D <- letters[P]
E <- letters[Q]
```

```{r}
df1 <- split(df$value, df$id) |> as.data.frame()
```

```{r}
jaccard <- function(x, y, weighted = TRUE) {
  if (weighted) {
    return(1 - (sum(min(x, y)) / sum(max(x, y))))
  } else {
    return(1 - (length(intersect(x, y)) / length(union(x, y))))
  }
}

cosine <- function(x, y) {
  return(1 - (sum(x * y) / (sum(x ^ 2) ^ (1 / 2) * sum(y ^ 2) ^ (1 / 2))))
}

tanimoto <- function(x, y, weighted = TRUE) {
  if (weighted) {
    return(1 - (length(intersect(x, y)) / length(x) + length(y) - length(intersect(x, y))))
  }
  return(1 - (sum(x * y) / (sum(sum(x ^ 2), sum(y ^ 2)) - sum(x * y))))
}

minkowski <- function(x, y, p) {
  return(sum(abs(x - y) ^ p) ^ (1 / p))
}

canberra <- function(x, y) {
  return(sum(abs(x - y) / (abs(x) + abs(y))))
}
```

```{r}
distance <- function(df) {
  M <- matrix(0, ncol(df), ncol(df))
  
  for (i in 1:ncol(df)) {
    for (j in 1:ncol(df)) {
      M[i, j] <- tanimoto(df[, i], df[, j])
    }
  }
  colnames(M) <- rownames(M) <- colnames(df)
  return(M)
}

distance(df1)
```

todo
- find metrics for countries (worlddata?)
- compare speed with basic chatgpt answer
- cluster using different k-mers (how to find optimal?)
- also different clustering algo.
- based on clustering together make a graph (frequency as degree)

```{python}
import timeit
import random

def cosine(x, y):
    dot_product = sum(a * b for a, b in zip(x, y))
    magnitude_x = sum(a ** 2 for a in x) ** 0.5
    magnitude_y = sum(b ** 2 for b in y) ** 0.5
    return dot_product / (magnitude_x * magnitude_y)
  
x = [random.random() for _ in range(10**6)]
y = [random.random() for _ in range(10**6)]

execution_time = timeit.timeit('cosine(x, y)', globals=globals(), number=100)

print(f"Average execution time: {execution_time / 100:.6f} seconds")
```

```{python}
import numpy as np
import timeit

def cosine_numpy(x, y):
    x = np.array(x)
    y = np.array(y)
    return np.dot(x, y) / (np.linalg.norm(x) * np.linalg.norm(y))

# Test vectors
x = np.random.rand(10**6)
y = np.random.rand(10**6)

# Benchmark
execution_time = timeit.timeit('cosine_numpy(x, y)', globals=globals(), number=100)
print(f"Average execution time: {execution_time / 100:.6f} seconds")

```

```{r}
library(microbenchmark)

optimized_cosine <- function(x, y) {
  dot_product <- sum(x * y)
  magnitude_x <- sqrt(sum(x ^ 2))
  magnitude_y <- sqrt(sum(y ^ 2))
  return(dot_product / (magnitude_x * magnitude_y))
}
oneliner_cosine <- function(x, y) {
  return(sum(x * y) / (sum(x ^ 2)^(1/2) * sum(y ^ 2)^(1/2)))
}

microbenchmark(
  original = cosine(x, y),
  optimized = optimized_cosine(x, y),
  oneliner = oneliner_cosine(x, y),
  times = 100
)
```

