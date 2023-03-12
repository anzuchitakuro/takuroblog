---
layout:     post
title:      "HorizontalPagerの中にLazyRowを置いたときにpagerのスクロールを無効にする方法"
subtitle:   ""
description: "HorizontalPagerの中にLazyRowを置いたときにpagerのスクロールを無効にする方法"
excerpt: "HorizontalPagerの中にLazyRowを置いたときにpagerのスクロールを無効にする方法"
date:       2023-03-11 09:32:00
author:     "takuro"
image: "/img/2018-04-11-service-mesh-vs-api-gateway/background.jpg"
tags:
    - Compose
    - Android
URL: "/2023/03/11/horizontal-pager/"
categories: [ Tech ]
---

# HorizontalPagerとLazyRowのスクロールが競合してしまう問題点
LazyRowの末端までスクロールした時に親のHorizontalPagerのスクロールに移ってしまいpagerが動いてしまう問題がありました。
要件的にpagerのスクロールを無効にしたいため解決策を調べました。

![](https://storage.googleapis.com/zenn-user-upload/3fc5dfd3f4fc-20220723.gif)

# 実装方法
依存関係のバージョンは以下です。
```
implementation 'com.google.accompanist:accompanist-pager:0.23.0'
```
Accompanistは、Jetpack Composeにはまだ実装されていない機能を補完したライブラリ群です。

```kotlin: PagerScreen.kt
HorizontalPager(
    state = pagerState,
    count = 3,
) {
    Column {
        Spacer(modifier = Modifier.height(300.dp))
        LazyRow {
            item {
                Spacer(modifier = Modifier.width(20.dp))
            }
            (1..10).forEach { index ->
                item {
                    Box(
                        modifier = Modifier
                            .width(160.dp)
                            .height(90.dp)
                            .background(Color.Gray),
                        contentAlignment = Alignment.Center
                    ) {
                        Text(text = index.toString())
                    }
                    Spacer(modifier = Modifier.width(20.dp))
                }
            }
        }
    }
}
```

# 原因
自動ネストスクロールによってスクロールアクションを開始する操作は、子から親に自動的に伝播されます。
つまり、子であるLazyRowのスクロールがそれ以上できなくなると、親であるHorizontalPagerのスクロールに移ってしまうことが原因でした。

参考：https://developer.android.com/jetpack/compose/gestures?hl=ja#auto-nested-scrolling

# 対応方法
LazyRowをBoxで囲いmodifierにスクロール可能にするscrollableと
特定方向のスワイプを無効にするdraggableを使うことによって解決できました。

![](https://storage.googleapis.com/zenn-user-upload/bfc3131d6c8b-20220723.gif)

```diff kotlin: PagerScreen.kt
HorizontalPager(
    state = pagerState,
    count = 3,
) {
    Column {
        Spacer(modifier = Modifier.height(300.dp))
+        Box(
+            modifier = Modifier
+                // LazyRowが末端までスクロール後、HorizontalPagerを動かないようにする
+                .scrollable(rememberScrollableState { it }, Orientation.Horizontal)
+                .draggable(
+                    interactionSource = remember { MutableInteractionSource() },
+                    state = remember { DraggableState {} },
+                    orientation = Orientation.Horizontal,
+                )
+        ) {
            LazyRow() {
                item {
                    Spacer(modifier = Modifier.width(20.dp))
                }
                (1..10).forEach { index ->
                    item {
                        Box(
                            modifier = Modifier
                                .width(160.dp)
                                .height(90.dp)
                                .background(Color.Gray),
                            contentAlignment = Alignment.Center
                        ) {
                            Text(text = index.toString())
                        }
                        Spacer(modifier = Modifier.width(20.dp))
                    }
                }
            }
+        }
    }
}

```
