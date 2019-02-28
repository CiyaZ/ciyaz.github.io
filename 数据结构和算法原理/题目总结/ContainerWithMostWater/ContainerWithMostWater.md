# 题目

Given n non-negative integers a1, a2, ..., an, where each represents a point at coordinate (i, ai). n vertical lines are drawn such that the two endpoints of line i is at (i, ai) and (i, 0). Find two lines, which together with x-axis forms a container, such that the container contains the most water.

Note: You may not slant the container and n is at least 2.

中文意思：x轴上有n条竖线，高度分别为a1,a2...等，求由两条竖线和x轴围成的最大面积。

# 初步思考

遍历穷举可以解决这个问题。时间复杂度为O(n^2)，但遗憾的是提交时运行超时了。

```c
#define min(a,b) (((a) < (b)) ? (a) : (b))

int maxArea(int* height, int heightSize)
{
    int max = 0;
    for(int i = 0; i < heightSize; i++)
    {
        for(int j = i + 1; j < heightSize; j++)
        {
            int temp = min(height[j], height[i]) * (j - i);
            if(temp > max)
            {
                max = temp;
            }
        }
    }
    return max;
}
```

# 线性时间复杂度算法

假设我们找到能取最大容积的纵线为 i , j (设i<j)，那么得到的最大容积 `Sij = min（ai，aj）*（j-i）`

可以证明：i左边的所有竖线都比i短，j右边的所有竖线都比j短

* 若存在`k<i`，`ak>ai`，则`Skj=min（ak，aj）*（j-k）`，这种情况下：
  * 显然`（j-k）>（j-i）`
  * 若`min（ai，aj）=ai`，`ak>ai`，则`min（ak，aj）=ak`或`aj>min（ai，aj）=ai`
  * 若`min（ai，aj）=aj`，则`min（ak，aj）=aj=min（ai，aj）=aj`
* 显然`Skj=min（ak，aj）*（j-k）>Sij=min（ai，aj）*（j-i）`，所以这样的k不存在
* 同理可证，j右边也不存在比aj大的ak

所以，若当前的两条竖线为x，y（设x<y），则可能存在的围成更大面积的两条竖线x'和y'一定在`[x，y]`内。且`x'>x`，`y'>y`。

算法基本步骤：

i，j两个索引分别指向数组两边，向中间靠拢，收缩过程中优先从ai，aj中较小的边开始。

```c
#define max(a,b) (((a) > (b)) ? (a) : (b))
#define min(a,b) (((a) < (b)) ? (a) : (b))

int maxArea(int* height, int heightSize)
{
	int result = 0;
	int i = 0;
	int j = heightSize - 1;
	while(i < j)
	{
		result = max(result, min(height[i], height[j]) * (j - i));
		if(height[i] < height[j])
		{
			int k = i;
			while(height[k] <= height[i] && k < j)
			{
				k++;
			}
			i = k;
		}
		else
		{
			int k = j;
			while(height[k] <= height[j] && k > i)
			{
				k--;
			}
			j = k;
		}
	}
	return result;
}
```
