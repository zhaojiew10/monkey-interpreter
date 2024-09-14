
## 本书构建的解释器有哪些功能？
关于解释器的资料可以分为两类，1.重视理论的大部头，2.蜻蜓点水的介绍，这本书是介于这两者之间的

有些解释器很精简，只是单纯地解释输入，有些则进行了高层次优化，使用了高级解析和求值技术。有些不会求值而是将其转换为字节码等，甚至有JIT解释器将输入转换成机器码。本书的解释器介于上述之间，会解析源码，构建AST，然后求值。这类解释器名为tree-walking解释器

## chapter1中的lexer主要的作用？

词法分析器（Lexer）的作用是把输入的代码字符串分割成独立的单词（Token），比如数字、运算符、关键字、标识符等。

## 什么是普拉特解析？

普拉特解析（Pratt Parser）是一种用于解析算术表达式的算法，它使用一种称为“自顶向下”的递归下降算法。没有将解析函数和语法规则关联，而是将表达式的每个部分都看做是一个单独的语法单元，然后将语法单元和解析函数联系起来。

## chapter2中为什么能够区分类似x!=y和!-x，x-y和x--y这种表达？

x!=y和x-y，在构建token的时候就已经区分开了。而!-x和x--y，会根据上下文环境（优先级），将!和-x或者-和-y区分开。

```golang
// 这里如果curToken和nextToken是相同的，则必然会跳出循环，nextToken被解析为前缀表达式的符号
for !p.peekTokenIs(token.SEMICOLON) && precedence < p.peekPrecedence() {
    infix := p.infixParseFns[p.peekToken.Type]
    if infix == nil {
        return leftExp
    }

    p.nextToken() // 以x + x为例，此时currentToken为+

    leftExp = infix(leftExp) // 以x + x为例，leftExp为x，跳转到p.parseInfixExpression
}
```

## 示例中是如何使用普拉特解析法生成ast的？

见书章节2.7，解释的非常清楚

## 普拉特解析法中的优先级有什么作用？

让具备高优先级的表达式位于ast中更高的位置，而优先级越低的表达式位于ast中越低的位置。优先级代表了符号的约束能力，约束能力高的符号能够约束更多的表达式

## 如何实现"()"的解析？

左括号"("对应最高优先级的符号，在任何解析函数后都跳转到parseGroupedExpression，吸收所有右侧的表达式

```go
func (p *Parser) parseGroupedExpression() ast.Expression {
	p.nextToken()

	exp := p.parseExpression(LOWEST)

	if !p.expectPeek(token.RPAREN) {
		return nil
	}

	return exp
}
```

## 为什么if表达式的解析用的是block不是statement？

if词法单元后可能包括多个statement，因此需要类比program的构造方法