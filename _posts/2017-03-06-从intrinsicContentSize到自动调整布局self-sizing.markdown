---
tags: IOS,swift
grammar_cjkRuby: true
grammar_abbr: true
grammar_table: true
grammar_defList: true
grammar_emoji: true
grammar_footnote: true
grammar_ins: true
grammar_mark: true
grammar_sub: true
grammar_sup: true
grammar_checkbox: true
grammar_mathjax: true
grammar_flow: true
grammar_sequence: true
grammar_plot: true
grammar_code: true
grammar_highlight: true
grammar_html: true
grammar_linkify: true
grammar_typographer: true
grammar_video: true
grammar_audio: true
grammar_attachment: true
grammar_mermaid: true
grammar_classy: true
grammar_cjkEmphasis: true
grammar_cjkRuby: true
grammar_center: true
grammar_align: true
grammar_tableExtra: true
layout: post
title:  "从intrinsicContentSize到自动调整布局self-sizing"
date:   2017-03-06 12:05:48 +0800
categories: ios
author: taosiyu
---

# 1. intrinsic Content Size
	
	什么是intrinsicContentSize？这个属性是在哪里的？
	刚开始看到这个肯定会有很多的疑问。
	本人接触到这个也是因为功能的需要。（一个自动适应cell内容的collectionview）。
	在刚编程的时候，第一想到的还是手动适应，计算contentSize然后设置frame适应内容，但是这样给人的感觉还是有点low啊.
	果断换其他方法。直到发现了intrinsicContentSize这个便捷的属性，简直不要太兴奋。
	那到底什么是intrinsicContentSize呢？
	Intrinsic Contenet Size – Intrinsic Content Size：固有的大小。
	在AutoLayout中，它作为UIView的属性（不是语法上的属性），意思就是说我知道自己的大小，如果你没有为我指定大小，我就按照这个大小来。
	比如：大家都知道在使用AutoLayout的时候，UILabel是不用指定尺寸大小的，只需指定位置即可，就是因为，只要确定了文字内容，字体等信息，它自己就能计算出大小来。
	同样的UILabel，UIImageView，UIButton等这些组件及某些包含它们的系统组件都有 Intrinsic Content Size 属性，也就说他们都有自己计算size的能力。
	同样的会发现UICollectionView也有这个属性。
	
# 利用intrinsicContentSize自动计算size

>那如何利用intrinsicContentSize呢。
>既然intrinsicContentSize能够自己计算，当然我们也可以手动指定他的大小，这样会有
>人说。那不还是自己计算吗？
>其实不是，当我们改变UICollectionView中cell的约束和内容时，UICollectionView是会自动计算size适应的，这就是contentSize，每当改变contentSize也会随着改变，计算UICollectionView都帮我们算好了，我们就可以利用contentSize来设置内容的size.
>使用在UICollectionView中的 -(CGSize)intrinsicContentSize: 方法。 
并且在需要改变这个值的时候调用：invalidateIntrinsicContentSize 方法，通知系统这个值改变了，如下
```
//重写该方法返回一个size
override func intrinsicContentSize() -> CGSize {
        //此处你可以返回一个自己设定的size值，当然也可以返回contentSize
        return self.contentSize
    }
```
>但是有人比较懒不想主动调用方法，也有更好的自动调用的方法，就是利用layoutSubviews
```
//自定义的UICollectionView
class DynamicCollectionView: UICollectionView {
    override func layoutSubviews() {
        super.layoutSubviews()
        //此处加判断如果相等就不更新size
        if !CGSizeEqualToSize(self.bounds.size, intrinsicContentSize()) {
            invalidateIntrinsicContentSize()
        }
    }
    override func intrinsicContentSize() -> CGSize {
        //这里返回自定义的UICollectionView自己的属性contentSize
        return self.contentSize
    }
}
```
>这样的话每当UICollectionView内部的内容发生改变，他都会自己适应内容的大小达到自适应的目的.当然前提是不能添加高度的约束，添加了高度约束intrinsicContentSize会失效啊，当然你也可以自定义UILabel等，只要有intrinsicContentSize属性的都可以设置自动适应的能力。

## 注意：
>还有当我们使用自己自定义的UICollectionView的时候，每当刷新数据调用reloadData()方法的时候：
>务必调用 collectionViewLayout.invalidateLayout()方法，不然可能会发生下面的错误
```
UICollectionView received layout attributes for a cell with an index path that does not exist
```
当然，由于还没有特别深入的了解写的不是很详细，其实还有很多有趣的属性，比如：
invalidateItemsAtIndexPaths: 
invalidateSupplementaryElementsOfKind:atIndexPaths: 
invalidateDecorationElementsOfKind:atIndexPaths:
的：
invalidatedItemIndexPaths 
invalidatedSupplementaryIndexPaths 
invalidatedDecorationIndexPaths 
等等有待研究。