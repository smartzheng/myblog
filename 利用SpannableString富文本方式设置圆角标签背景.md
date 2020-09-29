+++
title= "利用SpannableString富文本方式设置圆角标签背景"
date= "2018-03-28"
categories= [ "Android" ]
+++
#### 项目中遇到一个需求,需要在商品标题加上标签,而标签是客户可以后台配置的,所以不是用的图片,而是用的文字.如下图:  

![](https://upload-images.jianshu.io/upload_images/2983970-c45ecdf7ffd29eb2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/340)


####  众所周知,在Android中,View都是呈方形布置的,所以如果标签和文字如果不是同一个View,那么如果文字换行,就会出现标签和TextView分别在左右两边的效果:  

![](https://upload-images.jianshu.io/upload_images/2983970-276b1e708bb147d1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/340)



#### 经过思考和查阅资料,发现可以用SpannableString设置背景,并通过重写ReplacementSpan替换原来的BackgroundColorSpan来实现圆角标签.
#### 下面是代码:

Activity: (goodsTags为标签文字集合,即上图的"促","九","八"...goodsName为商品名字

```java
for (int i = 0; i < goodsTags.size(); i++) {
	//为了显示效果在每个标签文字前加两个空格,后面加三个空格(前两个和后两个填充背景,最后一个作标签分割)
    goodsName.insert(0, "  " + goodsTags.get(i).getTags_name() + "   ");
    int start = 0;
    int end = 5;
    //稍微设置标签文字小一点
    goodsName.setSpan(new RelativeSizeSpan(0.9f), start, end, Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);
    //设置圆角背景
    goodsName.setSpan(new RoundBackgroundColorSpan(Color.parseColor("#" + goodsTags.get(i).getTags_color()),Color.WHITE), start, end, Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);
}
```

自定义RoundBackgroundColorSpan:

```java
public class RoundBackgroundColorSpan extends ReplacementSpan {
    private int bgColor;
    private int textColor;
    public RoundBackgroundColorSpan(int bgColor, int textColor) {
        super();
        this.bgColor = bgColor;
        this.textColor = textColor;
    }
    @Override
    public int getSize(Paint paint, CharSequence text, int start, int end, Paint.FontMetricsInt fm) {
   		//设置宽度为文字宽度加16dp
        return ((int)paint.measureText(text, start, end)+Util.px2Dp(16));
    }

    @Override
    public void draw(Canvas canvas, CharSequence text, int start, int end, float x, int top, int y, int bottom, Paint paint) {
        int originalColor = paint.getColor();
        paint.setColor(this.bgColor);
        //画圆角矩形背景
        canvas.drawRoundRect(new RectF(x,
                top+ Util.px2Dp(3),
                x + ((int) paint.measureText(text, start, end)+ Util.px2Dp(16)),
                bottom-Util.px2Dp(1)),

                Util.px2Dp(4),
                Util.px2Dp(4),
                paint);
        paint.setColor(this.textColor);
        //画文字,两边各增加8dp
        canvas.drawText(text, start, end, x+Util.px2Dp(8), y, paint);
        //将paint复原
        paint.setColor(originalColor);
    }
}
```