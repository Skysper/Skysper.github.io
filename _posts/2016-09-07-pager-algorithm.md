---
layout: post
tId: 1609071
title: "分页算法"
date: 2016-09-07 13:00:00 +0800
categories: 分页,算法
codelang: csharp
desc: "Web应用中，最常见的就是分页算法了，本文介绍下通过另外一种思路实现的分页算法"
---

这是一个经过简单重构的分页算法辅助类，并实现了html的拼接输出。当然，这个应该经过更进一步的重构，可以进行更进一步的定制化。
分页在Web的前后端引用中，太常见了，如果项目时间比较紧张，可以直接套用。

```
    public class HtmlPager
    {
        /// <summary>
        /// 上一页文本
        /// </summary>
        public string PrevText { set; get; }
        /// <summary>
        /// 下一页文本
        /// </summary>
        public string NextText { set; get; }
        /// <summary>
        /// 首页文本
        /// </summary>
        public string FirstText { set; get; }
        /// <summary>
        /// 尾页文本
        /// </summary>
        public string EndText { set; get; }

        private int _maxPage;
        /// <summary>
        /// 最大页码
        /// </summary>
        public int MaxPage
        {
            get
            {
                if (_maxPage == 0)
                {
                    _maxPage = (int)Math.Ceiling((decimal)TotalCounts / PageSize);
                }
                return _maxPage;
            }
        }

        /// <summary>
        /// url生成规则
        /// </summary>
        public Func<int, string> UrlGenerate { set; get; }
        /// <summary>
        /// 总数据数量
        /// </summary>
        public int TotalCounts { set; get; }
        /// <summary>
        /// 单页数据量
        /// </summary>
        public int PageSize { set; get; }

        private string OnUrlGenerate(int index)
        {
            if (UrlGenerate != null)
            {
                return UrlGenerate(index);
            }
            else {
                return string.Empty;
            }
        }


        public HtmlPager(int totalCounts, int pageSize, string prevText, string nextText, Func<int, string> func = null)
        {
            this.TotalCounts = totalCounts;
            this.PageSize = pageSize;
            this.PrevText = prevText;
            this.NextText = nextText;
            this.UrlGenerate = func;
        }

        public string GetString(int currentIndex)
        {
            //边界条件判断和处理
            if (MaxPage == 1) return "";
            if (currentIndex > MaxPage) currentIndex = MaxPage;
            if (currentIndex < 1) currentIndex = 1;
            //分页结果显示数量
            int showCount = 9;
            
            int offSetLeft = 0;
            int offSetRight = 0;
            //右边显示最大页码
            int rightMax = currentIndex + showCount / 2;
            //左边显示最大页码
            int leftMin = currentIndex - showCount / 2;
            //如果临近左边，显示不足一半，需要右边补充的数量
            offSetLeft = 1 - leftMin;
            offSetLeft = offSetLeft < 0 ? 0 : offSetLeft;
            //同样右边
            offSetRight = rightMax - MaxPage;
            offSetRight = offSetRight < 0 ? 0 : offSetRight;
            //判定处理offset
            //分页末尾页码区域
            if (offSetLeft > 0 && offSetRight == 0)
            {
                rightMax += offSetLeft;
                if (rightMax > MaxPage)
                {
                    rightMax = MaxPage;
                }
            }
            //分页起始页码区域
            else if (offSetRight > 0 && offSetLeft == 0)
            {
                leftMin -= offSetRight;
                if (leftMin < 1) leftMin = 1;
            }
            //统一的页码范围判定
            if (leftMin <= 0) leftMin = 1;
            if (rightMax > MaxPage) rightMax = MaxPage;
            //判定是否需要首尾“...”
            bool isAddFirst = false;
            bool isAddMax = false;
            if (showCount >= 7)
            {
                if (leftMin >= 2)
                {
                    leftMin = leftMin + 2;
                    isAddFirst = true;
                }
                if (rightMax <= MaxPage - 1)
                {
                    rightMax = rightMax - 2;
                    isAddMax = true;
                }
            }

            StringBuilder str = new StringBuilder();
            str.Append("<div>");
            if (currentIndex <= 1)
            {
                str.Append(string.Format("<span  class=\"preview_off\">{0}</span>", PrevText));
            }
            else
            {
                str.Append(string.Format("<a  class=\"preview_on\" href=\"{0}\">{1}</a>", OnUrlGenerate(currentIndex - 1), PrevText));
            }
            if (isAddFirst)
            {
                str.Append(string.Format("<a   href=\"{0}\">{1}</a>", OnUrlGenerate(1), 1));
                str.Append("<span  class=\"nolink\">...</span>");
            }
            for (int item = leftMin; item <= rightMax; item++)
            {
                if (item == currentIndex)
                {
                    str.Append(string.Format("<a class=\"linknow\">{0}</a>", item));
                    continue;
                }
                str.Append(string.Format("<a href=\"{0}\">{1}</a>", OnUrlGenerate(item), item));
            }
            if (isAddMax)
            {
                str.Append("<span  class=\"nolink\">...</span>");
                str.Append(string.Format("<a href=\"{0}\">{1}</a>", OnUrlGenerate(MaxPage), MaxPage));
            }
            if (currentIndex >= MaxPage)
            {
                str.Append(string.Format("<span class=\"next_off\" >{0}</span>", NextText));
            }
            else
            {
                str.Append(string.Format("<a class=\"next_on\"  href=\"{0}\">{1}</a>", OnUrlGenerate(currentIndex + 1), NextText));
            }
            str.Append("</div>");
            return str.ToString();
        }
    }

```

整个算法的核心关键点在于通过计算中心页面的左右显示极值，以及相应极值下左右两侧可能的偏移和页码补位。在得到起始页码和结束页码之后，便可以处理输出内容了。这也是可以再次重构的地方，可以将输出部分和算法部分分开处理，以便于针对不同的样式和结果输出进行控制。

效果[非代码直接效果，需修正]：
<ul class="pagination"> 
    <li><a href="javascript:;">&lt;</a></li>
    <li><a href="javascript:;">1</a></li> 
    <li class="disabled"><a href="javascript:;">...</a></li> 
    <li><a href="javascript:;">4</a></li> 
    <li><a href="javascript:;">5</a></li> 
    <li class="active"><a href="javascript:;">6</a></li> 
    <li><a href="javascript:;">7</a></li> 
    <li><a href="javascript:;">8</a></li> 
    <li class="disabled"><a href="javascript:;">...</a></li> 
    <li><a href="javascript:;">20</a></li> 
    <li><a href="javascript:;">&gt;</a></li> 
</ul>

