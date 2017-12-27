## MVP中activity,presenter，adapter之间的各种回调
#### activity（fragment）
activity是用来展示界面的，它可以继承一个baseactivity，用来做每个activity都有的一些操作（事实上一个app都会有一个baseactivity），定义一个接口，这个接口是有些activity共有的一些行为（不是所有的activity都有的一些行为），然后activity去实现这个接口

    public interface IUIHomeFollow extends IUIPtrStatus{

    /**
     * 更新数据列表
     * @param dataSet 数据
     * @description: Created by Boqin on 2016/12/5 9:14
     */
    void onUpdateItem(List<ContentItemBean> dataSet);

    /**
     * 添加数据
     * @param dataSet
     * @description: Created by Boqin on 2016/12/5 9:20
     */
    void onAddItem(List<ContentItemBean> dataSet);

    /**
     * 加载更多完成
     * @description: Created by Boqin on 2016/12/5 9:21
     */
    void onLoadMoreComplete();


    /**
     * 加载完成
     * @description: Created by Boqin on 2016/12/5 9:14
     */
    void onResetLoadMore();
    }
这里定义了一个接口，继承这个接口的activity有四个功能，那就是更新数据列表（主要用于下拉刷新），添加数据（上拉加载），加载更多完成（没有更多了），加载完成（这一页已经加载完成）

fragment也可以继承一个basefragment（抽象类），代表有两个属性，可见和不可见（可以实现fragment的懒加载）

    public abstract class FragmentPagerBase extends FragmentBase{

    /**
     * 当ViewPager可见时
     * @description: Created by Boqin on 2016/11/10 14:38
     */
    public abstract void onVisibleInViewPager();

    /**
     * 当ViewPager隐藏时
     * @description: Created by Boqin on 2016/11/10 14:40
     */
    public abstract void onInvisibleInViewPager();
    }
实现懒加载的原理（在fragment可见的时候就去加载数据，在fragment不可见的时候不去加载数据）

#### presenter
mvp中的activity只是用来展示数据的，加载数据是在presenter中实现的，不过要在activity中初始化presenter

    mPresenterFollow = new PresenterHomeFollow(this);
然后在presenter中就拥有了一个IUIHomeFollow对象

       public PresenterHomeFollow(IUIHomeFollow uiFollow) {
        mUIFollow = uiFollow;
        mNextPosition = "0";
        mIsLoading = false;
        mIsShowLoadNoConnectedToast = true;
        }
可以再定义一个接口那就是IPresenterHome，它只有两个行为，那就是加载数据和添加数据，对应的是下拉刷新和上拉加载

    public interface IPresenterHome extends IPresenterBase{
    /**
     * 执行刷新
     * @description: Created by Boqin on 2016/11/26 15:37
     */
    void performRefresh();
    /**
     * 加载更多
     * @description: Created by Boqin on 2016/12/4 18:39
     */
    void performLoadMore();
    }
这时候presenter就可以实现这个接口，在这两个方法中加载数据

    @Override
    public void performRefresh() {
        //网络请求
        mIsShowLoadNoConnectedToast = true;
        MissProtocolHall.getInstance().getFollowList(mRefreshCallback, null);
    }

    @Override
    public void performLoadMore() {
        //有起始页数据
        if ("0".equals(mNextPosition)) {
            if (mUIFollow == null) {
                return;
            }
            mUIFollow.onLoadMoreComplete();
        } else if (!mIsLoading) {
            if (!NetworkUtil.isNetworkConnected() && mIsShowLoadNoConnectedToast) {
                mUIFollow.showToast("无网络连接");
                mIsShowLoadNoConnectedToast = false;
                return;
            }
            mIsLoading = true;
            MissProtocolHall.getInstance().getFollowList(mLoadMoreCallback, mNextPosition);
        }
    }
然后在callback中将数据回调给activity

     mUIFollow.onUpdateItem(contentItemListBean.getItemList());
     mUIFollow.onRefreshComplete();
然后在activity的onUpdateItem方法中就有了数据（activity实现了IUIHomeFollew），接下来就是adapter去渲染数据的事了

    @Override
    public void onUpdateItem(List<ContentItemBean> dataSet) {
        if (mAdapter != null) {
            mAdapter.updateDataset(dataSet);
            if (dataSet.size() != 0) {
                mPtrFrameLayout.onNormalContent();
            } else {
                //跳转到空页
                mPtrFrameLayout.onEmpty();
            }
            mRecyclerView.notifyDataSetChanged();
        }

    }

#### adapter
adapter主要干的就是一件事，那就是数据和UI的绑定
如果你用的是recyclerview就是adapter和viewholder配合，这个可以再看其他资料，比如说recyclerview怎么实现复杂布局
如果涉及到组件之间的通信，就可以用eventbus或者rxjava