---
layout: post
title: "开发 Yearn Lang(1)：实现变量赋值语句和复合语句"
date: 2025-11-18 22:14:00 +0800
categories: [开发]
math: false
mermaid: false
pin: false
---

项目地址：[yearn-lang](https://github.com/CrispyXYZ/yearn-lang)

昨晚赶在熄灯前终于把这一版本的 Yearn lang 写好了，主要实现了赋值语句和复合语句，也包括变量符号表，还有些许 bug，下周再修。这工程量比想象中大多了，但看到代码能正常跑起来还是挺有成就感的qwq

## 设计文法

编写新功能，首先要决定的就是文法的扩展，列出修改情况如下：

```patch
+ program           : statement_list
+ compound_statement: '{' statement_list '}' 
+ statement_list    : statement (';' statement)*
+ statement         : compound_statement
+                   | assignment
+ assignment        : variable '=' expr
  expr              : term (('+' | '-') term)*
  term              : factor (('*' | '/') factor)*
  factor            : ('+' | '-') factor
                    | INTEGER
                    | '(' expr ')'
+                   | variable
+ variable          : IDENTIFIER
```

~~可以看出工程量有亿点点大啊qwq~~

## 新增 Token

接下来是根据文法分析所需 Token ，这一步很简单，直接列出来吧：

```patch
  {
      Null,
      Plus,
      Minus,
      Multiply,
      Divide,
+     Assign,
      Integer,
      LeftParen,
      RightParen,
+     LeftBrace,
+     RightBrace,
+     Semicolon,
+     Identifier,
      Eof,
  };
```

## 词法分析器

然后就到了词法分析器，这里着重展示下 `Identifier` 的解析。

如果词法分析器当前字符是字母，那么就进入解析 `Identifier` 的函数：

```cpp
if(isalpha(currentChar)) {
    return getIdToken();
}
```

目前实现的设计是，读取连续的数字与字母组合：

```cpp
Token Lexer::getIdToken() {
    std::stringstream ss;
    while(currentChar != '\0' && isalnum(currentChar)) {
        ss << currentChar;
        advance();
    }
    return Token{TokenType::Identifier, ss.str()};
}
```

## 语法分析器

接下来是语法分析器 ~~（也是最复杂的一部分qwq）~~ ，要依次对上面的文法进行解析，输出 AST 。

### 更新 AST

为了支持新的语法结构，需要定义几个新的 AST 节点类型。`Compound` 节点用来表示由多个语句组成的代码块，`Variable` 节点表示变量引用，`Assignment` 节点表示赋值语句：

```cpp
class Compound final : public ASTNode {
public:
    std::vector<NodePtr> children;
    explicit Compound(std::vector<NodePtr> nodes);
    int accept(ASTVisitor &visitor) override;
};

class Variable final : public ASTNode {
public:
    Token token;
    std::string name;
    explicit Variable(Token token);
    int accept(ASTVisitor &visitor) override;
};

class Assignment final : public ASTNode {
public:
    Token op;
    NodePtr left;
    NodePtr right;
    Assignment(Token op, NodePtr left, NodePtr right);
    int accept(ASTVisitor &visitor) override;
};
```

### `variable`

这个简单，只需要一个 `Identifier` 的 Token 即可：

```cpp
NodePtr Parser::variable() {
    NodePtr node = std::make_unique<Variable>(currentToken);
    consume(TokenType::Identifier);
    return node;
}
```

### `factor` 的 `variable` 部分

这个最简单，只需要在原有逻辑之后加一个判断即可：

```cpp
if(type == TokenType::Identifier) {
    return variable();
}
```

### `assignment`

这一个也是依次解析即可：

```cpp
NodePtr Parser::assignment() {
    NodePtr left = variable();
    Token const token = currentToken;
    consume(TokenType::Assign);
    NodePtr right = expr();
    return std::make_unique<Assignment>(token, std::move(left), std::move(right));
}
```

### `statement`

判断第一个 Token 的类型即可：

```cpp
NodePtr Parser::statement() {
    NodePtr node;
    if(currentToken.getType() == TokenType::LeftBrace) {
        node = compound_statement();
    } else if(currentToken.getType() == TokenType::Identifier) {
        node = assignment();
    } else {
        throw InterpreterError("Unexpected token type ", currentToken.getType(), " while parsing statement");
    }
    return node;
}
```

### `statement_list`

语句列表的解析需要注意分号的处理。这里采用了比较宽松的策略，允许语句之间用分号分隔，但在遇到右大括号或文件结束时自动结束：

```cpp
std::vector<NodePtr> Parser::statement_list() {
    std::vector<NodePtr> nodes;
    if(currentToken.getType() == TokenType::RightBrace) {
        return nodes;
    }

    nodes.push_back(statement());
    while(currentToken.getType() == TokenType::Semicolon) {
        consume(TokenType::Semicolon);
        if (currentToken.getType() == TokenType::RightBrace || currentToken.getType() == TokenType::Eof) {
            break;
        }
        nodes.push_back(statement());
    }

    return nodes;
}
```

### `compound_statement`

复合语句就是由花括号包裹的语句列表，解析起来相对直接：

```cpp
NodePtr Parser::compound_statement() {
    consume(TokenType::LeftBrace);
    std::vector<NodePtr> nodes = statement_list();
    consume(TokenType::RightBrace);
    return std::make_unique<Compound>(std::move(nodes));
}
```

### `program`

整个程序就是一个大的复合语句，这里为了简洁，省去了大括号：

```cpp
NodePtr Parser::parse() {
    return std::make_unique<Compound>(std::move(statement_list()));;
}
```

## 解释器

解释器部分也是个大坑，之前傻乎乎地跟着IDE提示给每个方法都加上了 `[[nodiscard]]` 和 `const`，结果现在加入符号表，编译错误满天飞，直接炸裂qwq。目前用 `std::unordered_map<std::string, int> symbols;` 实现了符号表，还没实现作用域，其他的部分就没什么好看的了
