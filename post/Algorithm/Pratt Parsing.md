Pratt Parsing 是一种专门用来 parse 常见的算术表达式的算法，大多数编程语言的 Expression 都可以用这种算法手写 parser。

主要参考资料：
- [Blog](https://matklad.github.io/2020/04/13/simple-but-powerful-pratt-parsing.html)

# 核心思想

将 token 向高优先级的算符靠近，比如下面的算式：
```
1 + 2 * 3
```

我们给 `*` 赋予更高的优先级：
```
expr:   A    +    B    *    C
power:     3   3     5    5
```

我们赋予开头和结尾最低的power:
```
expr:    A   +   B   *   C
power: 0   3   3   5   5   0
```
对于 `B` 而言，右边的算符 power 更高，所以和右边结合。

我们想要令 `+` 和 `*` 都变成左结合，只需要让算符的右边 power 高于左边 power 即可 （右结合反过来）：
```
expr:    A   +     B   +     C
power: 0   3   3.1   3   3.1   0
```
这样一来，`B` 就会和左边的 `+` 结合。

算法的流程就是从左向右扫描，先找到一个 `term`，然后查看下一个中缀算符 `op`，对比 `op`的左 power `l_bp` 和当前环境的 power `min_bp`：
1. `l_bp < min_bp`：意味着 `term` 要和之前的结合，当前的环境结束。
2. `l_bp > min_bp`：意味着 `term` 要和 `op` 结合，继续向后 parse。

代码如下：
```rust
fn expr(input: &str) -> S {
    let mut lexer = Lexer::new(input);
    expr_bp(&mut lexer, 0) 
}

fn expr_bp(lexer: &mut Lexer, min_bp: u8) -> S { 
    let mut lhs = match lexer.next() {
        Token::Atom(it) => S::Atom(it),
        t => panic!("bad token: {:?}", t),
    };

    loop {
        let op = match lexer.peek() {
            Token::Eof => break,
            Token::Op(op) => op,
            t => panic!("bad token: {:?}", t),
        };

        let (l_bp, r_bp) = infix_binding_power(op);
        if l_bp < min_bp { 
            break;
        }

        lexer.next(); 
        let rhs = expr_bp(lexer, r_bp);

        lhs = S::Cons(op, vec![lhs, rhs]); 
    }

    lhs
}
```

更复杂度的算符也可以支持，比如前缀算符和后缀算符，只需要简单修改代码。这里不讨论了，详见 [Blog](https://matklad.github.io/2020/04/13/simple-but-powerful-pratt-parsing.html)

# 代码

```rust
use std::iter::Peekable;

#[derive(Debug, Clone, Copy, PartialEq, Eq)]
enum Token {
    Atom(char),
    Op(char),
}

#[derive(Debug, Clone)]
struct Lexer {
    tokens: Vec<Token>,
}

impl Lexer {
    fn new(input: &str) -> Self {
        let mut tokens = input
            .chars()
            .filter(|c| !c.is_ascii_whitespace())
            .map(|c| match c {
                '0'..='9' | 'a'..='z' | 'A'..='Z' => Token::Atom(c),
                _ => Token::Op(c),
            })
            .collect::<Vec<_>>();
        tokens.reverse();
        Lexer { tokens }
    }
}

impl Iterator for Lexer {
    type Item = Token;

    fn next(&mut self) -> Option<Self::Item> {
        self.tokens.pop()
    }
}

#[derive(Debug, Clone, PartialEq, Eq)]
pub enum S {
    Atom(char),
    Cons(char, Vec<S>),
}

impl std::fmt::Display for S {
    fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
        match self {
            S::Atom(c) => write!(f, "{}", c),
            S::Cons(op, args) => {
                write!(f, "({}", op)?;
                for arg in args {
                    write!(f, " {}", arg)?;
                }
                write!(f, ")")
            }
        }
    }
}

type ParseResult = Result<S, String>;

pub fn parse(input: &str) -> ParseResult {
    let mut stream = Lexer::new(input).peekable();
    let res = parse_bp(&mut stream, 0)?;
    if stream.peek().is_some() {
        Err("Unexpected tokens at the end of input".to_string())
    } else {
        Ok(res)
    }
}

fn parse_bp<I>(stream: &mut Peekable<I>, min_bp: u8) -> ParseResult
where
    I: Iterator<Item = Token>,
{
    let mut lhs = match stream
        .next()
        .ok_or_else(|| "Unexpected end of input".to_string())?
    {
        Token::Atom(c) => S::Atom(c),
        Token::Op('(') => {
            let lhs = parse_bp(stream, 0)?;
            if let Some(Token::Op(')')) = stream.next() {
                lhs
            } else {
                return Err("Expected closing parenthesis".to_string());
            }
        }
        Token::Op(op) => {
            let ((), rbp) = prefix_binding_power(op);
            let rhs = parse_bp(stream, rbp)?;
            S::Cons(op, vec![rhs])
        }
    };

    loop {
        let op = match stream.peek() {
            Some(Token::Op(op)) => *op,
            Some(t) => return Err(format!("Expected operator, but found: {:?}", t)),
            None => break,
        };

        if let Some((lbp, ())) = postfix_binding_power(op) {
            if lbp < min_bp {
                break;
            }
            stream.next(); // consume the operator

            lhs = if op == '[' {
                let rhs = parse_bp(stream, 0)?;
                if let Some(Token::Op(']')) = stream.next() {
                    S::Cons(op, vec![lhs, rhs])
                } else {
                    return Err("Expected closing bracket".to_string());
                }
            } else {
                S::Cons(op, vec![lhs])
            };
        } else if let Some((lbp, rbp)) = infix_binding_power(op) {
            if lbp < min_bp {
                break;
            }
            stream.next(); // consume the operator

            let rhs = parse_bp(stream, rbp)?;
            lhs = S::Cons(op, vec![lhs, rhs]);
        } else {
            break;
        }
    }

    Ok(lhs)
}

fn prefix_binding_power(op: char) -> ((), u8) {
    match op {
        '+' | '-' => ((), 9),
        _ => panic!("Unknown prefix operator: {}", op),
    }
}

fn infix_binding_power(op: char) -> Option<(u8, u8)> {
    match op {
        '=' => Some((2, 1)),
        '+' | '-' => Some((5, 6)),
        '*' | '/' => Some((7, 8)),
        '^' => Some((10, 9)),
        '.' => Some((14, 13)),
        _ => None,
    }
}

fn postfix_binding_power(op: char) -> Option<(u8, ())> {
    match op {
        '!' => Some((11, ())),
        '[' => Some((11, ())),
        _ => None,
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_s_expr() {
        let s_expr = S::Cons('+', vec![S::Atom('1'), S::Atom('2')]);
        assert_eq!(format!("{}", s_expr), "(+ 1 2)");
    }

    #[test]
    fn test_parsing() {
        // left associative
        let input = "1 + 2 + 3";
        let parsed = parse(input).unwrap();
        assert_eq!(format!("{}", parsed), "(+ (+ 1 2) 3)");

        // precedence
        let input = "1 + 2 * 3";
        let parsed = parse(input).unwrap();
        assert_eq!(format!("{}", parsed), "(+ 1 (* 2 3))");

        let input = "1 + 2 * 3 / 4 - 5";
        let parsed = parse(input).unwrap();
        assert_eq!(format!("{}", parsed), "(- (+ 1 (/ (* 2 3) 4)) 5)");

        let input = "a = b = 1 + 2 + 3 * 4 / 5 + 6";
        let parsed = parse(input).unwrap();
        assert_eq!(
            format!("{}", parsed),
            "(= a (= b (+ (+ (+ 1 2) (/ (* 3 4) 5)) 6)))"
        );

        // right associative
        let input = "a ^ b ^ c";
        let parsed = parse(input).unwrap();
        assert_eq!(format!("{}", parsed), "(^ a (^ b c))");

        let input = "f . g . h";
        let parsed = parse(input).unwrap();
        assert_eq!(format!("{}", parsed), "(. f (. g h))");

        // postfix
        let input = "1 * 2!";
        let parsed = parse(input).unwrap();
        assert_eq!(format!("{}", parsed), "(* 1 (! 2))");

        let input = "1 + f[2 + 3 * 4] * 5";
        let parsed = parse(input).unwrap();
        assert_eq!(format!("{}", parsed), "(+ 1 (* ([ f (+ 2 (* 3 4))) 5))");

        // parentheses
        let input = "(1 + 2) * (3 - 4)";
        let parsed = parse(input).unwrap();
        assert_eq!(format!("{}", parsed), "(* (+ 1 2) (- 3 4))");

        let input = "(((0)))";
        let parsed = parse(input).unwrap();
        assert_eq!(format!("{}", parsed), "0");

        // prefix
        let input = "- - -1 + 2";
        let parsed = parse(input).unwrap();
        assert_eq!(format!("{}", parsed), "(+ (- (- (- 1))) 2)");
    }

    #[test]
    fn test_fail() {
        let input = "1 + 2 *";
        let parsed = parse(input);
        assert_eq!(
            parsed,
            ParseResult::Err("Unexpected end of input".to_string())
        );

        let input = "1 + (2 * 3";
        let parsed = parse(input);
        assert_eq!(
            parsed,
            ParseResult::Err("Expected closing parenthesis".to_string())
        );

        let input = "(1 + 2 * f[3)";
        let parsed = parse(input);
        assert_eq!(
            parsed,
            ParseResult::Err("Expected closing bracket".to_string())
        );

        let input = "1 + 2 * 3)";
        let parsed = parse(input);
        assert_eq!(
            parsed,
            ParseResult::Err("Unexpected tokens at the end of input".to_string())
        );

        let input = "(1 + 2) (";
        let parsed = parse(input);
        assert_eq!(
            parsed,
            ParseResult::Err("Unexpected tokens at the end of input".to_string())
        );
    }
}
```

