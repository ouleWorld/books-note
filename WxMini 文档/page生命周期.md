[toc]

页面的生命周期

# 1. 页面的生命周期
1. onLoad: 页面加载时触发, 一个页面只会调用一次
2. onShow: 页面显示/切入前台时触发
3. onReady: 页面初次渲染完成时触发。一个页面只会调用一次，代表页面已经准备妥当，可以和视图层进行交互
4. onHide: 页面隐藏/切入后台时触发(tabBar 切页面也会触发)
5. onUnload: 页面卸载时触发

# 2. 页面事件处理函数
1. onPullDownRefresh: 监听用户下拉刷新事件
2. onReachBottom: 监听用户上拉触底事件
3. onShareAppMessage: 监听用户滑动页面事件
4. onPageScroll: 监听用户点击页面内转发按钮
5. onResize: 小程序屏幕旋转时触发
6. onTabItemTap: 点击 tab 时触发