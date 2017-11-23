<!--
author: 冷火-王胜 
date: 2017-2-8 
title: Android中的事件分发和处理
tags: Android view
category: Android
status: publish 
-->
###我们在开发过程中都会跟各种各样的view打交道，例如经常会用到onClick,onTouch,onTouchEvent,但是对于这些view事件都是一知半解，本文主要是通过研究view的事件分发和处理让我们更好的了解view
####首先我们要先了解android中事件是什么
	MotionEvent表示用户的触摸事件，用户的一次点击、触摸或者滑动都会产生一系列的MotionEvent：
	MotionEvent.ACTION_DOWN 表示用户的手指刚接触到屏幕
	MotionEvent.ACTION_MOVE 表示用户的手指正在移动
	MotionEvent.ACTION_UP 表示用户的手指从屏幕上抬起 

	所以一次用户触摸屏幕可能会产生这些事件:  
	点击屏幕然后松开，Down->Up  
	点击屏幕，然后滑动一段距离，松开屏幕 ，Down->Move->…->Move->Up

####我们先了解view的事件分发，然后再去了解ViewGroup的事件分发
首先你需要知道一点，那就是只要你触摸到了任何一个控件，就一定会调用该控件的dispatchTouchEvent方法。这个方法实现了当我们点击最顶层的ViewGroup的时候事件向下一层一层的传递给子view，我们来看一下View中dispatchTouchEvent的源码   
 
	public boolean dispatchTouchEvent(MotionEvent event) {  
    if (mOnTouchListener != null && (mViewFlags & ENABLED_MASK) == ENABLED &&  
            mOnTouchListener.onTouch(this, event)) {  
        return true;  
    }  
    return onTouchEvent(event);  
	}  
这个方法里面首先是进行了一个判断，如果mOnTouchListener != null， (mViewFlags & ENABLED_MASK) == ENABLED，mOnTouchListener.onTouch(this, event)，这三个条件都为真才返回true，否则的话就去执行onTouchEvent(event)方法并返回。 
 
其中，第一个mOnTouchListener正是在setOnTouchListener方法里赋值的，也就是说只要我们给控件注册了touch事件，mOnTouchListener就一定被赋值了。 
 
第二个条件(mViewFlags & ENABLED_MASK) == ENABLED是判断当前点击的控件是否是enable的，控件是enable的为true，否则为false。 
 
第三个条件是mOnTouchListener.onTouch(this, event)，其实也就是去回调控件注册touch事件时的onTouch方法。也就是说如果我们在onTouch方法里返回true，控件是可点击的，那么整个方法就返回true，否则就会再次执行onTouchEvent(event)方法。 
 
我们再来看看onTouchEvent的源码 

		public boolean onTouchEvent(MotionEvent event) {  
	    final int viewFlags = mViewFlags;  
	    if ((viewFlags & ENABLED_MASK) == DISABLED) {  
	        // A disabled view that is clickable still consumes the touch  
	        // events, it just doesn't respond to them.  
	        return (((viewFlags & CLICKABLE) == CLICKABLE ||  
	                (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE));  
	    }  
	    if (mTouchDelegate != null) {  
	        if (mTouchDelegate.onTouchEvent(event)) {  
	            return true;  
	        }  
	    }  
	    if (((viewFlags & CLICKABLE) == CLICKABLE ||  
	            (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE)) {  
	        switch (event.getAction()) {  
	            case MotionEvent.ACTION_UP:  
	                boolean prepressed = (mPrivateFlags & PREPRESSED) != 0;  
	                if ((mPrivateFlags & PRESSED) != 0 || prepressed) {  
	                    // take focus if we don't have it already and we should in  
	                    // touch mode.  
	                    boolean focusTaken = false;  
	                    if (isFocusable() && isFocusableInTouchMode() && !isFocused()) {  
	                        focusTaken = requestFocus();  
	                    }  
	                    if (!mHasPerformedLongPress) {  
	                        // This is a tap, so remove the longpress check  
	                        removeLongPressCallback();  
	                        // Only perform take click actions if we were in the pressed state  
	                        if (!focusTaken) {  
	                            // Use a Runnable and post this rather than calling  
	                            // performClick directly. This lets other visual state  
	                            // of the view update before click actions start.  
	                            if (mPerformClick == null) {  
	                                mPerformClick = new PerformClick();  
	                            }  
	                            if (!post(mPerformClick)) {  
	                                performClick();  
	                            }  
	                        }  
	                    }  
	                    if (mUnsetPressedState == null) {  
	                        mUnsetPressedState = new UnsetPressedState();  
	                    }  
	                    if (prepressed) {  
	                        mPrivateFlags |= PRESSED;  
	                        refreshDrawableState();  
	                        postDelayed(mUnsetPressedState,  
	                                ViewConfiguration.getPressedStateDuration());  
	                    } else if (!post(mUnsetPressedState)) {  
	                        // If the post failed, unpress right now  
	                        mUnsetPressedState.run();  
	                    }  
	                    removeTapCallback();  
	                }  
	                break;  
	            case MotionEvent.ACTION_DOWN:  
	                if (mPendingCheckForTap == null) {  
	                    mPendingCheckForTap = new CheckForTap();  
	                }  
	                mPrivateFlags |= PREPRESSED;  
	                mHasPerformedLongPress = false;  
	                postDelayed(mPendingCheckForTap, ViewConfiguration.getTapTimeout());  
	                break;  
	            case MotionEvent.ACTION_CANCEL:  
	                mPrivateFlags &= ~PRESSED;  
	                refreshDrawableState();  
	                removeTapCallback();  
	                break;  
	            case MotionEvent.ACTION_MOVE:  
	                final int x = (int) event.getX();  
	                final int y = (int) event.getY();  
	                // Be lenient about moving outside of buttons  
	                int slop = mTouchSlop;  
	                if ((x < 0 - slop) || (x >= getWidth() + slop) ||  
	                        (y < 0 - slop) || (y >= getHeight() + slop)) {  
	                    // Outside button  
	                    removeTapCallback();  
	                    if ((mPrivateFlags & PRESSED) != 0) {  
	                        // Remove any future long press/tap checks  
	                        removeLongPressCallback();  
	                        // Need to switch from pressed to not pressed  
	                        mPrivateFlags &= ~PRESSED;  
	                        refreshDrawableState();  
	                    }  
	                }  
	                break;  
	        }  
	        return true;  
	    }  
	    return false;  
		}  
 
这段代码比较复杂，不过我们看MotionEvent.ACTION_UP这个case中当手指抬起的时候里面会执行performClick()方法 
 
	public boolean performClick() {  
    sendAccessibilityEvent(AccessibilityEvent.TYPE_VIEW_CLICKED);  
    if (mOnClickListener != null) {  
        playSoundEffect(SoundEffectConstants.CLICK);  
        mOnClickListener.onClick(this);  
        return true;  
    }  
    return false;  
	}  
在这里我们见到了我们熟悉的onClick，那么一切都很清楚了，当触摸事件到达view的时候,执行dispatchTouchEvent--->setOnTouchListener--->onTouch--->onTouchEvent--->onClick,如果在onTouch方法中通过返回true将事件消费掉，onTouchEvent将不会再执行。 
 
另外需要注意的是，onTouch能够得到执行需要两个前提条件，第一mOnTouchListener的值不能为空，第二当前点击的控件必须是enable的。因此如果你有一个控件是非enable的，那么给它注册onTouch事件将永远得不到执行。对于这一类控件，如果我们想要监听它的touch事件，就必须通过在该控件中重写onTouchEvent方法来实现。
####我们了解完view的事件分发，接下来开始介绍ViewGroup的事件分发
ViewGroup比起view多了一个拦截方法onInterceptTouchEvent(MotionEvent event)，主要是三步 
 
* dispatchTouchEvent(MotionEvent event);
* onInterceptTouchEvent(MotionEvent event);
* onTouchEvent();
 
用一段伪代码解释三者的关系就是  

	public boolean dispatchTouchEvent(MotionEvent event) {
        boolean consume = false;
        if (onInterceptTouchEvent(event)) {
            consume = onTouchEvent(event);
        } else {
            consume = child.dispatchTouchEvent(event);
        }
 
        return consume;
    }

consume表示是否消费此事件，从这段伪代码中，我们可以看出来，在dispatchTouchEvent中，先调用ViewGroup自身的onInterceptTouchEvent方法，判断自己是否要拦截，如果这时候自己拦截，那就调用自己的onTouchEvent方法，如果onTouchEvent方法返回了True，那么这次的事件就算消耗了，事件传递到此为止，如果返回了False，证明这次没有消耗这次MotionEvent，那么这次的事件就会往上返回，由上一级继续处理；如果当前ViewGroup的onInterceptTouchEvent返回了False，那就会调用它的子view的dispatchTouchEvent方法，这样这个事件就传递下去了，如果它的子View处理不了，那么还会回来调用ViewGroup的onTouchEvent方法。 
 
* 一个view一旦拦截一个某个事件，当前事件所在的完整事件序列将都会由这个view去处理，反应在真实的代码中，就是一旦view拦截了down事件，那么此后的move和up事件都将不调用onInterceptTouchEvent，而直接由它处理，这就也意味着在onInterceptTouchEvent处理事件是不合适的，因为有可能来了事件，却直接跳过onInterceptTouchEvent方法。这个也意味着，一旦一个ViewGroup没有拦截ACTION_DOWN，那么这个事件序列的其他Action，它都将收不到，所以在处理ACTION_DOWN的时候，尤其需要谨慎。
* ViewGroup默认不拦截任何事件，源码中ViewGroup的onInterceptTouchEvent方法默认返回的是false
* 整个事件分发，看起来都是由外向内传递的，父View将事件传递给子View，理论上来看，子View是没有办法影响到父View的事件处理的，但是有一个标示位，requestDisallowInterceptTouchEvent方法，通过这个方法 ，子View能够影响父view的事件处理，这个可以用于解决父view和子view的滑动冲突，具体想了解的可以搜索它的相关用法，这里将不进行展开。