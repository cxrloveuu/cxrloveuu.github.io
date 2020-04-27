---
layout: post
title: 小程序购物车的实现
categories: [miniprogram, cart, js]
description: 简易购物车逻辑的实现
keywords: 小程序, 购物车
---

## 前言

最近老板心(nao)血(dai)来(chou)潮(feng),在我们的社交小程序上加了个商城,顺带让我实现了基本没人点的购物车功能。
对,功能可以不用,但你,不能不写。
作为一个有脾气的前端,我立即投入了工作,于是有了这篇博客。
如有哪里出现菜鸡操作,望大佬们不吝赐教......

## 需求

作为一个伟(cai)大(ji)的 web 前端,需求先给他分析一波:

    - 单选,全选作为购物车的基本功能

    1. 价格随选中状态变化
    2. 全选状态除了点击之外,随单选状态变化(看了有几个大佬的blog,好像有的忘了这一点)
    3. 单选状态随全选变化(废话)
    4. 选中删除

## WXML(HTML)

由于这个项目重点不在于页面,所以只是放了项目中的页面,所以可能有些地方看起来有点怪

```java
    <view class="title">
    <view class="left">
        <view class="num">共{{ list.length }}件商品</view>
        <view class="address text-cut" wx:if="{{ phone }}" bind:tap="chooseAddress"
        >收货地址：{{ country }}{{ detail }}</view
        >
        <view class="noaddress" wx:else bind:tap="chooseAddress">添加收货地址</view>
    </view>
    // 呼出删除
    <view class="right" bind:tap="operation">{{operation?'完成':'管理'}}</view>
    </view>
    <view  class="section">
    // 卡片
    <view class="list" wx:for="{{list}}" wx:key="idx">
    <view class="top">
        <view class="leftBox">
        <image src="/mall/1images/{{item.selected?'choose_img_on':'choose_type'}}@2x.png" bind:tap="select" data-index="{{index}}"/>
        </view>
        <view class="product" bind:tap="toDetail" data-commodityid="{{item.commodity.id}}">
        <image src="{{item.commodity.photos[0].image.url}}" />
        <view class="content">
            <view class="name text-cut">{{item.commodity.title}}</view>
            <view class="type">{{item.commodity_option?item.commodity_option:''}}</view>
        </view>
        </view>
    </view>
    <view class="bottoms">
        <view class="rightBox">
        <view class="priceBox">
            <view class="price">{{item.userPrice>0?'¥':''}}{{filter.getPrice(item.userPrice)}}</view>
            <view class="oldPrice" wx:if="{{user.vip.expire}}">市场价：¥{{filter.getPrices(item.commodity.market_price)}}</view>
        </view>
        <view class="count">
            <image src="/mall/1images/reduce@2x.png" bind:tap="reduce" data-index="{{index}}"/>
            <text >{{item.quantity}}</text>
            <image src="/mall/1images/add@2x.png" bind:tap="add" data-index="{{index}}"/>
        </view>
        </view>
    </view>
    </view>
    <view wx:if="{{loaded&& list.length==0}}" class="nolist">
    购物车空空如也~
    </view>
    </view>
    <!-- 页脚 -->
    <view class="bottom">
    <view class="all" bind:tap="selectAll"><image src="/mall/1images/{{selectedAll?'choose_img_on':'choose_type'}}@2x.png"/><text >全选</text></view>
    <view class="besure" wx:if="{{!operation}}">
        <view class="prices">
        <view class="prices-item">合计：¥{{filter.getPrices(price)}}</view>
        <view class="price-post" wx:if="{{freight>0}}">不含运费</view>
        </view>
        <view class="besure-button" bind:tap="openConfirm">结算<text wx:if="{{chooseList.length>0}}">({{chooseList.length}})</text></view>
    </view>
    <view class="delete" wx:else bind:tap="delete">删除</view>
    </view>
    <wxs src="../../wxs/filter.wxs" module="filter"></wxs>
```

其中,wxs(WeiXin Script)是小程序的一套脚本语言,我这里用作处理计价,类似于 VUE 的计算属性

## ts(用了 typescript,不过最终会转换成 JS 执行)

由于项目比较赶,所以 TS 很多断言和数据类型都没用,大家当 JS 看就好了,真实业务比较复杂,所以只贴了部分逻辑代码

获取数据及初始化就不交代了,我这里是判断加载后列表长度,用来判断是否为空

```java
    list.map((i) => {
      i.selected = false
    })
    //这个地方的list是引用数据类型,所以不需要list=list.map()
```

然后就是数据的增加和修改了

```java
    async reduce(e: event.Custom) {
        const {
        currentTarget: {
            dataset: { index },
        },
        } = e
        const {
        data: { list },
        } = this
        if (list[index].quantity < 2) {
        //我们逻辑是至少保留一件,给删除按钮留了面子
        list[index].quantity = 1
        return wx.showToast({
            title: '至少选择一件商品',
            icon: 'none',
        })
        } else {
        list[index].quantity -= 1
        }
        /** 这里写发送请求,最好加上一个防抖函数 */

        this.setData({
            list,
        })
        /** 如果选中,触发计价函数 */
        list[index].selected && this.getPrice()
    },
  add(e: event.Custom) {
    const {
      currentTarget: {
        dataset: { index },
      },
    } = e
    const {
      data: { list },
    } = this

    list[index].quantity += 1
    /** 这里写发送请求,最好加上一个防抖函数 */

    /** 触发计价函数 */
    list[index].selected && this.getPrice()
    this.setData({
      list,
    })
  },
```
