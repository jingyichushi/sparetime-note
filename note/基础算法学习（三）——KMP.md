# 基础算法学习（三）——KMP

## KMP算法求解的问题

给定两个字符串str和match，长度分别为N和M。实现一个算法，如果字符串str中含有子串match，则返回match在str中的开始位置。不含有则返回-1.

例如：

str="acbc"，match="bc"，返回2；

str="acbc"，match="bcc"，返回-1；

要求：如果match的长度大于str的长度（M>N），str必然不会含有match，可直接返回-1.但如果N≥M，要求算法复杂度为O（N）.



### 暴力求解

最普通的解法是从左到右遍历str的每一个字符，然后看如果以当前字符作为第一个字符开始出发是否匹配match，普通算法的时间复杂度较高，从每个字符出发时，匹配的代价都可能是O(N),那么一共有N个字符，所以整体的时间复杂度为O(N*M)。普通解法的时间复杂度这么高，是因为每次遍历到一个字符时，检查工作相当于从无开始，之前的遍历检查不能优化当前的遍历检查。



## KMP求解

### KMP基本过程

首先生成match字符串的nextArr数组，这个数组的长度与match字符串的长度一样，nextArr[i]的含义是在match[i]之前的字符串match[0...i-1]中，必须以match[i-1]结尾的后缀子串（不能包含match[0]）与必须以match[0]开头的前缀子串（不能包含match[i-1]）最大的匹配长度是多少。这个长度就是nextArr[i]的值。



假设从str[i]字符出发时，匹配到j位置的字符发现与match中的字符不一致。也就是说，str[i]与match[0]一样，并且从这个位置开始一直可以匹配，即str[i,,,j-1]与match[0...,j-i-1]一样，知道发现str[j]!=match[j-1]，匹配停止。因为现在已经有了match字符串的nextArr数组，nextArr[j-1]的值表示match[0...j-i-1]这一段字符串前缀和后缀的最大匹配。下一次直接让str[j]与match[k]进行匹配检查。对于match来说，相当于向右滑动，让match[k]滑动太str[j]同一个位置上，然后进行后续的匹配检查。直到在str的某一个位置把match完全匹配完，就说明str中有match。如果match滑到最后也没有匹配出来，就说明str中没有match.



匹配过程分析完毕，str中匹配的位置是不退回的，match则一直向右滑动，如果在str中的某个位置完全匹配出match，整个过程停止。否则match滑到str的最右侧过程也停止，所以滑动的长度最大为N，所以时间复杂度为O(N)。



### 实现效果

![](http://img.bcoder.top/2019.01.22/1.gif)



### nextArr数组生成过程

如何快速得到match字符串的nextArr数组，并且要证明得到nextArr数组的时间复杂度为O(M)。对于match[0]来说，在它之前，没有字符，所以nextArr[0]规定为-1。对于match[1]来说，在它之前有match[0]，但nextArr数组的定义要求任何子串的后缀不能包括第一个字符（match[0]），所以match[1]之前的字符串只有长度为0的后缀字符串，所以nextArr[1]为0



对`match[i](i>1)`来说，求解过程如下：

因为从左到右依次求解nextArr，所以在求解nextArr[i]时，nextArr[0...i-1]的值都已经求出。通过nextArr[i-1]的值可以知道B字符串的最长前缀和后缀匹配区域。设L区域为最长匹配的前缀子串，K区域为最长匹配的后缀子串，C为L区域之后的字符，B为K区域之后的字符，A是B字符之后的字符。然后查看字符C与字符B是否相等。
如果字符C与字符B相等，那么A字符之前的字符串的最长前缀和后缀匹配区域就可以确定，前缀子串为L区域+C字符，后缀子串为K区域+B字符，即nextArr[i] = nextArr[i-1]+1.

如果字符C与字符B不相等，就看字符C之前的前缀和后缀匹配情况，假设字符C是第cn个字符（match[cn]），那么nextArr[cn]就是其最长前缀和后缀匹配的长度。m区域和n区域分别是字符C之前的字符串的最长匹配的后缀与前缀区域，这是通过nextArr[cn]的值确定的。当然两个区域是相等的，m'区域为k区域最后的区域且长度与m区域一样，因为k区域和L区域是相等的，所以m区域和m'区域也是相等的，字符D为n区域之后的第一个字符，接下来比较字符D是否与字符B相等。



如果相等，A字符之前的字符串的最长前缀与后缀匹配区域就可以确定，前缀子串为n区域+D字符，后缀子串为m'区域+B字符，则令nextArr[i]=nextArr[cn]+1。
如果不等，继续往前跳到字符D，之后的过程与跳到字符C类似，一直进行这样的跳过程，跳的每一步都会有一个新的字符和B比较（就像C字符和D字符一样），只要有相等的情况，nextArr[i]的值就能确定。



如果向前调到最左位置（即match[0]的位置），此时nextArr[0]=-1，说明字符A之前的字符串不存在前缀和后缀匹配的情况，则令nextArr[I]=0。用这种不断向前跳的方式可以算出正确的nextArr[I]值的原因还是因为每跳到一个位置cn，nextArr[cn]的意义就表示它之前字符串的最大匹配长度。



### KMP代码实现

``` java
public static int getIndexOf(String s, String m) {
	if (s == null || m == null || m.length() < 1 || s.length() < m.length()) {
		return -1;
	}
	char[] ss = s.toCharArray();
	char[] ms = m.toCharArray();
	int si = 0;
	int mi = 0;
	int[] next = getNextArray(ms);
	while (si < ss.length && mi < ms.length) {
		if (ss[si] == ms[mi]) {
			si++;
			mi++;
		} else if (next[mi] == -1) {
			si++;
		} else {
			mi = next[mi];
		}
	}
	return mi == ms.length ? si - mi : -1;
}

public static int[] getNextArray(char[] ms) {
	if (ms.length == 1) {
		return new int[] { -1 };
	}
	int[] next = new int[ms.length];
	next[0] = -1;
	next[1] = 0;
	int pos = 2;
	int cn = 0;
	while (pos < next.length) {
		if (ms[pos - 1] == ms[cn]) {
			next[pos++] = ++cn;
		} else if (cn > 0) {
			cn = next[cn];
		} else {
			next[pos++] = 0;
		}
	}
	return next;
}
```
