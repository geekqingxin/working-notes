参考链接：[http://daringfireball.net/projects/markdown/syntax#html](http://daringfireball.net/projects/markdown/syntax#html)




# Block elements

This is a regular paragraph.
<table>
	<tr>
		<th>姓名</th>
		<th>籍贯</th>
	</tr>
	<tr>
		<td>刘艳峰</td>
		<td>河南 安阳</td>
	</tr>
	<tr>
		<td>刘明月</td>
		<td>河南 焦作</td>
	</tr>
	<tr>
		<td>刘艳飞</td>
		<td>河南 安阳</td>
	</tr>
</table>


&copy;

4 < 5

## Headers

# 一级标题
## 二级标题
### 三级标题
#### 四级标题


## Blockquotes
> Markdown uses email-style > characters for blockquoting. If you're familiar with
> quoting passages of text in an email message, then you know how to create  a
> blockquote in Markdown. It looks best if you hard wrap the text and put a >
> before every line.

> 少年强，则国强。
> > 康有为 <少年说>


> # 一级标题
> 1. xxx
> 2. xxx
> 

> Here's some example code:

>     return shell_exec("echo $input | $markdown_script");


## Lists

* Red
* Green
* Blue

is equivalent to:

+ Red
+ Green
+ Blue

and:

- Red
- Green
- Blue

### Ordered lists
1. Bird
2. Mchale
3. Parish


## Code blocks

This is a normal paragraph.


1. abc

    This is a code block.

## Horizontal rules

***

* * *

*****

- - -

------------


# Span Elements

## Links

### inline

This is [an example](http://www.baidu.com) inline like.

### reference

[baidu]: http://www.baidu.com "带您到百度去"

This is [an example][baidu] reference-style link.


## Emphasis (重点，强调)

*single asterisks*

_signle underscores_

**double asterisks**

__double underscores__


## Code

Use the `printf()` function.


## Images

### inline

Html 5 LOGO: ![Alt Html5](h5.jpg)

### reference
[baidu_logo]: https://www.baidu.com/img/bd_logo1.png

![Alt baidu logo][baidu_logo]


# Miscellaneous


## Automatic links

link: <http://www.baidu.com>

email: <liuyanfeng@taione.cn>

## Backslash escapes

\*literal asterisks\*

	\	backslash
	`	backtick
	*	asterisk
	_	underscore
	{}	curly braces
	[]	square brackets
	()	parentheses
	#	hash mark
	+	plus sign
	-	minus sign (hyphen)
	.	dot
	!	exclamation mark
