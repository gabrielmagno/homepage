---
layout: post
title: "Uma breve introdução ao Rust"
lang: pt
---

Esses dias criei meu primeiro projetinho em Rust, o [crab-dlna](https://github.com/gabrielmagno/crab-dlna). Gostei bastante da linguagem e vou destacar aqui o que eu mais gostei e achei interessante. Se você sempre teve curiosidade de saber qual que é a desse tal de Rust, chega mais!

![No topo ao centro o logotipo do Rust (uma roda dentada preta com um R dentro dela), seguido do escrito "The Rust Programming Language". Embaixo à esquerda o logotipo do crates (uma pilha de caixas amarelas com o logotipo do Rust). Embaixo à direita o Ferris, mascote do Rust (desenho de um carangueijo alaranjado com olhos e um sorriso simpático).](/imgs/posts/breve-introducao-rust/01-logos.png)

## Motivações

A motivação mais recente pra eu aprender Rust foi descobrir que ele é, por 6 anos consecutivos, a linguagem de programação mais amada segundo o [survey do Stack Overflow](https://insights.stackoverflow.com/survey/2021)!!

Eu venho usando Rust indiretamente há alguns anos, porque uso umas ferramentas de linha de comando no dia a dia que são feitas em Rust: 
  - [ripgrep](https://github.com/BurntSushi/ripgrep), alternativa ao `grep`.
  - [bat](https://github.com/sharkdp/bat), um `cat` com syntax highlighting.
  - [exa](https://github.com/ogham/exa), um `ls` cheio de recursos.
  - [fd](https://github.com/sharkdp/fd), alternativa ao `find`.
  - etc...

Depois vi que a Mozilla passou a usar Rust em algumas coisas dentro do Firefox, e também foi onde a linguagem surgiu. Hoje ela [financia a Rust Foundation](https://blog.mozilla.org/en/mozilla/mozilla-welcomes-the-rust-foundation/). Também ouvia uns pesquisadores e programadores da área da criptografia no Twitter usando e elogiando o Rust, a ponto de existir um [projeto pra reescrever o Tor em Rust](https://blog.torproject.org/announcing-arti/). Até o Linus Torvalds deu um voto de confiança no Rust, que acabou sendo [incorporado como segunda linguagem oficial no Kernel do Linux](https://zdnet.com/article/rust-takes-a-major-step-forward-as-linuxs-second-official-language/), além do C .

Daí nas férias separei um tempinho pra finalmente tentar aprender Rust, e saciar minha curiosidade de anos. O que fiz foi ler (um pouco) do [livro oficial gratuito do Rust](https://doc.rust-lang.org/book/), e desenvolver um projetinho pessoal.

## A Linguagem

A primeira coisa pra se destacar é que o Rust é uma linguagem compilada, e tem performance de tempo de execução e consumo de memória equivalentes à de C e C++.
![Um gráfico de barras verticais, onde cada barra ao longo do eixo X é uma linguagem de programação, e o eixo y é o tempo de execução de programa. As linguagens estão ordenadas da mais rápida para a mais lenta. Temos C, C++ e Rust como as mais rápidas (tempo pŕoximo de 1). Mais à direita temos o Go (tempo próximo de 3) e Java (tempo próximo de 4).](/imgs/posts/breve-introducao-rust/08-benchmark.png)

Rust é uma linguagem **estaticamente tipada**. Ou seja, o compilador precisa saber os tipos de todas as variáveis em tempo de compilação. Claro que tem um sistema de inferência de tipos, então você não precisa ser explícito toda hora.
```rust
let a = 1;
let b: i32 = 2;
let c = 2.0;
let d: f32 = 3.0;
let e = "Opa! 🦀";

fn soma(x: i32, y: i32) -> i32 {
    x + y
}

let f = soma(a, b);
```

Rust te permite criar novos tipos com uma **struct**, que nada mais é do que uma tupla com campos nomeados, podendo cada campo ser de um tipo diferente.
```rust
struct Tweet {
    id: u64,
    username: String,
    text: String,
    has_retweet: bool,
}

let my_tweet = Tweet {
    id: 1,
    username: String::from("@gabrielmagno"),
    text: String::from("Hello World!"),
    has_retweet: false,
};
```

Em Rust você também pode criar um **enum**, que é um tipo que enumera suas possíveis _variantes_. Você pode armazenar dados no enum, com campos e tipos diferentes entre as variantes. É como se fosse uma mistura de enum e union do C.
```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}

let msg_1 = Message::Write("Opa, bão?".to_string());
let msg_2 = Message::Write("Bão, e vc?".to_string());
let msg_3 = Message::ChangeColor(10, 30, 200);
let msg_4 = Message::Move { x: 10, y: 20 };
```

Tanto _structs_ quanto _enums_ podem ter métodos associados aos mesmos, permitindo que valores desses tipos tenham comportamentos associados a ele.
```rust
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

let my_rectangle = Rectangle {
    width: 10,
    height: 20,
};

println!("Area: {}", my_rectangle.area());
```

Rust tem **pattern matching**. Isso permite criar blocos `match` que condicionem a execução de acordo com o tipo identificado, e também que se "desconstrua" as estruturas dos tipos. O compilador vai checar se os padrões são validos e se todos os valores possíveis foram listados.
```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}

impl Message {
    fn to_string(&self) -> String {
        match self {
            Message::Quit => "Quit".to_string(),
            Message::Move { x, y } => format!("Move: x={}, y={}", x, y),
            Message::Write(text) => format!("Write: {}", text),
            Message::ChangeColor(r, g, b) => 
                format!("Change color: r={}, g={}, b={}", r, g, b),
        }
    }
}
```

Na biblioteca padrão do Rust existem dois tipos de Enum disponíveis que são bastante utilizados: `Option` e `Result`.
```rust
enum Option<T> {
    Some(T),
    None,
}

enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

Rust não tem valor ou ponteiro nulo. Ele tem um enum chamado **Option**, que pode ter um valor válido (`Some(T)`) ou nenhum valor (`None`). Com isso o Rust te "obriga" a tratar os 2 casos (valor válido, valor inválido) antes de usar, prevenindo que seu programa utilize ou repasse nulls.
```rust
// NOTE: podiamos ter simplesmente 
// retornado `s.chars().nth(1).unwrap()`
fn get_second_char(s: &str) -> Option<char> {
    if s.len() > 1 {
        Some(s.chars().nth(1).unwrap())
    } else {
        None
    }
}

let maybe_a_char = get_second_char("Opa!");
match maybe_a_char {
    Some(the_char) => println!("Segundo char: {}", the_char),
    None => println!("Não tem segundo char"),
}
```

Rust não tem exceções. O tratamento de erros é feito com o enum **Result**, que pode ter um valor válido (`Ok(T)`) ou um erro (`Err(e)`). Isso é legal porque você sempre é capaz de saber se uma função pode dar erro ou não, e o Rust vai te "obrigar" a tratar os 2 casos (`Ok` e `Err`).
```rust
use std::fs::File;

let maybe_a_file = File::open("hello.txt");

let file = match maybe_a_file {
    Ok(file) => file,
    Err(error) => panic!(
        "Problema abrindo arquivo: {:?}", 
        error
    ),
};
```

Tanto _Options_ quanto _Results_ têm uma série de métodos que facilitam fazer algumas operações comuns. Por exemplo, `unwrap` vai extrair o valor e forçar um erro caso não tenha; `unwrap_or` vai extrair o valor, e caso não tenha vai retornar o valor especificado.
```rust
let maybe_int = Some(33);

let definetely_int = maybe_int.unwrap();
let definetely_int = maybe_int.expect("No int found");
let int_or_10 = maybe_int.unwrap_or(10);
```

Rust não tem classes nem herança. O polimorfismo de tipos acontece através do conceito de **traits**. As traits te permitem definir um _comportamento_ de forma genérica. Daí você pode implementar comportamentos pros tipos que você cria, e pode também criar suas próprias traits.
```rust
struct Circle {
    radius: f64,
}

struct Rectangle {
    width: u32,
    height: u32,
}

trait Rotate {
    fn rotate(&mut self);
}

impl Rotate for Circle {
    fn rotate(&mut self) { 
        // Do nothing
    }
}

impl Rotate for Rectangle {
    fn rotate(&mut self) {
        let temp = self.width;
        self.width = self.height;
        self.height = temp;
    }
}
```

Rust não tem garbage collector (como Java), e vc não precisa alocar/desalocar memória manualmente (como C). O Rust usa uma terceira abordagem: o **sistema de ownership**, que é sua característica mais marcante (e talvez a mais difícil de aprender quando se vem de outras linguagens).

Se passamos uma variável pra uma função, essa função vira a dona do valor, e a variável original fica inválida. Porém podemos "emprestar" o valor para a função, sinalizando com um `&`. O borrow funciona de uma forma similar às referências em C, mas tem suas próprias regras.

```rust
struct Data {x: i32}

let a = Data {x: 1};

fn usar(d: Data) {
    println!("{}", d.x);
}

usar(a); // ao chamar a função, o valor de 'a' é
         // movido para a função, que agora é a dono do valor

println!("{}", a.x); // o valor não pertence mais a 'a', 
                     // então isso vai dar um erro de compilação


let b = Data {x: 2};

fn usar_emprestado(d: &Data) {
    println!("{}", d.x);
}

usar_emprestado(&b); // ao chamar a função, o valor de 'b' é
                     // emprestado para a função (passamos uma referência)

println!("{}", b.x) // vai funcionar, pois 'b' ainda é dono do valor
```

As regras de ownership usam o conceito de **lifetime**, que é a definição de por quanto "tempo" (escopo) o valor vai existir. Quando um valor sai de seu escopo ele deixa de ser válido e é desalocado. Em alguns casos precisamos definir explicitamente o lifetime das variáveis.
```rust
struct Data {x: i32}

let a = Data {x: 1};

// Definimos com `'a` que a saída da função vai ter 
// o mesmo lifetime que a entrada d. Neste caso o 
// lifetime não precisava ser definido explicitamente,
// mas em casos que existe ambiguidade, é necessário.
fn data_number<'a>(d: &'a Data) -> &'a i32 {
    &d.x
}

let x = data_number(&a);
```

Rust tem um sistema de **macros** bem sofisticado, que permite fazer _metaprogramação_ (código que gera código). O comando básico de imprimir (`println!`) é uma macro do tipo declarativa, caracterizada pelo `!` no final do nome. Podemos criar novas macros com o comando `macro_rules!`.
```rust
macro_rules! multiply_add {
    ($a:expr, $b:expr, $c:expr) => {$a * ($b + $c)};
}

let result = multiply_add!(2, 3, 4);

println!("O resultado é: {}", result);
```

Existe também um segundo tipo de macro: as macros procedurais. Elas geralmente vão receber código (ao invés de expressões), processar, e gerar novos códigos. Como exemplo temos algumas traits padrões que você pode implementar simplesmente usando a macro derive.
```rust
// A trait Debug permite, entre outras coisas, 
// que eu imprima o conteúdo da minha struct. 
// A trait Clone implementa o método .clone().
// A trait Copy permite que eu copie o valor de uma
// variável para outra de forma automática.
#[derive(Debug, Clone, Copy)]
struct Rectangle {
    width: u32,
    height: u32,
}

let rec = Rectangle {
    width: 10,
    height: 20,
};

let rec_2 = rec.clone();
let rec_3 = rec;

println!("Retângulo 1: {:?}", rec);
println!("Retângulo 2: {:?}", rec_2);
println!("Retângulo 3: {:?}", rec_3);
```

## As Ferramentas

Tendo algumas características incomuns em outras linguagens, o Rust tem uma curva de aprendizado considerada alta. Porém seu compilador (`rustc`) é uma mãe! Mostra mensagens de erro excelentes, tudo bem indicado e explicado. Ele pega na sua mão e te guia pra resolver os problemas.
![Print da mensagem de erro do compilador, onde temos os trechos de códigos mostrados em cinza num fundo preto. Temos algumas mensagens em azul explicando onde uma variável está sendo utilizada, e uma mensagem em vermelho apontando exatamente onde no código acontece o erro com o texto "value borrowed here after move".](/imgs/posts/breve-introducao-rust/24-mensagem_compilador.png)

Outra coisa bem feita no Rust é seu universo de ferramentas, muitas delas oficiais. Temos por exemplo o cargo, que faz o build e gerencia as dependências do seu projeto. Por exemplo, com um `cargo new my_project` você cria um novo projeto, que gera um arquivo `Cargo.toml`.
```toml
# File: Cargo.toml
[package]
name = "crab-dlna"
version = "0.1.0"
authors = ["Gabriel Magno <gabrielmagno1@gmail.com>"]
license = "MIT OR Apache-2.0"
repository = "https://github.com/gabrielmagno/crab-dlna"
homepage = "https://github.com/gabrielmagno/crab-dlna"
description = "A minimal UPnP/DLNA media streamer"
categories = ["command-line-utilities", "multimedia", "multimedia::video"]
keywords = ["dlna", "upnp", "cli", "stream", "video"]
edition = "2021"

[dependencies]
log = "0.4"
pretty_env_logger = "0.3"
futures = "0.3"
tokio = { version = "1", features = ["full"] }
pin-utils = "0.1"
xml-rs = "0.8"
http = "0.2"
rupnp = "1.1"
warp = "0.3"
clap = { version = "3.1.15", features = ["derive"] }
slugify = "0.1.0"

[profile.release]
strip = true
```

Existem também ferramentas extras (acessadas através do cargo) como o `rustfmt` que vai checar e formatar o código de acordo com especificações padrão ou personalizadas (fornecendo um arquivo rustfmt.toml).
![Print da execução do comando `cargo fmt --check`. A saída é um diff, onde o conteúdo removido é destacado em vermelho, e o conteúdo adicionado é adicionado em verde. Um dos trechos temos uma struct que antes era definida em uma linha única, e agora está identada em 3 linhas. O segundo exemplo é um assignment, onde alguns espaços são adicionados depois dos colchetes.](/imgs/posts/breve-introducao-rust/27-rustfmt.png)

E existe também o `clippy`, um linter que vai identificar diversos padrões e te dar sugestões para melhorar o código.
![Print da execução do comando `cargo clippy`. A saída mostra uma linha específica de um arquivo main.rs, e temos destacado em amarelo parte do trecho de código que usa o método `unwrap_or`, com uma mensagem também em amarelo: "help: try this: unwrap_or_else".](/imgs/posts/breve-introducao-rust/28-clippy.png)

Em termos de recursos para IDEs e editores, eu usei o VS Code com a extensão (agora oficial) rust-analyzer (disponível também pra outros editores). Vai te dar syntax highlighting, complete, goto definition, goto references, documentation on hover, auto import, etc.
![Print da tela de uma sessão do VS Code, onde temos à esquerda a árvore de arquivos (com diversos arquivos com extensão .rs). À direita em destaque temos um código, onde temos um pequeno painel com a explicação do enum Query, com indicação que receber um parâmetro u64 e uma String, e com a descrição "Render specified by a query string". Na parte de baixo um terminal vazio.](/imgs/posts/breve-introducao-rust/29-vs_code.png)

## O ecosistema

Outra coisa a se elogiar é a documentação do Rust. Existe por exemplo um livro oficial: The Rust Programming Language. Disponível [gratuitamente online](https://doc.rust-lang.org/book/), e também de forma offline (`rustup docs --book`), embutido na instalação básica do Rust.

Aliado ao `cargo` temos o [crates.io](https://crates.io) que é o repositório público oficial de pacotes do Rust. Atualmente com mais de 83k pacotes disponíveis, classificados em 54 categorias.

Associado ao crates.io temos o [docs.rs](http://docs.rs), que vai hospedar automaticamente (gratuito e aberto) a documentação de todos os pacotes do crates. A qualidade varia, mas pelo menos as bibliotecas populares (ex: [tokio](https://docs.rs/tokio/)) são bem escritas e cheias de exemplos.

A comunidade me pareceu ser bem saudável e inclusiva. Existe um [servidor de discord oficial do Rust](https://discord.gg/rust-lang) que tem um canal "beginners", onde você pode conversar e tirar dúvidas. Na minha experiência as pessoas respondem rápido e de forma educada.

## Desvantagens

Rust não é uma linguagem perfeita. O primeiro problema que destaco é a curva de aprendizado mais alta que o comum, por conta de dois fatores: (1) o sistema de "borrow" e (2) o fato de não ser orientada a objetos. Mas como disse o compilador, as docs e a comunidade te ajudam.

A segunda desvantagem é o tempo de compilação: dependendo do seu projeto pode demorar bastante pra compilar. Como não é possível ter libs pre-compiladas (por conta do generics) o compilador tem mais trabalho. Mas eles têm trabalhado constantemente pra tornar o rustc mais rápido.

## Conclusões

Minha impressão final é de que o Rust é uma linguagem robusta. Produz programas eficientes e seguros. Segurança não só por conta do acesso de memória, mas por conta do sistema de tipos, que te impede em tempo de compilação de fazer cagada (acessar null, esquecer de tratar erro).

Por ter essa segurança toda, as vezes pode ser considerado uma linguagem chata. Mas como disse antes, o compilador geralmente vai te dizer exatamente onde está o problema, qual o problema, e muitas vezes como resolver o problema.

Outras referências que me não mencionei explicitamente, mas me ajudaram:
  - [A quick tour of Rust’s Type System Part 1: Sum Types (a.k.a. Tagged Unions)](https://tonyarcieri.com/a-quick-tour-of-rusts-type-system-part-1-sum-types-a-k-a-tagged-unions)
  - [A half-hour to learn Rust](https://fasterthanli.me/articles/a-half-hour-to-learn-rust)
  - [Clear explanation of Rust's module system](https://sheshbabu.com/posts/rust-module-system/)

_Este post foi adaptado de uma [Thread que escrevi no Twitter](https://twitter.com/GabrielMagno/status/1528505240110174209)._
