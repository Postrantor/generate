---
tip: translate by baidu@2023-10-25 08:30:31
---
---

metaTitle: "C++ | Regular expressions"
description: "Basic regex_match and regex_search Examples, regex_iterator Example, Anchors, regex_replace Example, regex_token_iterator Example, Splitting a string, Quantifiers"
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Regular expressions

[Regular Expressions](https://en.wikipedia.org/wiki/Regular_expression) (sometimes called regexs or regexps) are a textual syntax which represents the patterns which can be matched in the strings operated upon.

> [正则表达式](https://en.wikipedia.org/wiki/Regular_expression)（有时称为 regexs 或 regexp）是一种文本语法，表示可以在操作的字符串中匹配的模式。

Regular Expressions, introduced in [c++11](https://stackoverflow.com/questions/tagged/c%2B%2B11), may optionally support a return array of matched strings or another textual syntax defining how to replace matched patterns in strings operated upon.

> 正则表达式，在[c++11]中引入([https://stackoverflow.com/questions/tagged/c%2b%2b11](https://stackoverflow.com/questions/tagged/c%2B%2B11))，可以可选地支持匹配字符串的返回数组或定义如何替换操作的字符串中的匹配模式的另一文本语法。

## Basic regex_match and regex_search Examples

```cpp
const auto input = "Some people, when confronted with a problem, think \"I know, I'll use regular expressions.\""s;
smatch sm;

cout << input << endl;

// If input ends in a quotation that contains a word that begins with "reg" and another word begining with "ex" then capture the preceeding portion of input
if (regex_match(input, sm, regex("(.*)\".*\\breg.*\\bex.*\"\\s*$"))) {
    const auto capture = sm[1].str();
    
    cout << '\t' << capture << endl; // Outputs: "\tSome people, when confronted with a problem, think\n"
    
    // Search our capture for "a problem" or "# problems"
    if(regex_search(capture, sm, regex("(a|d+)\\s+problems?"))) {
        const auto count = sm[1] == "a"s ? 1 : stoi(sm[1]);
        
        cout << '\t' << count << (count > 1 ? " problems\n" : " problem\n"); // Outputs: "\t1 problem\n"
        cout << "Now they have " << count + 1 << " problems.\n"; // Ouputs: "Now they have 2 problems\n"
    }
}

```

[Live Example](http://ideone.com/nSRXEa)

## regex_iterator Example

When processing of captures has to be done iteratively a `regex_iterator` is a good choice. Dereferencing a `regex_iterator` returns a `match_result`. This is great for conditional captures or captures which have interdependence. Let's say that we want to tokenize some C++ code. Given:

> 当必须迭代地处理捕获时，“regex_iterator”是一个不错的选择。取消引用“regex_iterator”将返回“match_result”。这对于条件捕获或具有相互依赖性的捕获非常有用。比方说，我们想将一些 C++ 代码标记化。给定：

```cpp
enum TOKENS {
    NUMBER,
    ADDITION,
    SUBTRACTION,
    MULTIPLICATION,
    DIVISION,
    EQUALITY,
    OPEN_PARENTHESIS,
    CLOSE_PARENTHESIS
};

```

We can tokenize this string: `const auto input = "42/2 + -8\t=\n(2 + 2) * 2 * 2 -3"s` with a `regex_iterator` like this:

> 我们可以用类似的 regex_iterator 标记这个字符串：`const auto-input=“42/2+-8\t=\n（2+2）*2*2-3”s`：

```cpp
vector<TOKENS> tokens;
const regex re{ "\\s*(\\(?)\\s*(-?\\s*\\d+)\\s*(\\)?)\\s*(?:(\\+)|(-)|(\\*)|(/)|(=))" };

for_each(sregex_iterator(cbegin(input), cend(input), re), sregex_iterator(), [&](const auto& i) {
    if(i[1].length() > 0) {
        tokens.push_back(OPEN_PARENTHESIS);
    }
    
    tokens.push_back(i[2].str().front() == '-' ? NEGATIVE_NUMBER : NON_NEGATIVE_NUMBER);
    
    if(i[3].length() > 0) {
        tokens.push_back(CLOSE_PARENTHESIS);
    }        
    
    auto it = next(cbegin(i), 4);
    
    for(int result = ADDITION; it != cend(i); ++result, ++it) {
        if (it->length() > 0U) {
            tokens.push_back(static_cast<TOKENS>(result));
            break;
        }
    }
});

match_results<string::const_reverse_iterator> sm;

if(regex_search(crbegin(input), crend(input), sm, regex{ tokens.back() == SUBTRACTION ? "^\\s*\\d+\\s*-\\s*(-?)" : "^\\s*\\d+\\s*(-?)" })) {
    tokens.push_back(sm[1].length() == 0 ? NON_NEGATIVE_NUMBER : NEGATIVE_NUMBER);
}

```

[Live Example](http://ideone.com/Rv5WNI)

A notable gotcha with regex iterators is that the `regex` argument must be an L-value, an R-value will not work: [Visual Studio regex_iterator Bug?](http://stackoverflow.com/q/29895747/2642059)

> regex 迭代器的一个值得注意的问题是，“regex”参数必须是 L 值，R 值将不起作用：[Visual Studio regex_iterator Bug？](http://stackoverflow.com/q/29895747/2642059)

## Anchors

C++ provides only 4 anchors:

- `^` which asserts the start of the string
- `$` which asserts the end of the string
- `\b` which asserts a `\W` character or the beginning or end of the string
- `\B` which asserts a `\w` character

Let's say for example we want to capture a number **with** it's sign:

```cpp
auto input = "+1--12*123/+1234"s;
smatch sm;

if(regex_search(input, sm, regex{ "(?:^|\\b\\W)([+-]?\\d+)" })) {

    do {
        cout << sm[1] << endl;
        input = sm.suffix().str();
    } while(regex_search(input, sm, regex{ "(?:^\\W|\\b\\W)([+-]?\\d+)" }));
}

```

[Live Example](http://ideone.com/uE4dGr)

An important note here is that the anchor does not consume any characters.

> 这里需要注意的一点是，锚点不消耗任何字符。

## regex_replace Example

This code takes in various brace styles and converts them to [One True Brace Style](https://en.wikipedia.org/wiki/Indent_style#Variant:_1TBS):

> 此代码接受各种大括号样式，并将它们转换为 [One True brace Style](https://en.wikipedia.org/wiki/Indent_style#Variant:_1TBS)：

```cpp
const auto input = "if (KnR)\n\tfoo();\nif (spaces) {\n    foo();\n}\nif (allman)\n{\n\tfoo();\n}\nif (horstmann)\n{\tfoo();\n}\nif (pico)\n{\tfoo(); }\nif (whitesmiths)\n\t{\n\tfoo();\n\t}\n"s;

cout << input << regex_replace(input, regex("(.+?)\\s*\\{?\\s*(.+?;)\\s*\\}?\\s*"), "$1 {\n\t$2\n}\n") << endl;

```

[Live Example](http://ideone.com/ICR5wM)

## regex_token_iterator Example

A [`std::regex_token_iterator`](http://en.cppreference.com/w/cpp/regex/regex_token_iterator) provides a tremendous tool for [extracting elements of a Comma Separated Value file](http://stackoverflow.com/a/28880605/2642059). Aside from the advantages of iteration, this iterator is also able to capture escaped commas where other methods struggle:

> 一个[`std:：regex_token_iterator`](http://en.cppreference.com/w/cpp/regex/regex_token_iterator)为[提取逗号分隔值文件的元素]提供了一个强大的工具([http://stackoverflow.com/a/28880605/2642059](http://stackoverflow.com/a/28880605/2642059))。除了迭代的优点之外，这个迭代器还能够在其他方法难以使用的地方捕获转义逗号：

```cpp
const auto input = "please split,this,csv, ,line,\\,\n"s;
const regex re{ "((?:[^\\\\,]|\\\\.)+)(?:,|$)" };
const vector<string> m_vecFields{ sregex_token_iterator(cbegin(input), cend(input), re, 1), sregex_token_iterator() };

cout << input << endl;

copy(cbegin(m_vecFields), cend(m_vecFields), ostream_iterator<string>(cout, "\n"));

```

[Live Example](http://ideone.com/lySlTJ)

A notable gotcha with regex iterators is, that the `regex` argument must be an L-value. [An R-value will not work](http://stackoverflow.com/q/29895747/2642059).

> regex 迭代器的一个显著问题是，“regex”参数必须是 L 值。[R 值无效](http://stackoverflow.com/q/29895747/2642059)。

## Splitting a string

```cpp
std::vector<std::string> split(const std::string &str, std::string regex)
{
    std::regex r{ regex };
    std::sregex_token_iterator start{ str.begin(), str.end(), r, -1 }, end;
    return std::vector<std::string>(start, end);
}

```

```cpp
split("Some  string\t with whitespace ", "\\s+"); // "Some", "string", "with", "whitespace"

```

## Quantifiers

Let's say that we're given `const string input` as a phone number to be validated. We could start by requiring a numeric input with a **zero or more quantifier**: `regex_match(input, regex("\\d*"))` or a **one or more quantifier**: `regex_match(input, regex("\\d+"))` But both of those really fall short if `input` contains an invalid numeric string like: "123" Let's use a **n or more quantifier** to ensure that we're getting at least 7 digits:

> 假设我们得到了“const 字符串输入”作为要验证的电话号码。我们可以从要求一个数字输入带有**零或多个量词**：`regex_match（input，regex（“\\d*”）` 或**一个或多个量化词**：`regx_match，input，正则x（“\\ d+”）` 开始。但如果 `input` 包含一个无效的数字字符串，如：“123”，那么这两个数字都不能满足要求。让我们使用一个 **n 或更多的量词**来确保我们得到至少 7 个数字：

```cpp
regex_match(input, regex("\\d{7,}"))

```

This will guarantee that we will get at least a phone number of digits, but `input` could also contain a numeric string that's too long like: "123456789012". So lets go with a **between n and m quantifier** so the `input` is at least 7 digits but not more than 11:

> 这将保证我们至少会得到一个数字的电话号码，但“输入”也可能包含太长的数字字符串，如：“123456789012”。因此，让我们使用 n 和 m 之间的**量词**，这样“输入”至少是 7 位数字，但不超过 11 位：

```cpp
regex_match(input, regex("\\d{7,11}"));

```

This gets us closer, but illegal numeric strings that are in the range of [7, 11] are still accepted, like: "123456789" So let's make the country code optional with a **lazy quantifier**:

> 这让我们更接近了，但[7，11]范围内的非法数字字符串仍然可以接受，比如：“123456789”所以让我们用**惰性量词**使国家代码可选：

```cpp
regex_match(input, regex("\\d?\\d{7,10}"))

```

It's important to note that the **lazy quantifier** matches **as few characters as possible**, so the only way this character will be matched is if there are already 10 characters that have been matched by `\d{7,10}`. (To match the first character greedily we would have had to do: `\d{0,1}`.) The **lazy quantifier** can be appended to any other quantifier.

> 需要注意的是，**惰性量词**与**匹配的字符尽可能少，因此匹配该字符的唯一方法是，如果已经有 10 个字符被“\d｛7，10｝”匹配。（为了贪婪地匹配第一个字符，我们必须这样做：`\d｛0,1｝`。）**懒惰量词**可以附加到任何其他量词后面。

Now, how would we make the area code optional **and** only accept a country code if the area code was present?

> 现在，我们如何使区号可选**，并且**只接受存在区号的国家/地区代码？

```cpp
regex_match(input, regex("(?:\\d{3,4})?\\d{7}"))

```

In this final regex, the `\d{7}` **requires** 7 digits. These 7 digits are optionally preceded by either 3 or 4 digits.

> 在最后一个正则表达式中，`\d｛7｝` **需要** 7 位数字。这 7 位数字前面可选地加上 3 位或 4 位数字。

Note that we did not append the **lazy quantifier**: <strike>`\d{3,4}?\d{7}`</strike>, the `\d{3,4}?` would have matched either 3 or 4 characters, preferring 3. Instead we're making the non-capturing group match at most once, preferring not to match. Causing a mismatch if `input` didn't include the area code like: "1234567".

> 请注意，我们没有附加**懒惰量词**：＜strike＞`\d｛3，4｝？\d｛7｝`</strike>，`\d｛3，4｝？` 将匹配 3 或 4 个字符，更喜欢 3 个。相反，我们最多进行一次非捕获小组赛，宁愿不匹配。如果“input”不包括区号（如：“1234567”），则会导致不匹配。

In conclusion of the quantifier topic, I'd like to mention the other appending quantifier that you can use, the **possessive quantifier**. **Either** the **lazy quantifier** or the **possessive quantifier** can be appended to any quantifier. The **possessive quantifier**'s only function is to assist the regex engine by telling it, greedily take these characters **and don't ever give them up even if it causes the regex to fail**. This for example doesn't make much sense: `regex_match(input, regex("\\d{3,4}+\\d{7}))` Because an `input` like: "1234567890" wouldn't be matched as `\d{3,4}+` will always match 4 characters even if matching 3 would have allowed the regex to succeed.<br>

> 在量词主题的结尾，我想提到你可以使用的另一个附加量词，**所有格量词******惰性量词**或**所有格量词**都可以附加到任何量词上。**所有格量词**的唯一功能是通过告诉正则表达式引擎，贪婪地接受这些字符**，并且永远不要放弃它们，即使这会导致正则表达式失败**。例如，这没有多大意义：`regex_match（input，regex（“\\d｛3，4｝+\\d{7｝））` 因为像“1234567890”这样的 `input` 不会匹配，因为 `\d｛3｝+` 将始终匹配 4 个字符，即使匹配 3 会使 regex 成功。<br>

The **possessive quantifier** is best used **when the quantified token limits the number of matchable characters**. For example:

> 当量化标记限制了可匹配字符的数量**时，最好使用**所有格量词**。例如：

```cpp
regex_match(input, regex("(?:.*\\d{3,4}+){3}"))

```

Can be used to match if `input` contained any of the following:

>

<p>123 456 7890<br>
123-456-7890<br>
(123)456-7890<br>
(123) 456 - 7890</p>

But when this regex really shines is when `input` contains an **illegal** input:

> 但是，当“input”包含**非法**输入时，这个正则表达式才会真正发光：

>

12345 - 67890

Without the **possessive quantifier** the regex engine has to go back and test **every combination of `.*` and either 3 or 4 characters** to see if it can find a matchable combination. With the **possessive quantifier** the regex starts where the 2<sup>nd</sup> **possessive quantifier** left off, the '0' character, and the regex engine tries to adjust the `.*` to allow `\d{3,4}` to match; when it can't the regex just fails, no back tracking is done to see if earlier `.*` adjustment could have allowed a match.

> 如果没有**所有格量词**，正则表达式引擎必须返回并测试**“.*”和 3 或 4 个字符**的每个组合，看看是否能找到匹配的组合。使用**所有格量词**，正则表达式从 2<sup>nd</sup>**所有量词**停止的位置开始，即“0”字符，正则表达式引擎尝试调整“.*”以允许“\d｛3，4｝”匹配；如果不能，regex 就会失败，则不会进行回溯，以查看早期的“.*”调整是否允许匹配。

#### Syntax

- regex_match // Returns whether the entire character sequence was matched by the regex, optionally capturing into a match object

> -regex_match//返回 regex 是否匹配整个字符序列，可选择捕获到匹配对象中

- regex_search // Returns whether a portion of the character sequence was matched by the regex, optionally capturing into a match object

> -regex_search//返回 regex 是否匹配字符序列的一部分，可以选择捕获到匹配对象中

- regex_replace // Returns the input character sequence as modified by a regex via a replacement format string

> -regex_replace//返回 regex 通过替换格式字符串修改的输入字符序列

- regex_token_iterator // Initialized with a character sequence defined by iterators, a list of capture indexes to iterate over, and a regex. Dereferencing returns the currently indexed match of the regex. Incrementing moves to the next capture index or if currently at the last index, resets the index and hinds the next occurrence of a regex match in the character sequence

> -regex_token_iterator//用迭代器定义的字符序列、要迭代的捕获索引列表和 regex 初始化。取消引用返回正则表达式的当前索引匹配项。递增移动到下一个捕获索引，或者如果当前处于最后一个索引，则重置索引并阻止字符序列中出现下一个正则表达式匹配

- regex_iterator // Initialized with a character sequence defined by iterators and a regex. Dereferencing returns the portion of the character sequence the entire regex currently matches. Incrementing finds the next occurrence of a regex match in the character sequence

> -regex_iterator//用迭代器和 regex 定义的字符序列初始化。取消引用返回整个正则表达式当前匹配的字符序列部分。递增在字符序列中查找下一个正则表达式匹配项

#### Parameters

| Signature                                                                                                                                          | Description                                                                                                                                                                                                                                                                                                                                                                                                          |  |  |  |  |  |  |  |  |
| -------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | - | - | - | - | - | - | - | - |
| `bool regex_match(BidirectionalIterator first, BidirectionalIterator last, smatch& sm, const regex& re, regex_constraints::match_flag_type flags)` | **`BidirectionalIterator`** is any character iterator that provides increment and decrement operators **`smatch`** may be `cmatch` or any other other variant of `match_results` that accepts the type of `BidirectionalIterator` the `smatch` argument may be ommitted if the results of the regex are not needed **Returns** whether `re` matches the entire character sequence defined by `first` and `last`      |  |  |  |  |  |  |  |  |
| `bool regex_match(const string& str, smatch& sm, const regex re&, regex_constraints::match_flag_type flags)`                                       | **`string`** may be either a `const char*` or an L-Value `string`, **the functions accepting an R-Value `string` are explicitly deleted** **`smatch`** may be `cmatch` or any other other variant of `match_results` that accepts the type of `str` the `smatch` argument may be ommitted if the results of the regex are not needed **Returns** whether `re` matches the entire character sequence defined by `str` |  |  |  |  |  |  |  |  |
