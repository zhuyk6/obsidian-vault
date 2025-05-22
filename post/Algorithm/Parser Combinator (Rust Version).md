# 定义

下面先给出基本的定义：

```rust
pub type ParseResult<'a, O> = Result<(&'a str, O), String>;

pub trait Parser<'a, O> {
    fn parse(&self, input: &'a str) -> ParseResult<'a, O>;

    fn and_then<O2, P2, F>(self, f: F) -> impl Parser<'a, O2>
    where
        Self: Sized,
        P2: Parser<'a, O2>,
        F: Fn(O) -> P2,
    {
        move |input1: &'a str| {
            let (input2, output1) = self.parse(input1)?;
            let parser = f(output1);
            parser.parse(input2)
        }
    }

    fn and<O2, P2>(self, p2: P2) -> impl Parser<'a, O2>
    where
        Self: Sized,
        P2: Parser<'a, O2>,
    {
        move |input1: &'a str| {
            let (input2, _) = self.parse(input1)?;
            let (input3, output2) = p2.parse(input2)?;
            Ok((input3, output2))
        }
    }

    fn pair<O2, P2>(self, p2: P2) -> impl Parser<'a, (O, O2)>
    where
        Self: Sized,
        P2: Parser<'a, O2>,
    {
        move |input1: &'a str| {
            let (input2, output1) = self.parse(input1)?;
            let (input3, output2) = p2.parse(input2)?;
            Ok((input3, (output1, output2)))
        }
    }
}

impl<'a, F, O> Parser<'a, O> for F
where
    F: Fn(&'a str) -> ParseResult<'a, O>,
{
    fn parse(&self, input: &'a str) -> ParseResult<'a, O> {
        self(input)
    }
}
```

下面分析一下代码并解释：

所谓 parse 就是判断 `&'a str` 能够得到结果 `O`，所以 `ParseResult<'a, O> = Result<(&'a str, O), String>`，也就是说我们考虑的是简单的情景：
- 如果 parse 成功，消耗字符，得到结果 `Ok((new_str, output))`；
- 如果失败，得到 `Err`

需要注意的是，由于 parser combinator 可能会多次调用同一个 parser 的 `parse`，所以我们要求 `parse` 的函数签名调用的是 `&self`。也正是这个原因，我们只能够对于 `F: Fn(&'a str) -> ParseResult<'a, O>` 实现 `Parser` trait，而不是 `FnOnce`，因为 `FnOnce` 无法多次调用，一次 `parse` 就消耗掉了。

注意到，因为我们 `ParseResult` 是一个 `Result`，所以可以使用 `Result` 的 monadic API，比如 `and`、`and_then` 等。更重要的是，结合 Rust 的 `?` 语法糖，我们完全可以实现类似 Haskell 的 `do` 语法糖，比如：
```rust
fn and_then<O2, P2, F>(self, f: F) -> impl Parser<'a, O2>
where
	Self: Sized,
	P2: Parser<'a, O2>,
	F: Fn(O) -> P2,
{
	move |input1: &'a str| {
		let (input2, output1) = self.parse(input1)?;
		let parser = f(output1);
		parser.parse(input2)
	}
}
```

需要注意的是，你可能想要复用 `and_then` 来实现 `and`，比如下面这样：
```rust
fn and<O2, P2>(self, p2: P2) -> impl Parser<'a, O2>
where
	Self: Sized,
	P2: Parser<'a, O2>,
{
	move |input1: &'a str| {
		let (input2, _) = self.parse(input1)?;
		let (input3, output2) = p2.parse(input2)?;
		Ok((input3, output2))
	}
}

fn wrong_and<O2, P2>(self, p2: P2) -> impl Parser<'a, O>
where
	Self: Sized,
	P2: Parser<'a, O>,
{
	self.and_then(|_| p2)
}
```
如果是在 Haskell 中，这样显然没问题，但这是 Rust，你需要仔细思考所有权。下面的实现报错：
```
cannot move out of `p2`, a captured variable in an `Fn` closure  
move occurs because `p2` has type `P2`, which does not implement the `Copy` traitrustc[Click for full compiler diagnostic](rust-analyzer-diagnostics-view:/diagnostic%20message%20[0]?0#file:///home/zhuyk6/Documents/parsec-rs/src/parser.rs)

parser.rs(31, 32): captured outer variable

parser.rs(36, 23): captured by this `Fn` closure

parser.rs(31, 22): if `P2` implemented `Clone`, you could clone the value
```
可以看到，不允许我们将 `p2` move 到闭包中，因为 `and_then` 需要传入 `Fn` 闭包，但是 `|_| p2` 的类型是 `FnOnce`，除非我们引入额外的要求 `P2: Clone`。那么能否放宽要求 `and_then` 接受 `FnOnce` 呢？也不行，这样一来 `and_then` 返回类型就是 `FnOnce`，我们上文中提到的，由于 `parse(&self...)` 所以 `FnOnce` 不是 `Parser`。

那么为什么 `and` 的实现是对的呢？因为我们把 `p2` move 到闭包中后，闭包只是用了 `p2` 的引用，因此可以重复使用，这个闭包的类型是 `Fn`。

# 报错信息

为了更好的报错，我们需要引入更多的信息：
- 错误发生在什么地方？第几行第几列（第几个字符）
- 报错的地方遇到的字符是什么？
- 期望的内容是什么？

因此，我们引入一些辅助的类型：
```rust
type Position = usize;

/// The state of the input. It contains the left input string and the current position.
#[derive(Debug, Clone, PartialEq, Eq, Copy)]
pub struct State<'a> {
    pub input: &'a str,
    pub pos: Position,
}

/// Error message for the parser.
#[derive(Debug, Clone, PartialEq, Eq)]
pub struct Message {
    /// The position of the error in the input string.
    pub pos: Position,
    /// The unexpected input string.
    pub unexpected: String,
    /// The expected input string.
    pub expected: Vec<String>,
}

impl Message {
    /// Merge two error messages.
    fn merge(msg1: Message, msg2: Message) -> Message {
        Message {
            pos: msg2.pos,
            unexpected: msg2.unexpected,
            expected: msg1.expected.into_iter().chain(msg2.expected).collect(),
        }
    }
}
```

同时修改 `Parser` trait 的定义：
```rust

/// The result of a parser.
/// If the parser is successful, it returns the output and the new state.
/// If the parser fails, it returns an error message.
pub type ParseResult<'a, O> = Result<(O, State<'a>), Message>;

/// The trait for parsers.
pub trait Parser<'a, O> {
    /// Parses the input and returns the result.
    fn parse(&self, state: State<'a>) -> ParseResult<'a, O>;

    /// Runs the first parser and runs the second parser if success according to the function and the result.
    fn and_then<O2, P2, F>(self, f: F) -> impl Parser<'a, O2>
    where
        Self: Sized,
        P2: Parser<'a, O2>,
        F: Fn(O) -> P2,
    {
        move |state: State<'a>| {
            let (output1, state) = self.parse(state)?;
            let parser = f(output1);
            parser.parse(state)
        }
    }

    /// Runs the first parser and then the second parser if success.
    fn and<O2, P2>(self, p2: P2) -> impl Parser<'a, O2>
    where
        Self: Sized,
        P2: Parser<'a, O2>,
    {
        move |s1| {
            let (_, s2) = self.parse(s1)?;
            p2.parse(s2)
        }
    }

    /// Combines two parsers.
    /// Only if the first parser fails and does not consume any input, the second parser is tried.
    fn or<P2>(self, p2: P2) -> impl Parser<'a, O>
    where
        Self: Sized,
        P2: Parser<'a, O>,
    {
        move |s @ State { pos: pos0, .. }| match self.parse(s) {
            Err(msg1 @ Message { pos: pos1, .. }) if pos0 == pos1 => match p2.parse(s) {
                Err(msg2) => Err(Message::merge(msg1, msg2)),
                Ok(result) => Ok(result),
            },
            res => res,
        }
    }

    /// Combines two parsers.
    /// If the first parser fails, the second parser is tried.
    fn or1<P2>(self, p2: P2) -> impl Parser<'a, O>
    where
        Self: Sized,
        P2: Parser<'a, O>,
    {
        move |s| match self.parse(s) {
            Err(msg1) => match p2.parse(s) {
                Err(msg2) => Err(Message::merge(msg1, msg2)),
                Ok(result) => Ok(result),
            },
            res => res,
        }
    }

    /// Runs the first parser and then the second parser.
    /// Returns the pair of the results.
    fn pair<O2, P2>(self, p2: P2) -> impl Parser<'a, (O, O2)>
    where
        Self: Sized,
        P2: Parser<'a, O2>,
    {
        move |s1| {
            let (output1, s2) = self.parse(s1)?;
            let (output2, s3) = p2.parse(s2)?;
            Ok(((output1, output2), s3))
        }
    }

    /// Runs the parser and returns the result after applying the function.
    fn map<O2, F>(self, map_fn: F) -> impl Parser<'a, O2>
    where
        Self: Sized,
        F: Fn(O) -> O2,
    {
        move |s| {
            let (output, s) = self.parse(s)?;
            Ok((map_fn(output), s))
        }
    }

    fn map_err<F>(self, f: F) -> impl Parser<'a, O>
    where
        Self: Sized,
        F: Fn(Message) -> Message,
    {
        move |s| match self.parse(s) {
            Ok(result) => Ok(result),
            Err(msg) => Err(f(msg)),
        }
    }

    /// Labels the parser with a string.
    /// This is used to provide better error messages.
    fn label(self, info: impl ToString) -> impl Parser<'a, O>
    where
        Self: Sized,
    {
        move |s| match self.parse(s) {
            Ok(result) => Ok(result),
            Err(msg) => Err(Message {
                expected: vec![info.to_string()],
                ..msg
            }),
        }
    }

    /// Labels the parser with a vector of strings.
    fn labels<V, S>(self, info: V) -> impl Parser<'a, O>
    where
        Self: Sized,
        S: ToString,
        V: AsRef<[S]>,
    {
        move |s| match self.parse(s) {
            Ok(result) => Ok(result),
            Err(msg) => Err(Message {
                expected: info.as_ref().iter().map(|s| s.to_string()).collect(),
                ..msg
            }),
        }
    }

    /// Runs the parser and then parses spaces.
    fn lexeme(self) -> impl Parser<'a, O>
    where
        Self: Sized,
    {
        move |s| {
            let (result, s) = self.parse(s)?;
            let (_, s) = spaces(s)?;
            Ok((result, s))
        }
    }
}
```

除了必须提供的 `parse` method，默认提供了 monadic API，这里重点介绍几个：
- `or`：如果我们想要实现的 parser 是 LL(1) 的文法，那么当且仅当「第一个 parser 失败且未消耗 input」，我们执行第二个 parser，并合并错误（如果第二个也失败）。
	这对于LL(1)文法是足够的，因为 LL(1) 文法每一个产生式的 first 集合不同，这样高效率；
	如果不想要这么严格的快速失败，那么可以使用 `or1`，它的实现是「第一个parser失败就执行第二个」。
- `label`：用于提供 parser 的信息，报错时的 expected 就是这些信息，`labels` 也是类似的；
- `lexeme`：是一种 parse 模式，用于把一个普通的 parser 变成「顺便把后续紧跟着的“空白”消耗掉」的parser，如果没有预先生成 tokens，那么这种模式能够简单的消耗无谓的空白；

然后当然需要提供一个默认的实现 `Parser` trait 的类型，也就是函数类型（不一定是函数，也可以是闭包等）：
```rust
impl<'a, F, O> Parser<'a, O> for F
where
    F: Fn(State<'a>) -> ParseResult<'a, O>,
{
    fn parse(&self, state: State<'a>) -> ParseResult<'a, O> {
        self(state)
    }
}
```

当然我们需要提供一个最外层的调用函数：
```rust
/// Runs the parser on the input string.
pub fn run_parser<'a, O, P>(parser: P, input: &'a str) -> ParseResult<'a, O>
where
    P: Parser<'a, O>,
{
    let state = State { input, pos: 0 };
    spaces.and(parser).parse(state)
}
```
这里使用 `lexeme` parse，需要预先 parse 消耗掉开头的空白。

# 几个简单的 `Parser` 实例

`pure` 函数，也就是 Monad 中的 pure，这里因为 Rust 的限制，我们需要 `O: Clone`，其实这个函数并没有那么有用，后面会看到，在 Rust 中并不像 Haskell 那样很依赖它，需要的时候手动返回 `Ok` 就好了。
```rust
/// Just returns the output.
pub fn pure<'a, O: Clone>(value: O) -> impl Parser<'a, O> {
    move |state| Ok((value.clone(), state))
}
```

第一个真正有用的 parser，接受一个函数，parse 一个字符并用该函数判断是否成功。看上去很长，其实是在处理错误信息。
```rust
/// Parse a character that satisfies a given predicate.
pub fn satisfy<'a, F>(f: F) -> impl Parser<'a, char>
where
    F: Fn(char) -> bool,
{
    move |State { input, pos }| match input.chars().next() {
        Some(ch) => {
            if f(ch) {
                let len = ch.len_utf8();
                let new_pos = pos + len;
                Ok((
                    ch,
                    State {
                        input: &input[len..],
                        pos: new_pos,
                    },
                ))
            } else {
                Err(Message {
                    pos,
                    unexpected: ch.to_string(),
                    expected: vec![],
                })
            }
        }
        None => Err(Message {
            pos,
            unexpected: "end of input".to_string(),
            expected: vec![],
        }),
    }
}
```

有了 `satisfy`，立马可以实现许多有用的 parser：
```rust
/// Parse a given character.
pub fn character<'a>(c: char) -> impl Parser<'a, char> {
    satisfy(move |ch| ch == c).label(c)
}

/// Parse a digit character.
pub fn digit(state: State) -> ParseResult<char> {
    satisfy(|ch| ch.is_ascii_digit())
        .label("digit")
        .parse(state)
}

/// Parse a letter character.
pub fn letter(state: State) -> ParseResult<char> {
    satisfy(|ch| ch.is_ascii_alphabetic())
        .label("letter")
        .parse(state)
}

/// Parse a given character that is in the specified set.
pub fn one_of(chars: &[char]) -> impl Parser<char> {
    satisfy(|ch| chars.contains(&ch)).labels(chars)
}
```

然后我们也可以实现 parse 一个字符串 `string`：
```rust
/// Parse a given string.
pub fn string<'a, 'b: 'a>(s: &'b str) -> impl Parser<'a, ()> {
    move |State { input, pos }| {
        input.strip_prefix(s).map_or_else(
            || {
                Err(Message {
                    pos,
                    unexpected: input.to_string(),
                    expected: vec![s.to_string()],
                })
            },
            |next| {
                let len = s.len();
                let new_pos = pos + len;
                Ok((
                    (),
                    State {
                        input: next,
                        pos: new_pos,
                    },
                ))
            },
        )
    }
}
```
需要注意的是，和 Haskell 的实现不同，这里并没有使用一个一个字符的 parse，因此失败时，并不会消耗输入。

# Parser Combinator

为什么我们叫作 parser combinator，就是因为我们可以方便的定义 parser 的高阶函数，下面介绍几个最常用最重要的 combinator。

重复 parse 一个给定的 parser：可以发现，我们并没有使用 Haskell 那样递归的方式，而且由于所有权的复杂性，我们也无法递归的实现，但是我们可以通过循环替代递归。后面会发现，通过直接书写逻辑（循环等）来替换递归是 Rust 中实现的重要手段，想要使用「递归」往往需要添加更多的约束（`Clone` trait）反而会很麻烦且低效。
```rust
/// Parse a given parser zero or more times.
pub fn zero_or_more<'a, O, P>(parser: P) -> impl Parser<'a, Vec<O>>
where
    P: Parser<'a, O>,
{
    move |state| {
        let mut results = Vec::new();
        let mut current_state = state;

        while let Ok((result, next_state)) = parser.parse(current_state) {
            results.push(result);
            current_state = next_state;
        }

        Ok((results, current_state))
    }
}

/// Parse a given parser one or more times.
pub fn one_or_more<'a, O, P>(parser: P) -> impl Parser<'a, Vec<O>>
where
    P: Parser<'a, O>,
{
    move |state| {
        let (first_result, mut current_state) = parser.parse(state)?;
        let mut results = vec![first_result];

        while let Ok((result, next_state)) = parser.parse(current_state) {
            results.push(result);
            current_state = next_state;
        }

        Ok((results, current_state))
    }
}
```

对应 Haskell 中的 `try` 函数，这是由于我们实现的 `or` 是 LL(1) 的，当失败后想要「回溯」而不消耗符号，就可以使用这个 `try_parser`。但其实使用 Rust 的写法这个函数并不常用。
```rust
/// Wraps a parser. If the parser fails, it will not consume any input.
pub fn try_parser<'a, O, P>(parser: P) -> impl Parser<'a, O>
where
    P: Parser<'a, O>,
{
    move |state @ State { pos, .. }| match parser.parse(state) {
        Ok(result) => Ok(result),
        Err(_) => Err(Message {
            pos,
            unexpected: String::new(),
            expected: vec![],
        }),
    }
}
```

括号等结构：
```rust
/// Parse a given parser with parentheses.
pub fn parens<'a, O, P>(parser: P, open: char, close: char) -> impl Parser<'a, O>
where
    P: Parser<'a, O>,
{
    move |state| {
        let (_, state) = character(open).parse(state)?;
        let (inner_result, state) = parser.parse(state)?;
        let (_, state) = character(close).parse(state)?;
        Ok((inner_result, state))
    }
}
```

# 例子

## 括号匹配

首先来看一个非常简单的例子：
> 字符串仅有左右括号`'(', ')'` ，判断是否所有的括号配对。

这个例子的文法如下：
```
S -> epsilon
S -> (S)
S -> SS
```
当然这不是 LL(1) 的，因为有左递归 `S -> SS`，我们消除后的文法如下：
```
S -> epsilon
S -> (S)
S -> (S)S
```

直接用我们实现的 parser 实现如下：
```rust
pub fn parens_parser(state: State) -> ParseResult<()> {
    parens(parens_parser, '(', ')')
        .and(parens_parser)
        .or(pure(()))
        .parse(state)
}
```

需要注意的是，这个 parser 永远不会失败，因为 `or(pure(()))`，所以需要在最外层判断是否消耗完所有的字符串。比如 `() )(` 它就会剩下 `)(` 不去 parse。

这里虽然可以使用递归（因为函数是 `Copy` 的），但是我们依然可以用循环干掉递归。考虑它本质上是若干个（可能是0个）`S`，我们可以改写文法：
```
S -> epsilon
S -> (S)
S -> (S)(S)
S -> ...
```

循环写法：
```rust
pub fn parens_p(mut state: State) -> ParseResult<()> {
    while let Ok(((), new_state)) = parens(parens_p, '(', ')').parse(state) {
        state = new_state;
    }
    Ok(((), state))
}
```
其本质上也是使用一个栈来记录共有多少个 `'('`，只不过这个栈是系统栈。

另外它也存在一样的问题，它不会失败，最后可能会剩下未匹配的部分。

## 算术表达式

详细内容参照我的[Blog](https://zhuyk6.github.io/2021/07/20/Parse-an-Expression/) 

简单来说，最初的文法：
```
E -> Number
E -> E + E | E - E | E * E | E / E
E -> (E)
```

去掉左递归：
```
E -> Term + E | Term
Term -> Number | (E)
```

但是这并不是 LL(1)，因为 `Term + E` 和 `Term` 的 first 重合，需要 `try`。而且结合性是「右结合」。
一个改进的方法是，使用 类似于
```haskell
sum :: [Int] -> Int
sum xs = run 0 xs
	where
		run acc [] = acc
		run acc (x:xs) = run (acc + x) xs
```
的方法，也就是用一个 accumulator 来实现尾递归。

用 Haskell 来表示代码：
```Haskell
term = num <|> parens expr  
  
expr :: Parser Expr  
expr = do  
    e <- term  
    maybeAddSuffix e  
  where  
    addSuffix e1 = do  
        symbol '+'  
        e2 <- term  
        maybeAddSuffix (Add e1 e2)  
    maybeAddSuffix e = addSuffix e <|> return e
```

但是这种方法用 Rust 实现比较麻烦，因为需要 `clone expr`，所以使用循环干掉递归，毕竟尾递归就是循环：
```rust
fn chainl1<'a, P1, P2>(term_parser: P1, op_parser: P2) -> impl Parser<'a, Expr>
where
    P1: Parser<'a, Expr> + Clone,
    P2: Parser<'a, BoxOperator2> + Clone,
{
    move |state| {
        let (mut lhs, mut state) = term_parser.parse(state)?;

        while let Ok((op, new_state)) = op_parser.parse(state) {
            let (rhs, new_state) = term_parser.parse(new_state)?;
            lhs = op(Box::new(lhs), Box::new(rhs));
            state = new_state;
        }

        Ok((lhs, state))
    }
}
```

关于运算的优先级，其实就是外层进行「低优先级」，内层进行「高优先级」：
```rust
pub fn expr(state: State) -> ParseResult<Expr> {
    chainl1(term, add_op).parse(state)
}

fn term(state: State) -> ParseResult<Expr> {
    chainl1(factor, mul_op).parse(state)
}

fn factor(state: State) -> ParseResult<Expr> {
    number.or(parens(expr, '(', ')')).parse(state)
}
```

