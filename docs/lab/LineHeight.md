---
author: dz
title: 行高实验室
sidebarDepth: 0
---

# 行高实验室

实验一个工具：百度云风格的字体行高计算器

原理假设：文字字号越小则行高和文字的比例越小，这样更符合用户体验

为什么： 因为当文字比较小的时候，就算使用很高的行高倍数，实际的间距计算值也会比较小，比如 12px 的文字，二倍也不过 24px，间距 12px，如果文字过大比如  30px，只有 1.5 倍的情况，行高也有 45px，间距也有 15px

工具原理，：输入一个最小字号和对应行高，一个最大字号和行高，则自动计算中间的插值。

<line-height :ask="[12,14,18,20,24,30,42]"/>