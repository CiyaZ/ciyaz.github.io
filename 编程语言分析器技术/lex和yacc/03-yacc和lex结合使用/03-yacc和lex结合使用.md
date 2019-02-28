# yacc和lex结合使用

上一篇笔记中，我们已经体验了yacc和lex结合使用的方法，但是那个例子非常简单，涉及到的lex/yacc用法还不足以解决“编写一个最简单的编程语言”这种需求。本篇笔记我们再来观察一个例子：解析简单的SQL语句。

当然SQL语法还算是比较复杂的，而且关键字也比较多，我们这里只是使用一种类似SQL的简化版语法，和SQL不太一样的。这个例子来自于我们的数据库课程实验。

给出三条例子语句：
```
select [ESSN='01'](projection [ESSN,PNAME](WORKS_ON join PROJECT));

projection [BDATE] (select [DNAME='Research' & ENAME='John'] (EMPLOYEE join DEPARTMENT));

select [DNAME='Research' & ENAME='Mary'] (EMPLOYEE join DEPARTMENT);
```

我们使用中括号表示查询条件，小括号包围的是查询的子句。解析器部分，我们的要求不高，写一个能恰好分析上面三条语句的解析器，并打印查询计划树即可。

## 词法分析器的设计

我们需要解析的TOKEN
```
SELECT PROJECTION JOIN EQUAL AND CONDITIONL CONDITIONR SUBQUERYL SUBQUERYR COMMA QUOTE STMTEND STRVAL
```

上面分别是：select关键字，projection关键字，join关键字，等号，与（&）号，左中括号，右中括号，左小括号，右小括号，逗号，单引号，分号（代表语句结束），字符串值

### parser.l

```
%{
#include <stdio.h>
#include <string.h>
#include "y.tab.h"
#include "parsetree.h"
extern int yylex(void);
%}
%%
select {
	yylval.queryNode = mallocQueryNode();
	yylval.queryNode->type = SELECT;
	snprintf(yylval.queryNode->text, NODE_TEXT_SIZE, "SELECT");
	return SELECT;
}
projection {
	yylval.queryNode = mallocQueryNode();
	yylval.queryNode->type = PROJECTION;
	snprintf(yylval.queryNode->text, NODE_TEXT_SIZE, "PROJECTION");
	return PROJECTION;
}
join {
	yylval.queryNode = mallocQueryNode();
	yylval.queryNode->type = JOIN;
	snprintf(yylval.queryNode->text, NODE_TEXT_SIZE, "JOIN");
	return JOIN;
}
= {
	return EQUAL;
}
& {
	return AND;
}
\[ {
	return CONDITIONL;
}
\] {
	return CONDITIONR;
}
\( {
	return SUBQUERYL;
}
\) {
	return SUBQUERYR;
}
, {
	return COMMA;
}
' {
	return QUOTE;
}
; {
	return STMTEND;
}
[a-zA-Z0-9_]+ {
	char *stringBuffer = (char *) malloc(sizeof(char) * STRBUFFER_SIZE);
	if(stringBuffer == NULL)
	{
		perror("projection_vars:projection_var malloc()");
		exit(EXIT_FAILURE);
	}
	strcpy(stringBuffer, yytext);
	yylval.strVal = stringBuffer;
	return STRVAL;
}
[ \t]+ {}
%%
int yywrap(void)
{
	return 1;
}
```

注意， 头文件我们引用了`y.tab.h`，这个头文件是yacc生成的，里面包含TOKEN的宏定义，想要结合yacc，就必须引用这个头文件。

yylval是用来向yacc传值的。yylval其实是联合体类型，这里我们不多说，看完yacc的代码，再回来看yylval就能理解了。传值时，一定要注意字符串的传递方式，字符串最好在堆上分配，然后从yytext拷贝一次，否则会发生意想不到的情况。而传递结构体类型，也是直接malloc的（mallocQueryNode函数就会在堆上分配）。

注意每个正则表达式的匹配动作，我们直接返回了一个TOKEN，其实这些TOKEN是int类型的，他们由yacc生成了宏定义，定义在`y.tab.h`中。

## 语法分析器的生成式设计

接下来我们设计解析的语法的文法生成式，仅仅满足上面三条语句的话，这个真的很简单。我想不需要多说了。

```
stmt->select_stmt STMTEND|projection_stmt STMTEND|join_stmt STMTEND

subquery_stmt->select_stmt|projection_stmt|join_stmt

select_stmt->SELECT CONDITIONL exprs CONDITIONR SUBQUERYL subquery_stmt SUBQUERYR

projection_stmt->PROJECTION CONDITIONL projection_vars CONDITIONR SUBQUERYL subquery_stmt SUBQUERYR

join_stmt->STRVAL JOIN STRVAL

exprs->expr|exprs AND expr

expr->STRVAL EQUAL QUOTE STRVAL QUOTE

projection_vars->projection_var|projection_vars COMMA projection_var

projection_var->STRVAL
```

### parser.y

下面我们根据上面的文法生成式，编写yacc文件。

```
%{
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include "parsetree.h"
#include "expr.h"
#include "optimiser.h"
void yyerror(char *s);
%}

%union {
	struct QueryNode *queryNode;
	struct Expr *exprNode;
	char *strVal;
}

%token SELECT PROJECTION JOIN EQUAL AND CONDITIONL CONDITIONR SUBQUERYL SUBQUERYR COMMA QUOTE STMTEND STRVAL

%type<queryNode> SELECT PROJECTION JOIN
%type<strVal> STRVAL

%type<queryNode> stmt subquery_stmt select_stmt projection_stmt join_stmt
%type<strVal> expr projection_vars projection_var
%type<exprNode> exprs
%%
stmt: select_stmt STMTEND {
		printf("Parser:select stmt identified\n");
		$$ = $1;
		dumpTree($$);
		$$ = optimise($$);
		dumpTree($$);
	}
	| projection_stmt STMTEND {
		printf("Parser:projection stmt identified\n");
		$$ = $1;
		dumpTree($$);
		$$ = optimise($$);
		dumpTree($$);
	}
	| join_stmt STMTEND {
		printf("Parser:join stmt identified\n");
		$$ = $1;
		dumpTree($$);
		$$ = optimise($$);
		dumpTree($$);
	}
	;

subquery_stmt: select_stmt {
		$$ = $1;
	}
	| projection_stmt {
		$$ = $1;
	}
	| join_stmt {
		$$ = $1;
	}
	;

select_stmt: SELECT CONDITIONL exprs CONDITIONR SUBQUERYL subquery_stmt SUBQUERYR {

		struct Expr *p = $3;
		struct QueryNode *root = $6;
		while(p != NULL)
		{
			struct QueryNode *newSelectNode = mallocQueryNode();
			newSelectNode->type = SELECT;
			strcpy(newSelectNode->text, "SELECT ");
			strcat(newSelectNode->text, p->text);

			appendChild(newSelectNode, root);
			root = newSelectNode;

			p = p->next;
		}

		$$ = root;
	}
	;

projection_stmt: PROJECTION CONDITIONL projection_vars CONDITIONR SUBQUERYL subquery_stmt SUBQUERYR {
		strcat($1->text, " ");
		strcat($1->text, $3);

		appendChild($1, $6);

		$$ = $1;
	}
	;

join_stmt: STRVAL JOIN STRVAL {
		struct QueryNode *tbChild1 = mallocQueryNode();
		tbChild1->type = STRVAL;
		strcpy(tbChild1->text, $1);

		struct QueryNode *tbChild2 = mallocQueryNode();
		tbChild2->type = STRVAL;
		strcpy(tbChild2->text, $3);

		appendChild($2, tbChild1);
		appendChild($2, tbChild2);

		$$ = $2;
	}
	;

exprs: expr {
		struct Expr *exprNode = mallocNewExpr();
		strcpy(exprNode->text, $1);

		$$ = exprNode;
	}
	|exprs AND expr {
		struct Expr *exprNode = mallocNewExpr();
		strcpy(exprNode->text, $3);
		appendExprList($1, exprNode);

		$$ = $1;
	}
	;

expr: STRVAL EQUAL QUOTE STRVAL QUOTE {
		char *stringBuffer = (char *) malloc(sizeof(char) * STRBUFFER_SIZE);
		if(stringBuffer == NULL)
		{
			perror("projection_vars:projection_var malloc()");
			exit(EXIT_FAILURE);
		}
		snprintf(stringBuffer, STRBUFFER_SIZE, "%s='%s'", $1, $4);
		$$ = stringBuffer;
	}
	;

projection_vars: projection_var {
		char *stringBuffer = (char *) malloc(sizeof(char) * STRBUFFER_SIZE);
		if(stringBuffer == NULL)
		{
			perror("projection_vars:projection_var malloc()");
			exit(EXIT_FAILURE);
		}
		strcpy(stringBuffer, $1);
		$$ = stringBuffer;
	}
	| projection_vars COMMA projection_var {
		char *stringBuffer = (char *) malloc(sizeof(char) * STRBUFFER_SIZE);
		if(stringBuffer == NULL)
		{
			perror("projection_vars:projection_vars COMMA projection_var malloc()");
			exit(EXIT_FAILURE);
		}
		snprintf(stringBuffer, STRBUFFER_SIZE, "%s,%s" , $1, $3);
		$$ = stringBuffer;
	}
	;

projection_var: STRVAL {
		char *stringBuffer = (char *) malloc(sizeof(char) * STRBUFFER_SIZE);
		if(stringBuffer == NULL)
		{
			perror("projection_var:STRVAL malloc()");
			exit(EXIT_FAILURE);
		}
		strcpy(stringBuffer, $1);
		$$ = stringBuffer;
	}
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

不要在意那些头文件，它们真的并不复杂，这里我们只需要关注yacc的用法就行了。

`%union`是一个联合体，这个联合体其实就是yylval，我们使用这个联合体从lex向yacc传递值。

`%type`这个是定义了一系列终结符和非终结符的数据类型，注意这里数据类型其实是指`%union`中定义过的变量名，而不是`int struct node*`什么的。

`$$`，`$1`，`$2`...这些在文法生成式的动作代码中，用来指代生成式中的符号。`$$`指生成式左部，`$1`等指右部各个符号，注意，即使某个TOKEN没有返回值，它也占用一个`$n`符号。我们的动作代码，基本就是构建逻辑查询计划树了。

其实我没学过编译原理，上面的写法可能不是很专业，但是通过这次学习，我确实感觉到`lex`+`yacc`的强大，打印生成的查询计划树的代码，我甚至编译过后没调试，就能运行成功了，很有成就感。要是像以前解析字符串的写法，不知要写到什么时候，还可能一大堆bug。

到现在为止，我们基本掌握了lex和yacc的结合使用，还有一些lex/yacc的高级用法没有介绍，不过我们现在掌握的知识已经足够可以开始自由发挥了：）
