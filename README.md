public class RecyclerViewDecoration extends RecyclerView.ItemDecoration {
    private ArrayList<ItemInfo> mItemInfos;
    private int mItemMargin;
    private int mItemLabelHeight;
    private int mItemDividerHeight;
    private int mItemLabelTextSize;
    private int mItemLabelTextTopPadding;
    private int mItemGapHeight;
    private int mLabelTextColor;
    private int mFloatCircleColor;
    private int mFloatCircleRadius;
    private boolean mReverse = true;

    private Paint mLabelTextPaint;
    private Paint mFloatCirclePaint;
    private Paint mLabelAreaPaint;
    private Paint mFloatCircleTextPaint;

    private Drawable mItemDivider;
    private Drawable mItemGap;

    public RecyclerViewDecoration(Context context, ArrayList<ItemInfo> itemInfos) {
        this.mItemInfos = itemInfos;

        mItemMargin = context.getResources().getDimensionPixelSize(R.dimen.item_margin);
        mItemLabelHeight = context.getResources().getDimensionPixelSize(R.dimen.item_label_height);
        mItemDividerHeight = context.getResources().getDimensionPixelSize(R.dimen.item_divider_height);
        mItemLabelTextTopPadding = context.getResources().getDimensionPixelSize(R.dimen.item_label_text_top_padding);
        mItemGapHeight = context.getResources().getDimensionPixelSize(R.dimen.item_gap_height);

        mItemLabelTextSize = context.getResources().getDimensionPixelSize(R.dimen.item_label_text_size);
        mLabelTextColor = context.getResources().getColor(R.color.labelTextColor);
        mFloatCircleColor = context.getResources().getColor(R.color.itemGapColor);

        mFloatCircleRadius = 3 * mItemLabelTextSize / 2;

        mLabelTextPaint = new Paint();
        mLabelTextPaint.setColor(mLabelTextColor);
        mLabelTextPaint.setTypeface(Typeface.DEFAULT_BOLD);
        mLabelTextPaint.setTextSize(mItemLabelTextSize);

        mLabelAreaPaint = new Paint();
        mLabelAreaPaint.setColor(Color.WHITE);

        mFloatCirclePaint = new Paint();
        mFloatCirclePaint.setColor(mFloatCircleColor);


        mFloatCircleTextPaint = new Paint();
        mFloatCircleTextPaint.setColor(Color.WHITE);
        mFloatCircleTextPaint.setTypeface(Typeface.DEFAULT_BOLD);
        mFloatCircleTextPaint.setTextSize(2 * mItemLabelTextSize);

        if (context instanceof Activity) {
            Activity activity = (Activity) context;
            mItemDivider = activity.getResources().getDrawable(R.drawable.recycler_view_divider, null);
            mItemGap = activity.getResources().getDrawable(R.drawable.recycler_view_gap, null);
        }
    }

    private void drawGroupViewReverse(Canvas c, RecyclerView parent) {
        int childCount = parent.getChildCount();
        int itemCount = mItemInfos.size();
        //Log.d("lk", "drawGroupView childCount =  " + childCount);
        for (int i = 0; i < childCount; i++) {
            View itemView = parent.getChildAt(i);
            int j = parent.getChildAdapterPosition(itemView);
            ItemInfo itemInfo = mItemInfos.get(j);
            //第0个位置不需要绘制
            if (j < itemCount - 1) {
                ItemInfo nextItemInfo = mItemInfos.get(j + 1);
                if (nextItemInfo.getAppNameInPinyin().charAt(0) != itemInfo.getAppNameInPinyin().charAt(0)) {
                    //需要绘item上方item gap、label title及divider分隔线
                    if ((nextItemInfo.getAppNameInPinyin().charAt(0) >= 'a' && nextItemInfo.getAppNameInPinyin().charAt(0) <= 'z')
                            || (itemInfo.getAppNameInPinyin().charAt(0) >= 'a' && itemInfo.getAppNameInPinyin().charAt(0) <= 'z')) {
                        mItemGap.setBounds(0, itemView.getBottom() + mItemLabelHeight + mItemDividerHeight,
                                parent.getWidth(), itemView.getBottom() + mItemLabelHeight + mItemDividerHeight + mItemGapHeight);
                        mItemGap.draw(c);
                        mLabelTextPaint.setAntiAlias(true);
                        char labelTitle = Character.toUpperCase(itemInfo.getAppNameInPinyin().charAt(0));
                        if (!Character.isLetter(labelTitle)) {
                            labelTitle = '#';
                        }
                        c.drawText("" + labelTitle, mItemMargin,
                                itemView.getBottom() + mItemDividerHeight + mItemLabelTextTopPadding, mLabelTextPaint);
                        mLabelTextPaint.setAntiAlias(false);
                        mItemDivider.setBounds(mItemMargin, itemView.getBottom(), itemView.getWidth() + mItemMargin, itemView.getBottom() + mItemDividerHeight);
                        mItemDivider.draw(c);
                    }
                } else {
                    //仅绘item下方的divider分隔线
                    mItemDivider.setBounds(mItemMargin, itemView.getBottom(), itemView.getWidth() + mItemMargin, itemView.getBottom() + mItemDividerHeight);
                    mItemDivider.draw(c);
                }
            } else if (j == itemCount - 1) {
                mItemGap.setBounds(0, itemView.getBottom() + mItemLabelHeight + mItemDividerHeight,
                        parent.getWidth(), itemView.getBottom() + mItemLabelHeight + mItemDividerHeight + mItemGapHeight);
                mItemGap.draw(c);
                mLabelTextPaint.setAntiAlias(true);
                char labelTitle = Character.toUpperCase(itemInfo.getAppNameInPinyin().charAt(0));
                if (!Character.isLetter(labelTitle)) {
                    labelTitle = '#';
                }
                c.drawText("" + labelTitle, mItemMargin,
                        itemView.getBottom() + mItemDividerHeight + mItemLabelTextTopPadding, mLabelTextPaint);
                mLabelTextPaint.setAntiAlias(false);
                mItemDivider.setBounds(mItemMargin, itemView.getBottom(), itemView.getWidth() + mItemMargin, itemView.getBottom() + mItemDividerHeight);
                mItemDivider.draw(c);
            }
        }
    }

    @Override
    public void onDraw(Canvas c, RecyclerView parent, RecyclerView.State state) {
        super.onDraw(c, parent, state);
        drawGroupViewReverse(c, parent);
    }

    private void drawFlaotView(Canvas c, RecyclerView parent) {
        int lastChildIndex = parent.getChildCount() - 1;
        View lastChild = parent.getChildAt(lastChildIndex);
        ItemInfo lastItemInfo = mItemInfos.get(parent.getChildAdapterPosition(lastChild));
        char floatLabelTitle = Character.toUpperCase(lastItemInfo.getAppNameInPinyin().charAt(0));
        int floatLabelTranslationY = 0;
        int screenHeight = Utilities.getDisplayMetrics(parent.getContext()).heightPixels;
        char floatCircleTitle = lastItemInfo.getAppNameInDefault().charAt(0);
        int childCount = parent.getChildCount();
        //screenHeight = screenHeight - 75;
        for (int i = 0; i < parent.getChildCount(); i++) {
            View childView = parent.getChildAt(i);
            int adapterPosition = parent.getChildAdapterPosition(childView);
            if (adapterPosition == mItemInfos.size() - 1) {
                continue;
            }
            //找到当前recyclerView中第一个带有Label的Item
            if (parent.findChildViewUnder(mItemMargin + childView.getWidth() / 2,
                    childView.getBottom() + mItemDividerHeight + mItemLabelHeight / 2) == null) {
                //第二个Label将第一个Label顶上去的情况
                if ((childView.getBottom() > screenHeight - 75 - 2 * mItemLabelHeight - mItemDividerHeight)
                        && childView.getBottom() < screenHeight - 75) {
                    floatLabelTitle = Character.toUpperCase(mItemInfos.get(adapterPosition).getAppNameInPinyin().charAt(0));
                    floatLabelTranslationY = 2 * mItemLabelHeight - (screenHeight - childView.getBottom());
                    Log.d("lk", "floatLabelTranslationY = " + floatLabelTranslationY +
                            ",bottom = " + childView.getBottom() + ",title = " + mItemInfos.get(adapterPosition).getAppNameInDefault());
                    break;
                }
            }
        }
        c.drawRect(new Rect(0, screenHeight - mItemLabelHeight - 75 + floatLabelTranslationY,
                parent.getWidth(), screenHeight - 75 + floatLabelTranslationY), mLabelAreaPaint);
        mItemDivider.setBounds(0, screenHeight - mItemLabelHeight - mItemDividerHeight - 75 + floatLabelTranslationY,
                parent.getWidth(), screenHeight - mItemLabelHeight - 75);
        mItemDivider.draw(c);
        mLabelTextPaint.setAntiAlias(true);
        if (!Character.isLetter(floatLabelTitle)) {
            floatLabelTitle = '#';
        }
        c.drawText("" + floatLabelTitle, mItemMargin, mItemLabelTextTopPadding + screenHeight - mItemLabelHeight - 75 + floatLabelTranslationY, mLabelTextPaint);
        mLabelTextPaint.setAntiAlias(false);

        /*if (parent.getScrollState() != RecyclerView.SCROLL_STATE_IDLE) {
            mFloatCirclePaint.setAntiAlias(true);
            c.drawCircle(Utilities.getDisplayMetrics(parent.getContext()).widthPixels / 2,
                    Utilities.getDisplayMetrics(parent.getContext()).heightPixels / 2, mFloatCircleRadius, mFloatCirclePaint);
            mFloatCirclePaint.setAntiAlias(false);
            mFloatCircleTextPaint.setAntiAlias(true);
            //设置文本水平左右居中
            mFloatCircleTextPaint.setTextAlign(Paint.Align.CENTER);
            Paint.FontMetrics fm = mFloatCircleTextPaint.getFontMetrics();
            float y = Utilities.getDisplayMetrics(parent.getContext()).heightPixels / 2 - fm.descent + (fm.bottom - fm.top) / 2;
            c.drawText("" + floatCircleTitle, Utilities.getDisplayMetrics(parent.getContext()).widthPixels / 2,
                    y, mFloatCircleTextPaint);
            mFloatCircleTextPaint.setAntiAlias(false);
        }*/
    }

    @Override
    public void onDrawOver(Canvas c, RecyclerView parent, RecyclerView.State state) {
        super.onDrawOver(c, parent, state);
        drawFlaotView(c, parent);
    }

    @Override
    public void getItemOffsets(Rect outRect, View view, RecyclerView parent, RecyclerView.State state) {
        super.getItemOffsets(outRect, view, parent, state);
        int count = mItemInfos.size();
        int childPosition = parent.getChildAdapterPosition(view);
        ItemInfo itemInfo = mItemInfos.get(childPosition);
        if (childPosition == count - 1) {
            outRect.set(0, 0, 0, mItemLabelHeight + mItemDividerHeight);
        } else {
            ItemInfo nextItemInfo = mItemInfos.get(childPosition + 1);

            if (nextItemInfo.getAppNameInPinyin().charAt(0) != itemInfo.getAppNameInPinyin().charAt(0)) {
                if ((nextItemInfo.getAppNameInPinyin().charAt(0) >= 'a' && nextItemInfo.getAppNameInPinyin().charAt(0) <= 'z')
                        || (itemInfo.getAppNameInPinyin().charAt(0) >= 'a' && itemInfo.getAppNameInPinyin().charAt(0) <= 'z')) {
                    outRect.set(0, 0, 0, mItemGapHeight + mItemLabelHeight + mItemDividerHeight);
                }
            } else {
                outRect.set(0, 0, 0, mItemDividerHeight);
            }
        }
    }
}
