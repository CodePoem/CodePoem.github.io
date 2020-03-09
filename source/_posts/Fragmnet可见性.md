---
title: Fragment可见性
date: 2020-03-05 22:40:14
updated: 2020-03-05 22:40:08
categories:
- Android杂录
tags:
- Fragment
- 可见性
---

```java
class ExampleFragment extends Fragment{
    /**
         * 当fragment与viewpager、FragmentPagerAdapter一起使用时，切换页面时会调用此方法
         *
         * @param isVisibleToUser 是否对用户可见
         */
        @Override
        public void setUserVisibleHint(boolean isVisibleToUser) {
            boolean change = isVisibleToUser != getUserVisibleHint();
            super.setUserVisibleHint(isVisibleToUser);
            // 在viewpager中，创建fragment时就会调用这个方法，但这时还没有resume，为了避免重复调用visible和invisible，
            // 只有当fragment状态是resumed并且初始化完毕后才进行visible和invisible的回调
            if (isResumed() && change) {
                if (getUserVisibleHint()) {
                    onVisible();
                } else {
                    onInvisible();
                }
            }
        }
    
        /**
         * 当使用show/hide方法时，会触发此回调
         *
         * @param hidden fragment是否被隐藏
         */
        @Override
        public void onHiddenChanged(boolean hidden) {
            super.onHiddenChanged(hidden);
            if (hidden) {
                onInvisible();
            } else {
                onVisible();
            }
        }
    
    
        @Override
        public void onResume() {
            super.onResume();
            // onResume并不代表fragment可见
            // 如果是在viewpager里，就需要判断getUserVisibleHint，不在viewpager时，getUserVisibleHint默认为true
            // 如果是其它情况，就通过isHidden判断，因为show/hide时会改变isHidden的状态
            // 所以，只有当fragment原来是可见状态时，进入onResume就回调onVisible
            if (getUserVisibleHint() && !isHidden()) {
                onVisible();
            }
        }
    
        @Override
        public void onPause() {
            super.onPause();
            // onPause时也需要判断，如果当前fragment在viewpager中不可见，就已经回调过了，onPause时也就不需要再次回调onInvisible了
            // 所以，只有当fragment是可见状态时进入onPause才加调onInvisible
            if (getUserVisibleHint() && !isHidden()) {
                onInvisible();
            }
        }
    
        private void onInvisible() {
        }
    
        private void onVisible() {
            initData();
        }
}

```