---
layout:     post
title:     分类滑动视图源码分析
subtitle:      
date:       2021-04-12
author:     Neil
header-img: img/post-bg-coffee.jpeg
catalog:      true
tags:
    - 视图
---


背景：
已有的JXCategoryView的功能比较复杂，实际使用的功能并没有怎么多，打算打造轻量级的标题与页面切换的视图补充到UI组件来支持UI组件


JXCategoryView的UML图

![8855546B-43D3-497C-85B5-D425CB613F6A.png](https://cdn.nlark.com/yuque/0/2020/png/368112/1599217574109-88664ac6-9d5e-4218-9c84-a66177400161.png#align=left&display=inline&height=656&margin=%5Bobject%20Object%5D&name=8855546B-43D3-497C-85B5-D425CB613F6A.png&originHeight=656&originWidth=682&size=76609&status=done&style=none&width=682)


JXCategoryView 功能比较多也比较全而实际我们只用到了titleImageView，结合现有的相关开源库和我们使用的场景，打算对JXCategoryView轻量化，同时支持新app的开发工作

对比一个开源库XLPageViewController发现还是JXCategoryView设计要好一点，因为它实现了titleView和ContainerView的分离，所以我打算结合XLPageViewController和JXCategoryView，在JXCategoryView的基础去掉多余功能简化JXCategoryView
#### 
#### 如何使用
```
/// 初始化
KSPageTitleImageView *mypageView = [[KSPageTitleImageView alloc] init]
mypageView.titles = self.titles;
mypageView.imageNames = imageNames;
mypageView.selectedImageNames = selectedImageNames;
mypageView.imageZoomEnabled = YES;
mypageView.imageZoomScale = 1.3;
mypageView.averageCellSpacingEnabled = NO;
/// 设置指示器
KSPageIndicatorLineView *lineView = [[KSPageIndicatorLineView alloc] init];
lineView.indicatorWidth = 20;
lineView.lineStyle = KSPageIndicatorLineStyle_Lengthen;
mypageView.indicators = @[lineView];
/// 刷新
[pageView reloadData];
```


#### 实现协议


KSPageView的代理
```
#pragma mark - KSPageViewDelegate

- (void)categoryView:(KSPageView *)categoryView didSelectedItemAtIndex:(NSInteger)index {
    NSLog(@"%@", NSStringFromSelector(_cmd));
}

- (void)categoryView:(KSPageView *)categoryView didScrollSelectedItemAtIndex:(NSInteger)index {
    NSLog(@"%@", NSStringFromSelector(_cmd));
}
```


KSPageListContainerView内容区代理
```
#pragma mark - KSPageListContainerViewDelegate

- (id<KSPageListContentViewDelegate>)listContainerView:(KSPageListContainerView *)listContainerView initListForIndex:(NSInteger)index {
    ListViewController *list = [[ListViewController alloc] init];
    return list;
}

- (NSInteger)numberOfListsInlistContainerView:(KSPageListContainerView *)listContainerView {
    return self.titles.count;
}
```


#### 扩展


Indicator扩展：可继承KSPageIndicatorComponentView视图，实现协议KSPageIndicatorProtocol协议
```
/// 刷新状态
- (void)ks_refreshState:(KSPageIndicatorParamsModel *)model{
}

/// 滑动中
- (void)ks_contentScrollViewDidScroll:(KSPageIndicatorParamsModel *)model{
}

///  选中了某一个cell
- (void)ks_selectedCell:(KSPageIndicatorParamsModel *)model{
}
```


KSPageView样式扩展：可继承KSPageTitleView，实现KSPageView中的UISubclassingHooks方法
```
/// 返回当前的类
- (Class)preferredCellClass {
}
/// 刷新数据
- (void)refreshDataSource {
}
/// 重置某个index下cell的状态
- (void)refreshCellModel:(KSPageViewCellModel *)cellModel index:(NSInteger)index {
    [super refreshCellModel:cellModel index:index];
}
/// 选中某个item时，刷新将要选中与取消选中的cellModel
- (void)refreshSelectedCellModel:(KSPageViewCellModel *)selectedCellModel unselectedCellModel:(KSPageViewCellModel *)unselectedCellModel {
}
/// 当contentScrollView滚动时候，处理跟随手势的过渡效果。 （KSPageIndicatorView 特有）
- (void)refreshLeftCellModel:(KSPageViewCellModel *)leftCellModel rightCellModel:(KSPageViewCellModel *)rightCellModel ratio:(CGFloat)ratio {
    [super refreshLeftCellModel:leftCellModel rightCellModel:rightCellModel ratio:ratio];
}
/// 计算index对应cell的宽度
- (CGFloat)preferredCellWidthAtIndex:(NSInteger)index {
}
```


#### 原理


KSPageView设计UML图
![34174712-E7C8-4174-B3B0-33319510C836.png](https://cdn.nlark.com/yuque/0/2020/png/368112/1599652194318-ca7d424c-edcf-46cd-92ef-7e876cf96619.png#align=left&display=inline&height=1154&margin=%5Bobject%20Object%5D&name=34174712-E7C8-4174-B3B0-33319510C836.png&originHeight=1154&originWidth=1696&size=341566&status=done&style=none&width=1696)


core部分
定义了基类部分KSPageView，定义子类继承其需要实现的方法UISubclassingHooks
KSPageView设置了属性listContainer使得分类视图和内容视图分离，通过依赖注入的方式传入
KSPageView定义了事件回调代理KSPageViewDelegate


listContainer
定义了KSPageListContainerViewDelegate协议，子类如果使用KSPageListContainerView需要实现这个协议

Indicator
KSPageIndicatorView继承KSPageView，实现KSPageView需要实现的方法
KSPageIndicatorView相当于Indicator的管理类，负责管理多个Indicator
定义一个KSPageIndicatorProtocol协议，所有Indicator样式都需要去实现KSPageIndicatorProtocol协议，其中包括刷新、滚动和选中三个方法（其中并没有表面遵循协议，使用时表示其尊许即可）


Title部分
KSPageTitleView继承KSPageIndicatorView，实现KSPageView和KSPageIndicatorView需要实现的方法


#### 总结
模块化
为了使得模块分离比如KSPageView和KSPageListContainerView，可以将功能尽心拆分，各自实现不同的协议，依赖注入即可


可扩展
定义出子类需要实现的协议或者方法，比如通过继承实现KSPageIndicatorProtocol和UISubclassingHooks来扩展其不同的样式内容（实现协议时候可不表明遵循协议）


分层
UI层和数据层的分离，比如Cell和CellModel的使用


当我遇到新的UI组件的时候可以使用上面的方案，首先进行功能拆分使得模块分离。其次定义什么样的协议可以支持扩展或者基类定义需要子类实现的方法一下，都是保证其有可扩展性。最后界面和模型的分离更有层次感。

作者
zhuhao@ksjgs.com
