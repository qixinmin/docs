#trycatch性能的简单测试
在写一段逻辑的时候发现，为了获取一个路径，为了不让应用崩溃，写了一堆ifelse判断来保证健壮性
首先抛开其他的逻辑限定，只针对这部分来说，下面的代码
```
if(mDfToutiaoNewsitem.bigPictures != null && mDfToutiaoNewsitem.bigPictures.size() > 0) {
    DfToutiaoNewsItem.Thumbnail thumbnail = mDfToutiaoNewsitem.bigPictures.get(0);
    if(thumbnail != null) {
        imageUrl = thumbnail.src;//mDfToutiaoNewsitem.getImageUrl();
    } else {
        imageUrl = "";
    }

}else {
    imageUrl = "";
}
```
写了三个判断来保证健壮性，觉得不好，如果采用下面的写法
```
try {
    DfToutiaoNewsItem.Thumbnail thumbnail = mDfToutiaoNewsitem.bigPictures.get(0);
    imageUrl = thumbnail.src;
}catch (Exception e) {
    imageUrl = "";
}
``` 
感觉简单明了，但是担心性能的问题，于是写了一个测试来检查效果如何， 测试代码如下：
```
package com.browser2345.module.news.view;

import com.browser2345.homepages.dftoutiao.model.DfToutiaoNewsItem;

import java.util.ArrayList;

/**
 * Created by xinmin on 16/02/2017.
 */

public class TestTryCatch {

    DfToutiaoNewsItem mDfToutiaoNewsitem = new DfToutiaoNewsItem();
    String imageUrl = null;

    public TestTryCatch(){
        mDfToutiaoNewsitem.bigPictures = new ArrayList<>();
        DfToutiaoNewsItem.Thumbnail thumbnail = new DfToutiaoNewsItem.Thumbnail();
        mDfToutiaoNewsitem.bigPictures.add(thumbnail);
    }

    private void testif() {

        if(mDfToutiaoNewsitem.bigPictures != null && mDfToutiaoNewsitem.bigPictures.size() > 0) {
            DfToutiaoNewsItem.Thumbnail thumbnail = mDfToutiaoNewsitem.bigPictures.get(0);
            if(thumbnail != null) {
                imageUrl = thumbnail.src;//mDfToutiaoNewsitem.getImageUrl();
            } else {
                imageUrl = "";
            }

        }else {
            imageUrl = "";
        }
    }

    private void  testTrr() {
        try {
            DfToutiaoNewsItem.Thumbnail thumbnail = mDfToutiaoNewsitem.bigPictures.get(0);
            imageUrl = thumbnail.src;
        }catch (Exception e) {
            imageUrl = "";
        }
    }

    public static void main(String[] args) {
        int loopsize = 100000;
        TestTryCatch test = new TestTryCatch();
        long startTime = System.currentTimeMillis();
        for(int i=0; i<loopsize; ++i){
            test.testif();
        }
        long endTime = System.currentTimeMillis();
        for(int i=0; i<loopsize; ++i) {
            test.testTrr();
        }
        long endTime2 = System.currentTimeMillis();
        System.out.println("iftime:" + (endTime - startTime)+"ms");
        System.out.println("trytime:" + (endTime2 - endTime)+"ms");
    }
}

```结果如下：
| Tables        | 正常           | 异常  |
| ------------- |:-------------:| -----:|
| iftest| 4ms | 3ms|
| trytest |3ms |147ms|

