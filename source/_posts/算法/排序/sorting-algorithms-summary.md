---
title: 排序算法总结
date: 2020-06-10 13:30:17
tags: 
categories: 排序
thumbnail: https://i.loli.net/2020/03/30/EA54xPoiXY7wuJm.png
---

## 排序算法总结

<!--more-->

![常用排序算法](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/image-20210701004637185.png)





|              |           时间复杂度            | 空间复杂度  |   是否为稳定排序   |   是否为原地排序   |
| :----------: | :-----------------------------: | :---------: | :----------------: | :----------------: |
| 直接插入排序 |            $O(n^2)$             |   $O(1)$    | :heavy_check_mark: | :heavy_check_mark: |
| 折半插入排序 | $ O(n^2)$ (只优化了比较的过程)  |   $O(1)$    | :heavy_check_mark: | :heavy_check_mark: |
|   希尔排序   |    和增量有关，最坏$O(n^2)$     |   $O(1)$    |        :x:         | :heavy_check_mark: |
|   冒泡排序   |            $O(n^2)$             |   $O(1)$    | :heavy_check_mark: | :heavy_check_mark: |
| **快速排序** |          $O(nlog_2n)$           | $O(log_2n)$ |        :x:         | :heavy_check_mark: |
| 简单选择排序 |            $O(n^2)$             |   $O(1)$    |        :x:         | :heavy_check_mark: |
|  **堆排序**  |          $O(nlog_2n)$           |   $O(1)$    |        :x:         | :heavy_check_mark: |
| **归并排序** |          $O(nlog_2n)$           |   $O(n)$    | :heavy_check_mark: |        :x:         |
| ==基数排序== |         $O(dn) d是位数$         |   $O(d)$    | :heavy_check_mark: |        :x:         |
|  ==桶排序==  | $O(n+k)$ k为桶个数，n为元素数量 |  $O(n+k)$   | :heavy_check_mark: |        :x:         |



## References

* 严版数据结构
* 数据结构与算法之美