---
title: React Fiber
tags:
  - fiber
categories:
  - react
date: 2018-12-24 21:47:57
---

React Fiber 是对 React 核心算法的重新实现，目标是提高其对动画，布局和手势等领域的适用性。它的最重要的特性是 incremental rendering（增量渲染）：它能够将渲染 work 拆分成多块并将这些任务块分散到多个帧执行。
