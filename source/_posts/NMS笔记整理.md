---
title: NMS笔记整理
date: 2020-06-20 18:49:29
tags:
- Deep Learning Notes
categories:
- Deep Learning Notes
- Notes
mathjax: true
---

NMS, Soft NMS
<!--more-->

# NMS, Soft NMS
***

## NMS

NMS是用来过滤proposal的.对于检测算法来说,过滤重复的proposal,保留有用的proposal是非常重要的一步处理.
NMS的基本思路是:
- 将box按照score从大到小排序得到index
- 从index中选择分数最高的box放入result中
- 计算box与剩下所有box的iou
- 过滤掉所有iou大于阈值的box,保留剩下box的index
- 重复2直到剩下的index为空
```python
import numpy as np
def horizontal_nms(boxes, threshold):
    # check
    assert len(boxes.shape) == 2, 'box shape must have two dimention:' + str(len(boxes.shape))
    assert boxes.shape[1] == 5, 'the second dimention of boxes must be 5:' + str(boxes.shape[1])

    # no boxes
    if(len(boxes) == 0):
        return []
    
    # horizontal nms

    # all axis
    x1 = boxes[:, 0]
    y1 = boxes[:, 1]
    x2 = boxes[:, 2]
    y2 = boxes[:, 3]
    scores = boxes[:, 4]

    union = (x2 - x1 + 1) * (y2 - y1 + 1) # nonzero
    index = scores.argsort()[::-1]
    keep = []
    while index.size > 0:
        i = index[0]
        keep.append(i) # preserve first
        # calculate intersection
        xx1 = np.maximum(x1[i], x1[index[1:]])
        yy1 = np.maximum(y1[i], y1[index[1:]])
        xx2 = np.minimum(x2[i], x2[index[1:]])
        yy2 = np.minimum(y2[i], y2[index[1:]])

        inter = np.maximum(0.0, xx2 - xx1 + 1) * np.maximum(0.0, yy2 - yy1 + 1)
        iou = inter / (index[i] + union[index[1:]] - inter)
        inds = np.where(iou <= threshold)[0]
        index = index[inds + 1]

    return keep
```

## Soft NMS
在实际应用过程中,有些物体本身由于挨得很近,有部分发生重叠,所以标注的bounding box也会有部分重叠,直接去掉box的话会让这部分的召回率下降.因此采用打分机制,重叠区域大就在原有分数的基础上乘以一个权重.

![示例](./soft_nms.jpg)

算法流程图如下:

![Soft NMS 流程图](./soft_nms_pipeline.jpg)

如图所示,两者的区别就是一个直接把box去掉,另一个只是把box分数调低了.

Soft NMS算法的大致思路为：M为当前得分最高框，bi 为待处理框，bi 和M的IOU越大，bi 的得分si 就下降的越厉害.

对NMS来说:

$s_i = \left\{ \begin{aligned} s_i & ,  iou(M, b_i) < N_t \\ 0 & , iou(M,b_i) \geq N_t  \end{aligned} \right.$

对Soft NMS来说:

1. 线性加权:$s_i = \left\{ \begin{aligned} s_i &,& iou(M, b_i) < N_t \\ s_i(1 - iou(M, b_i)) &,& iou(M, b_i)\geq N_t\end{aligned} \right.$
2. 高斯加权:$s_i = s_ie^{-\frac{iou(M,b_i)^2}{\sigma}}, \forall b_i \notin D$

代码实现如下

```python
import numpy as np
def soft_nms(boxes, sigma=0.5, Nt=0.1, threshold=0.001, method=1):
    N = boxes.shape[0]
    pos = 0
    maxscore = 0
    maxpos = 0
	
    for i in range(N):
    	# [0, i)之间的box都是排好序的
        maxscore = boxes[i, 4]
        maxpos = i

        tx1 = boxes[i,0]
        ty1 = boxes[i,1]
        tx2 = boxes[i,2]
        ty2 = boxes[i,3]
        ts = boxes[i,4]

        pos = i + 1
    # 从[i, N]中找到分数最高的box
        while pos < N:
            if maxscore < boxes[pos, 4]:
                maxscore = boxes[pos, 4]
                maxpos = pos
            pos = pos + 1

    # 交换boxes[i]和找到的分数最大的box
        boxes[i,0] = boxes[maxpos,0]
        boxes[i,1] = boxes[maxpos,1]
        boxes[i,2] = boxes[maxpos,2]
        boxes[i,3] = boxes[maxpos,3]
        boxes[i,4] = boxes[maxpos,4]

        boxes[maxpos,0] = tx1
        boxes[maxpos,1] = ty1
        boxes[maxpos,2] = tx2
        boxes[maxpos,3] = ty2
        boxes[maxpos,4] = ts
	# 重新赋值为分数最大的box
        tx1 = boxes[i,0]
        ty1 = boxes[i,1]
        tx2 = boxes[i,2]
        ty2 = boxes[i,3]
        ts = boxes[i,4]
	# 现在boxes[i]就是分数最大的box
        pos = i + 1
    # NMS iterations, note that N changes if detection boxes fall below threshold
        while pos < N:
            x1 = boxes[pos, 0]
            y1 = boxes[pos, 1]
            x2 = boxes[pos, 2]
            y2 = boxes[pos, 3]
            s = boxes[pos, 4]

            area = (x2 - x1 + 1) * (y2 - y1 + 1)
            iw = (min(tx2, x2) - max(tx1, x1) + 1)
            if iw > 0:
                ih = (min(ty2, y2) - max(ty1, y1) + 1)
                if ih > 0:
                	# area是i后面某个box的面积, ua是两个box面积的并集,iw*ih是交集
                    ua = float((tx2 - tx1 + 1) * (ty2 - ty1 + 1) + area - iw * ih)
                    ov = iw * ih / ua #iou between max box and detection box
					
                    if method == 1: # linear
                        if ov > Nt:
                            weight = 1 - ov
                        else:
                            weight = 1
                    elif method == 2: # gaussian
                        weight = np.exp(-(ov * ov)/sigma)
                    else: # original NMS
                        if ov > Nt:
                            weight = 0
                        else:
                            weight = 1

                    boxes[pos, 4] = weight*boxes[pos, 4]
                    print(boxes[:, 4])

            # 如果box分数小于阈值的话就移到最后,直接丢弃
            # update N
                    if boxes[pos, 4] < threshold:
                        boxes[pos,0] = boxes[N-1, 0]
                        boxes[pos,1] = boxes[N-1, 1]
                        boxes[pos,2] = boxes[N-1, 2]
                        boxes[pos,3] = boxes[N-1, 3]
                        boxes[pos,4] = boxes[N-1, 4]
                        N = N - 1
                        pos = pos - 1

            pos = pos + 1
    keep = [i for i in range(N)]
    return keep
```