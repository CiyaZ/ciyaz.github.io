# yacc语法分析器

yacc，Yet Another Compiler Compiler。这个名字起的很是厉害。

yacc用作语法分析器，它接受词法分析器lex的输入，并将一系列token解析为抽象语法树（AST）。阅读本文前，需要先阅读lex章节的内容。本文将介绍一些关于yacc的简单使用。参考书为《Lex与Yacc》第二版。

yacc的GNU版叫做bison，两者基本用法相同。在我的操作系统中（Ubuntu14.04），bison已经预装了，yacc命令也是指向bison。

查看yacc(bison)工具的帮助
```
man bison
```

## yacc的使用

和学习lex一样，我们直接从例子入手，这个例子是参考书上实现的。书中的例子比较复杂，我这里稍微简化一下，容易理解。

英语中，单词词性有：NOUN（名词），PRONOUN（代词），VERB（动词），ADVERB（副词），ADJECTIVE（形容词），PREPOSITION（介词），CONJUNCTION（连词）。我们编写一个程序，识别一个序列是不是一个英文句子。我们的序列用固定的几个字符串组合表示，例如：`adj noun adv verb adj noun`根据英语语法，这个序列表示一个句子。

如何判定一个序列是一个句子呢？根据英语语法，我们简单的定义如下文法产生式：

```
sentence->simple_sentence|compound_sentence

simple_sentence->subject verb object|subject verb object prep_phrase

compound_sentence->simple_sentence CONJUNCTION simple_sentence|compound_sentence CONJUNCTION simple_sentence

subject->NOUN|PRONOUN|ADJECTIVE subject

verb->VERB|ADVERB VERB|verb VERB

object->NOUN|ADJECTIVE object

prep_phrase->PREPOSITION NOUN
```

注：上面的产生式中，大写单词表示lex识别的token（终结符）。小写的单词都是中间推导的过程（非终结符）。

理解了上面的规则，其实就好办了，我们只要按照yacc的文件格式，编写`.y`文件就行了。当然除此之外，我们还需要一个`.l`文件供lex使用。

english.l
```
%{
#include <stdio.h>
#include "y.tab.h"
%}
%%
verb {
	printf("VERB TOKEN\n");
	return VERB;
}
adj {
	printf("ADJECTIVE TOKEN\n");
	return ADJECTIVE;
}
adv {
	printf("ADVERB TOKEN\n");
	return ADVERB;
}
noun {
	printf("NOUN TOKEN\n");
	return NOUN;
}
prep {
	printf("PREPOSITION TOKEN\n");
	return PREPOSITION;
}
pron {
	printf("PRONOUN TOKEN\n");
	return PRONOUN;
}
conj {
	printf("CONJUCTION TOKEN\n");
	return CONJUNCTION;
}
[ \t]+ {}
%%
int yywrap(void)
{
	return 1;
}
```

english.y
```
%{
#include <stdio.h>
%}
%token NOUN PRONOUN VERB ADVERB ADJECTIVE PREPOSITION CONJUNCTION
%%
sentence: simple_sentence {
	printf("parsed simple sentence\n");
	}
	| compound_sentence {
		printf("parsed compound sentence\n");
	}
	;

simple_sentence: subject verb object
	| subject verb object prep_phrase
	;

compound_sentence: simple_sentence CONJUNCTION simple_sentence
	| compound_sentence CONJUNCTION simple_sentence
	;

subject: NOUN
	| PRONOUN
	| ADJECTIVE subject
	;

verb: VERB
	| ADVERB VERB
	| verb VERB
	;

object: NOUN
	| ADJECTIVE object
	;

prep_phrase: PREPOSITION NOUN
	;

%%

extern FILE *yyin;

int main(void)
{
	yyparse();
}

void yyerror(char *s)
{
	fprintf(stderr, "%s\n", s);
}
```

下面我们分别解释一下这两个文件。

## lex结合yacc

和上一篇使用lex的方式不同，这里我们结合使用了yacc和lex，我们注意lex文件的写法：

* `#include "y.tab.h"`：yacc在编译`.y`文件时，会对声明的TOKEN进行宏定义，其结果存储在`y.tab.h`中，lex文件需要引用这个头文件才能使用这些TOKEN宏定义。
* 注意每个匹配动作的return，返回值是一个TOKEN（的宏定义），这些TOKEN我们在`.y`文件中编写。

## yacc文件格式

yacc文件格式和lex很像，yacc设计时就为了让用户方便，设计成了和lex一样的格式。

首先`{%`和`%}`内放置了一些C语言代码，这和lex文件是一样的，我们在这里使用了`include`语句。

接下来的`%token`定义了一系列TOKEN。实际上，我们可以打开`y.tab.h`看看，这些定义的TOKEN会被yacc编译成宏定义的数字。

之后，`%%`表示当前区块结束，进入规则区块。规则区块中，我们发现，其实编写的内容和上文我们写的“文法产生式”基本是一样的，yacc其实就是这样使用的，我们定义一系列符合BNF的产生式，yacc能够生成能够自动推导的代码。这部分的写法，遵循上面的例子就好了，没什么多解释的。

最后，又出现了`%%`表示下一部分是用户代码区块，这和lex文件一样。我们在这一部分，编写了我们的main函数，main函数中调用了`yyparse()`，注意这个函数是yacc提供的不是lex的。总之，调用`yyparse()`，整个程序就能按照我们的预期，由lex进行词法分析并输出TOKEN序列，接着yacc按照我们定义的产生式进行推导。

注意`yyerror()`，这个函数是语法分析器遇到错误语法时的默认回调函数，我们实际上是覆盖了默认的`yyerror()`函数（默认的`yyerror()`通过宏进行定义，如果我们覆盖了yyerror，就会使用我们定义的yyerror）。

## 测试上面的程序

总之，我们现在编写好了我们的yycc和lex程序，我们测试一下。

编译：
```
lex english.l
yacc -d english.y
gcc lex.yy.c y.tab.c
```

先测试一个错的：
```
# ciyaz @ Linux in ~/workspace/mcc/yacc [20:41:05] C:1
$ ./a.out
adj noun verb adv adj noun
ADJECTIVE TOKEN
NOUN TOKEN
VERB TOKEN
ADVERB TOKEN

syntax error
```

注意：`syntax error`其实就是由yyerror输出的，语法分析器遇到错误语法，默认就会输出这条信息。

一个简单英文句子：
```
# ciyaz @ Linux in ~/workspace/mcc/yacc [20:42:10] C:1
$ ./a.out
adj noun adv verb adj noun
ADJECTIVE TOKEN
NOUN TOKEN
ADVERB TOKEN
VERB TOKEN
ADJECTIVE TOKEN
NOUN TOKEN

parsed simple sentence
```

一个复杂英文句子：
```
# ciyaz @ Linux in ~/workspace/mcc/yacc [20:43:31]
$ ./a.out
adj noun adv verb adj noun conj noun verb noun
ADJECTIVE TOKEN
NOUN TOKEN
ADVERB TOKEN
VERB TOKEN
ADJECTIVE TOKEN
NOUN TOKEN
CONJUCTION TOKEN
NOUN TOKEN
VERB TOKEN
NOUN TOKEN

parsed compound sentence
```

注意：Linux控制台下，请使用`ctrl+D`向标准输入发送EOF结束输入。

## 一些其他的yacc用法

上面的例子比较简单，覆盖的内容不是很全面，例如如何从lex向yacc传递识别的数据，如何定义TOKEN类型，如何构建语法树等。在后一篇笔记中，我们将介绍一个lex和yacc结合使用的另一个例子：类似SQL语法的解析。
