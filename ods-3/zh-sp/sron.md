# SRON -- ODS 3

> - Author: [DiannaoJun](mailto:aheiwuchang@163.com)
> - Date: 2025/04/30 20:46 UTC+08
> - Version: 0.1.0

## 简介
[SRON](https://github.com/SRON-org/SRON) (SRInternet Object Notation) 是由思锐发明的一种配置文件格式。

## 定义
```ebnf
<sron>          = [<line> (, <line>)*]
<line>          = <path> [, ':', <type>], '=', <value>
<path>          = <ident> (, '.', <ident>)*
<type>          = 'string' | 'integer' | 'float' | 'boolean' | 'null' | 'array' | <defined_class>
<defined_class> = (? 用户声明的类型名称 须匹配<ident> ?)
<ident>         = <ident_head> <ident_body>*
<ident_head>    = <alpha> | '_'
<alpha>         = <alpha_low> | <alpha_up>
<alpha_low>     = 'a' | 'b' | 'c' | ... | 'z'
<alpha_up>      = 'A' | 'B' | 'C' | ... | 'Z'
<ident_body>    = <ident_head> | <dec_body>
<value>         = <string> | <integer> | <real> | <boolean> | <null> | <array> | <defined_type>
<defined_type>  = (? 用户声明的类型 value 语法 ?)
<digit>         = '0b' <bin> | '0' <oct> | '0x' <hex> | '0t' <ter> | <dec>
<dec>           = <dec_body> | '+' <dec_body> | '-' <dec_body>
<bin>           = {<bin_dig> ['_']} <bin_dig>
<oct>           = {<oct_dig> ['_']} <oct_dig>
<ter>           = {<ter_dig> ['_']} <ter_dig>
<hex>           = {<hex_dig> ['_']} <hex_dig>
<dec_body>      = {<dec_dig> ['_']} <dec_dig>
<hex_dig>       = <dec_dig> | 'a' | 'b' | 'c' | ... | 'f' | 'A' | 'B' | 'C' |... | 'F'
<dec_dig>       = <oct_dig> | '8' | '9'
<oct_dig>       = <ter_dig> | '3' | '4' | ... | '7'
<ter_dig>       = <bin_dig> | '2'
<bin_dig>       = '0' | '1'
<float>         = ['-' | '+'] [<dec_body>] '.' [<dec_body>]
<real>          = <float> | <float> 'e' <dec> | <float> 'E' <dec>
<boolean>       = 'true' | 'false' | 'null' | '0' | '1'
<null>          = 'null'
<string>        = '"' <any>* '"' | "'" <any>* "'" | '"""' <any>* '"""' | "'''" <any>* "'''"
<array>         = '[', [<value>] (, ',', <value>)*, ']'
<space>         = ' ' | '\t' | '\n' | '\r' | '\f' | '\v' | ' ' | <note>
<note>          = '//' <any>* '\n' | '/*' <any>* '*/'

(*
    注意，本规则里单词内部字符之间不允许出现<space>
    而单词之间则允许出现<space>，且<space>会被忽略。
    单词内部字符间直接连接，单词间使用','连接。
*)

```
