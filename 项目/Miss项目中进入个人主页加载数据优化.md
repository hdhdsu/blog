<!--
author: 冷火-王胜 
date: 2017-2-10 
title: Miss项目中进入个人主页加载数据优化
tags: Android
category: Android Miss
status: publish 
-->
场景：点击用户头像进入个人主页
个人主页和粉丝关注列表类似，但是和关注粉丝列表不同的是，它里面的fragment是根据服务器传过来的type来确定数目和类型。
优化步骤和关注粉丝列表差不多，都是在fragment初始化的时候不加载数据，在viewpager切换的时候加载fragment的数据（fragment会暴露出一个initData（）方法加载数据），不同的是此时的标志位不是boolean类型，而是一个boolean数组类型，因为这个时候fragment不止两个，要保证每个fragment第一次加载完成之后就不再重复加载，代码如下

        mViewPager.setOnPageChangeListener(new ViewPager.OnPageChangeListener() {
              @Override
               public void onPageScrolled(int position, float positionOffset, int positionOffsetPixels) {
            }

            @Override
            public void onPageSelected(int position) {
                Log.d("xuanze", position + "");
                mCurrentPosition = position;

                switch (mType.get(position)) {
                    case "关于我":
                        if(isLoad[0]){
                            mFragmentAboutMe.initData();
                            isLoad[0]=false;
                        }
                        break;
                    case "动态":
                        if(isLoad[1]){
                            mFragmentPersonMblog.initData();
                            isLoad[1]=false;
                        }
                        break;
                    case "作品":
                        if(isLoad[2]){
                            mFragmentPersonWorks.initData();
                            isLoad[2]=false;
                        }
                        break;
                    case "推文":
                        if(isLoad[3]){
                            mFragmentPersonBlog.initData();
                            isLoad[3]=false;
                        }
                        break;
                    case "教学":
                        if(isLoad[4]){
                            mFragmentPersonCourse.initData();
                            isLoad[4]=false;
                        }
                        break;
                    case "时尚":
                        if(isLoad[5]){
                            mFragmentPersonFashion.initData();
                            isLoad[5]=false;
                        }
                        break;
                    default:
                        break;
                }
            }

            @Override
            public void onPageScrollStateChanged(int state) {

            }
        });