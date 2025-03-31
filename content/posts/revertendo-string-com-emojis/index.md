+++
title = "Revertendo string com emojis"
slug = "revertendo-string-com-emojis"
date = 2021-11-26

[taxonomies]
tags=["javascript"]
+++

Quando queremos separar uma string letra por letra ou até mesmo reverter um texto utilizamos o método [`split()`](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Global_Objects/String/split), porém, existem diversos problemas em se utilizar este método, a própria documentação faz um breve comentário sobre [reverter uma String usando `split("")`](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Global_Objects/String/split#revertendo_uma_string_usando_split) não ser a melhor opção.

<!-- more -->

## Problemas do `String.prototype.split("")`

Ao fazer `.split("")` a divisão não é feita "letra por letra" por conta da maneira que o Javascript lida com o strings esta divisão pode não ocorrer como o esperado.

Por exemplo, enquanto este emoji `😃` é representado por 2 caracteres,  `😶‍🌫️` esse ocupa 6 caracteres. Isso é:

```javascript
> "a".length
1
> "😃".length
2
> "😶‍🌫️".length
6
> "H̵̙͗ė̴̘l̴̥͒ḷ̶͂o̶̰͝".length
20
```

E o  resultado de tentar utilizar este método para lidar com simbolos mais complexos é esse:

```javascript
> "Rust & Go".split("")
Array(9) [ "R", "u", "s", "t", " ", "&", " ", "G", "o" ]

> "𝟘𝟙𝟚𝟛".split('')
Array(8) [ "\ud835", "\udfd8", "\ud835", "\udfd9", "\ud835", "\udfda", "\ud835", "\udfdb" ]

> '🦀 Rust & 🐹 Go'.split('')
Array(15) [ "\ud83e", "\udd80", " ", "R", "u", "s", "t", " ", "&", … ]

> "😶‍🌫️".split('')
Array(6) [ "\ud83d", "\ude36", "‍", "\ud83c", "\udf2b", "️" ]
```

## Tentativas falhas

Nenhum desses métodos resolvem o problema de fato e todos retornam os mesmos output. Emojis compostos `[ "😶‍🌫️", "🏳️‍🌈", ...]` ainda não serão divididos corretamente.

* Spread syntax (`...`)

Utilizar o [spread operator (`...`)](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Operators/Spread_syntax) na String dentro de um array.

```javascript
> [..."🦀 Rust & 🐹 Go"]
Array(13) [ "🦀", " ", "R", "u", "s", "t", " ", "&", " ", "🐹", … ]

> [..."😶‍🌫️"]
Array(4) [ "😶", "‍", "🌫", "️" ]
```

* `Array.from`

Nesse método o [`Array.from`](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Global_Objects/Array/from) te permite transformar a string em um array.

* RegExp `u` flag[`String.prototype.split(" ")`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/split)

Nesse método é utilizado [RegExp](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Global_Objects/RegExp), não diria ser uma boa escolha pela legibilidade e compressão do código.

* Reduce method

Nesse método, por conta do [`for...of`](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Statements/for...of), é necessário criar uma variável para armazenar os valores.

## Solução

A solução recente que encontrei com um amigo é utilizando o [Intl.Segmenter()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/Segmenter). Que server para padronizar datas e textos de acordo com o local, [permitindo obter grafemas, palavras ou sentenças de forma precisa](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/Segmenter#basic_usage_and_difference_from_string.prototype.split).

* Generator

```javascript
function* splitEmoji(s) { 
  for (const ch of new Intl.Segmenter().segment(s)) 
    yield ch.segment;
}

console.log([...splitEmoji('😶‍🌫️ 🏁')].reverse()) // ['🏁', ' ', '😶‍🌫️']
console.log([...splitEmoji('👨‍👨‍👧‍👦 🇧🇷')].reverse()) // ['🇧🇷', ' ', '👨‍👨‍👧‍👦']
```

* Spread Operator and Array.from

```javascript
const segmenter = new Intl.Segmenter()

console.log([...segmenter.segment('😶‍🌫️ 🏁')].map(({ segment }) => segment).reverse())
// ['🏁', ' ', '😶‍🌫️']

console.log(Array.from(segmenter.segment('👨‍👨‍👧‍👦 🇧🇷')).map(({ segment }) => segment).reverse())
// ['🇧🇷', ' ', '👨‍👨‍👧‍👦']
```
