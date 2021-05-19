---
title: 排序算法总结
date: 2020-06-10 13:30:17
tags: 
categories: 排序
thumbnail: https://i.loli.net/2020/03/30/EA54xPoiXY7wuJm.png
---

## 排序算法总结

<!--more-->

![排序算法细分](https://i.loli.net/2020/06/10/eXjLwimpAI3SBbT.png)

## 衡量(内部排序)



|              |   时间复杂度    |   是否为稳定排序   |   是否为原地排序   |
| :----------: | :-------------: | :----------------: | :----------------: |
| 直接插入排序 |    $O(n^2)$     | :heavy_check_mark: | :heavy_check_mark: |
|   希尔排序   |   $O(nlogn)$    |        :x:         | :heavy_check_mark: |
|   冒泡排序   |    $O(n^2)$     | :heavy_check_mark: | :heavy_check_mark: |
|   快速排序   |   $O(nlogn)$    |        :x:         | :heavy_check_mark: |
| 简单选择排序 |    $O(n^2)$     |        :x:         | :heavy_check_mark: |
|    堆排序    |   $O(nlogn)$    |        :x:         | :heavy_check_mark: |
|   归并排序   |   $O(nlogn)$    | :heavy_check_mark: |        :x:         |
|   基数排序   | $O(dn) d是维度$ | :heavy_check_mark: |        :x:         |

## References

* 严版数据结构
* 数据结构与算法之美