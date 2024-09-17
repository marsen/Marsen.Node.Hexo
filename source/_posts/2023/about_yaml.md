---
title: " [翻譯] 在 X 分鐘學會 YAML "
date: 2023/04/19 14:33:33
---

## 本文

YAML 是一種用於數據序列化的語言，設計初衷是讓人類能夠直接讀寫。  
它是 JSON 的一個嚴格超集，具有類似 Python 的有意義的換行和縮排。  
不同的是，YAML 不允許使用文字 tab 字符進行縮排，而需要使用空格進行縮排。

## 範例檔

```yaml
---  # 文件開始

# YAML中的註解看起來像這樣。
# YAML支援單行註解。

################
#    純量類型    #
################

# 我們的 root 物件是一個 map,
# 就像是其它語言的 dictionary, hash 或 object .
key: value
another_key: Another value goes here.
a_number_value: 100
scientific_notation: 1e+12
hex_notation: 0x123  # 0x開頭為 16 進制，所以值為 291
octal_notation: 0123 # 0開頭為 8 進制，所以值為 83

# 1 會被視作數字而非 boolean 值
# 如果要用 boolean 請使用 true 

boolean: true
null_value: null
another_null_value: ~
key with spaces: value

# Yes No (不分大小寫) 也視作 boolean
# Yes→true 而 No→false 
# 如果要用實際值，請用單引號或雙引號
no: no            # 視作 false
yes: No           # 視作 false
not_enclosed: yes # 視作 true
enclosed: "yes"   # 視作字串 「yes」

# 請注意，字符串不需要引號。但是，它們可以使用引號。
however: 'A string, enclosed in quotes.'
'Keys can be quoted too.': "Useful if you want to put a ':' in your key."
single quotes: 'have ''one'' escape pattern'
double quotes: "have many: \", \0, \t, \u263A, \x0d\x0a == \r\n, and more."
# UTF-8/16/32字符需要編碼
Superscript two: \u00B2

# 特殊字符必須用單引號或雙引號括起來
special_characters: "[ John ] & { Jane } - <Doe>"

# 多行字符串可以使用“文字塊”（使用|）或“折疊塊”（使用'>'）來寫入。
# 文字塊會將字符串中的每個換行符轉換為文字換行符（\n）。
# 折疊塊則刪除字符串中的換行符。
literal_block: |
  This entire block of text will be the value of the 'literal_block' key,
  with line breaks being preserved.

  The literal continues until de-dented, and the leading indentation is
  stripped.

      Any lines that are 'more-indented' keep the rest of their indentation -
      these lines will be indented by 4 spaces.
folded_style: >
  This entire block of text will be the value of 'folded_style', but this
  time, all newlines will be replaced with a single space.

  Blank lines, like above, are converted to a newline character.

      'More-indented' lines keep their newlines, too -
      this text will appear over two lines.

# |- 和 >- 可以刪除結尾的空行（也被稱為文字塊的 "strip"）
literal_strip: |-
  This entire block of text will be the value of the 'literal_block' key,
  with trailing blank line being stripped.
block_strip: >-
  This entire block of text will be the value of 'folded_style', but this
  time, all newlines will be replaced with a single space and 
  trailing blank line being stripped.

# |+ 和  >+ 可以保留結尾的空行（也被稱為文字塊的 "keep"）
literal_keep: |+
  This entire block of text will be the value of the 'literal_block' key,
  with trailing blank line being kept.

block_keep: >+
  This entire block of text will be the value of 'folded_style', but this
  time, all newlines will be replaced with a single space and 
  trailing blank line being kept.

####################
#    集合類型    #
####################

# 巢狀結構使用縮排. 建議值為 2 個空白 (非必要).
a_nested_map:
  key: value
  another_key: Another Value
  another_nested_map:
    hello: hello

# Maps 不一定要用 string 作key.
0.25: a float key

# Keys 也可以很複雜, 比如多行物件
# 我們用 ?| 開頭表示複雜的物件
? |
  This is a key
  that has multiple lines
: and this is its value

# YAML 也可以用複雜物件作 Key Value 的映射
# 但有解析器可能會不過
# 如下例
? - Manchester United
  - Real Madrid
: [ 2001-01-01, 2002-02-02 ]

# 序列（Sequences 或 lists 或 arrays) 看起來像這樣
# (注意 '-' 有縮排):
a_sequence:
  - Item 1
  - Item 2
  - 0.5  # sequences 可以包含不同類型
  - Item 4
  - key: value
    another_key: another_value
    - - This is a sequence
      - inside another sequence
    - - - Nested sequence indicators
        - can be collapsed

# 因為 YAML 是 JSON 的超集, 你也可以寫出 JSON-風格的 maps 與 sequences:
json_map: { "key": "value" }
json_seq: [ 3, 2, 1, "takeoff" ]
and quotes are optional: { key: [ 3, 2, 1, takeoff ] }

#######################
# 額外 YAML 功能 #
#######################

# YAML 很方便的功能叫 'anchors', 讓你可以很輕易的複製文件內容
# 使用 & 來定義值
# 使用 * 來呼叫值
# 下面兩個 key 有相同的值
anchored_content: &anchor_name This string will appear as the value of two keys.
other_anchor: *anchor_name

# Anchors 可以用於複製與繼承的屬性
base: &base
  name: Everyone has same name

# << 表示式代表 'Merge Key Language-Independent Type'. 
# 用於合併已存在 map 成為新的 map 並保留 key value
# 注意: 如果合併時要合併的 key已經存在，則該鍵的別名（alias）不會被合併
# 也就是說，如果要合併的 key 已經存在於數據結構中，則合併操作只會更新該 key 的值
foo:
  <<: *base # doesn't merge the anchor
  age: 10
  name: John
bar:
  <<: *base # base anchor will be merged
  age: 20

# foo 與 bar 將會有相同的名字

# YAML 也有 tags, 請參考以下 Syntax
# Syntax: !![typeName] [value]
explicit_boolean: !!bool true
explicit_integer: !!int 42
explicit_float: !!float -42.24
explicit_string: !!str 0.5
explicit_datetime: !!timestamp 2022-11-17 12:34:56.78 +9
explicit_null: !!null null

# 其中有一些解析器（parser）會實作語言特定的 tag，像是 Python 的複數類型（complex number）
# 可以使用 !!python/complex 這個 tag 來表示複數。
# 在這個範例中，python_complex_number 變數的值被指定為一個 Python 複數類型的物件，其實際值為 1+2j。
python_complex_number: !!python/complex 1+2j

# 我們可以使用 yaml 的 keys 轉換為語言的特定資料類型
? !!python/tuple [ 5, 7 ]
: Fifty Seven
# 以上將被 Python 解析為 {(5, 7): 'Fifty Seven'} 

####################
# 額外的 YAML 類型 #
####################

# 除了基本型別外(string number)
# Yaml 也支援 ISO-formatted 的日期與時間 
datetime_canonical: 2001-12-15T02:59:43.1Z
datetime_space_separated_with_time_zone: 2001-12-14 21:59:43.10 -5
date_implicit: 2002-12-14
date_explicit: !!timestamp 2002-12-14

# 你可以用 !!binary 這個 tag 表示其後的值是 base64-encoded 的二進位編碼
gif_file: !!binary |
  R0lGODlhDAAMAIQAAP//9/X17unp5WZmZgAAAOfn515eXvPz7Y6OjuDg4J+fn5
  OTk6enp56enmlpaWNjY6Ojo4SEhP/++f/++f/++f/++f/++f/++f/++f/++f/+
  +f/++f/++f/++f/++f/++SH+Dk1hZGUgd2l0aCBHSU1QACwAAAAADAAMAAAFLC
  AgjoEwnuNAFOhpEMTRiggcz4BNJHrv/zCFcLiwMWYNG84BwwEeECcgggoBADs=

# YAML 也有集合(set)類型
set:
  ? item1
  ? item2
  ? item3
or: { item1, item2, item3 }

# Sets 將 maps 向 null 值; 上面的 yaml 範例將與下面相等
set2:
  item1: null
  item2: null
  item3: null

...  # document end
```

## 參考

- [Learn X in Y minutes](https://learnxinyminutes.com/docs/yaml/)
- [八、十六進制線上試算](https://www.convertworld.com/zh-hant/numerals/hexadecimal.html)

(fin)
