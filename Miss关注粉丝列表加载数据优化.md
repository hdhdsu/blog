<!--
author: 冷火-王胜 
date: 2017-2-10 
title: Miss关注粉丝列表加载数据优化
tags: Android
category: Android
status: publish 
-->
场景：Miss项目中“我的”点击关注或者粉丝，会跳转到关注粉丝界面

优化之前：无论是点击关注跳转到关注粉丝界面还是点击粉丝跳转到关注粉丝界面，都会一次性加载两个fragment需要的数据，如下图：


![Paste_Image.png](http://upload-images.jianshu.io/upload_images/4669002-f0bbf2d3a0b4c359.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

优化之后：点击关注跳转只加载关注数据，点击粉丝跳转只加载粉丝数据

优化步骤：之所以在跳转之前就全部加载完成，主要是在前一个activity添加两个fragment的时候创建fragment的时候直接在onCreateView方法中加载了数据，代码如下     

    private void setupViewPager(ViewPager upViewPager) {
        mAdapterFollow = new AdapterFollow(getSupportFragmentManager());
        //关注fragment
        mFragmentFollow = new FragmentPersonFollow();
        Bundle bundleAttention = new Bundle();
        bundleAttention.putString("userId", mUserId);
        mFragmentFollow.setArguments(bundleAttention);

        //粉丝fragment
        mFragmentFans = new FragmentPersonFans();
        Bundle bundleFans = new Bundle();
        bundleFans.putString("userId", mUserId);
        mFragmentFans.setArguments(bundleFans);

        mAdapterFollow.addFragment(mFragmentFollow, "关注");
        mAdapterFollow.addFragment(mFragmentFans, "粉丝");
        mViewPager.setAdapter(mAdapterFollow);
        mViewPager.setCurrentItem(mNumber);
    }
fragment的onCreateView直接加载数据

       public View onCreateView(LayoutInflater inflater, ViewGroup container,
            Bundle savedInstanceState) {
        mLayout = inflater.inflate(R.layout.fragment_attention_layout, container, false);
        initView();
        initData(mUserId);
        return mLayout;
    }
解决方案：不在fragment初始化的时候加载数据，在viewpager切换的时候加载数据，而且要加个标志位是不是第一次加载，如果是第一次加载就加载，加载完之后标志位立刻变成false，第二次就不用重复加载了，代码如下： 

     mViewPager.setOnPageChangeListener(new ViewPager.OnPageChangeListener() {
            @Override
            public void onPageScrolled(int position, float positionOffset, int positionOffsetPixels) {

            }

            @Override
            public void onPageSelected(int position) {

                if (mIsLoad) {
                    if (position != mNumber) {
                        if (position == 0) {
                            mFragmentFollow.initData(mUserId);
                        } else if (position == 1) {
                            mFragmentFans.initData(mUserId);
                        }
                    }
                }
                mIsLoad = false;
            }

            @Override
            public void onPageScrollStateChanged(int state) {

            }
        });

但是第一次进来的时候没有走onPageSelected,所以第一次进来要加载一次数据   

         if (mNumber == 0) {
            mFragmentFollow.initData(mUserId);
            stopWaiting();
        }
        if (mNumber == 1) {
            mFragmentFans.initData(mUserId);
            stopWaiting();
        }