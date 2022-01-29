> Artigo original [aqui](https://blackode.github.io/elixir-tips/)
> TODO: Traduzir o artigo aqui para servir de documentação.

# Dicas de Elixir

Dicas e truques do Elixir a partir da experiência de desenvolvimento. Cada parte consiste em 10 dicas e truques exclusivos com uma explicação clara com exemplos ao vivo e resultados. Essas dicas irão acelerar o seu desenvolvimento e poupar tempo na digitação do código.

Você pode ler partes específicas com os seguintes links ...

## Convenções de nomenclatura de aplicativos Mix

Até este ponto, escrevemos código principalmente em iex ou colocamos todo o nosso código em um único arquivo.

O Mix permite que você estruture seu código de uma maneira mais organizada e lidará com o carregamento correto de seu código quando o aplicativo for executado.

Existem algumas convenções de nomenclatura que você seguiria:

- Use um módulo de nível superior para evitar colisões de nomes
- Cada módulo deve estar em um único arquivo, e cada arquivo deve ter um único módulo
- Os nomes dos arquivos devem ser escritos em snake
- Os nomes dos módulos devem ser "camel case"
- A estrutura da pasta deve espelhar os nomes dos módulos

### Exemplo p/ o modulo:

```elixir
defmodule Fvi.ServiceTest do
  #...
end
```

Teremos o arquivo `service_test.ex` e um diretório `ServiceTest`

## Dialyxir

- Ver [aqui](https://github.com/jeremyjh/dialyxir).

## Creating Custom Sigils and Documenting

Each x sigil calls its respective sigil_x definition

### Defining Custom Sigils

```elixir
defmodule MySigils do
  #returns the downcasing string if option l is given then returns the list of downcase letters
  def sigil_l(string,[]), do: String.downcase(string)
  def sigil_l(string,[?l]), do: String.downcase(string) |> String.graphemes

  #returns the upcasing string if option l is given then returns the list of downcase letters
  def sigil_u(string,[]), do: String.upcase(string)
  def sigil_u(string,[?l]), do: String.upcase(string) |> String.graphemes
end
```

### Usage

Load the module into iex

```elixir
iex> import MySigils
iex> ~l/HELLO/
"hello"

iex> ~l/HELLO/l
["h", "e", "l", "l", "o"]
iex> ~u/hello/
"HELLO"

iex> ~u/hello/l
["H", "E", "L", "L", "O"]
```

## Definicao de Erros Customizaveis

### Define Custom Error

```elixir
defmodule BugError do
   defexception message: "BUG BUG .." # message is the default
end
```

### Usage

```elixir
iex bug_error.ex

iex> raise BugError
** (BugError) BUG BUG ..
iex> raise BugError, message: "I am Bug.." #here passing the message dynamic
** (BugError) I am Bug..
```

## Escreva Protocolos

- Ver este post [aqui](https://samuelmullen.com/articles/elixir-protocols/).
- Ver esta conversa [aqui](https://stackoverflow.com/questions/26215206/difference-between-protocol-behaviour-in-elixir).

### Define a Protocol

A Protocol is a way to dispatch to a particular implementation of a function based on the type of the parameter. The macros defprotocol and defimpl are used to define Protocols and Protocol implementations respectively for different types in the following example.

```elixir
defprotocol Triple do
  def triple(input)
end
```

```elixir
defimpl Triple, for: Integer do
  def triple(int) do
    int * 3
  end
end
```

```elixir
defimpl Triple, for: List do
  def triple(list) do
    list ++ list ++ list
  end
end
```

### Usage

Load the code into iex and execute

```elixir
iex> Triple.triple(3)
9
Triple.triple([1, 2])
[1, 2, 1, 2, 1, 2]
```

## Ternary Operator

There is no ternary operator like true ? "yes" : "no" . So, the following is suggested.

```elixir
"no" = if 1 == 0, do: "yes", else: "no"
```

## Binary Pattern Matching

This is my recent discovery. I always encounter a situation like converting "$34.56" which is a string and I suppose do arithmetic operations. I usually do something like this before binary pattern matching..

```elixir
iex> value = "$34.56"
iex ...      |> String.split("$")
iex ...      |> tl
iex ...      |> List.first
iex ...      |> String.to_float
34.56
```

### Tip Approach

This tip makes my day easy. I recently used this is in one of my projects.

```elixir
iex> "$" <> value = "$34.56"
"$34.56"
iex> String.to_float value
34.56
```

## The Goodness of Data decoration [ inspect with :label ] option

We can decorate our output with inspect and label option. The string of label is added at the beginning of the data we are inspecting.

```elixir
iex(1)> IO.inspect [1, 2, 3], label: "the list "
the list : [1, 2, 3]
[1, 2, 3]
```

If you closely observe this it again returns the inspected data. So, we can use them as intermediate results in |> pipe operations like following……

```elixir
[1, 2, 3]
|> IO.inspect(label: "before change")
|> Enum.map(&(&1 * 2))
|> IO.inspect(label: "after change")
|> length
```

You will see the following output

```
before change: [1, 2, 3]
after change: [2, 4, 6]
3
```

## Piping Anonymous functions

We can pass the anonymous functions in two ways. One is directly using &like following..

```elixir
[1, 2, 3, 4, 5]
|> length()
|> (&(&1*&1)).()
```

This is the most weirdest approach. How ever, we can use the reference of the anonymous function by giving its name.

```elixir
square = & &1 * &1
[1, 2, 3, 4, 5]
|> length()
|> square.()
```

The above style is much better than previous . You can also use fn to define anonymous functions.

## Retrieve Character Integer Codepoints — `?`

We can use `?` operator to retrieve character integer codepoints.

```elixir
iex> ?a
97

iex> ?#
35
```

The following two tips are mostly useful for beginners…

## Performing Subtraction on Lists

We can perform the subtraction over lists for removing the elements in list.

```elixir
iex> [1, 2, 3, 4.5] -- [1, 2]
[3, 4.5]

iex> [1, 2, 3, 4.5, 1] -- [1]
[2, 3, 4.5, 1]

iex> [1, 2, 3, 4.5, 1] -- [1, 1]
[2, 3, 4.5]

iex> [1, 2, 3, 4.5] -- [6]
[1, 2, 3, 4.5]
```

We can also perform same operations on char lists too..

```elixir
iex(12)> 'blackode' -- 'ode'
'black'

iex(13)> 'blackode' -- 'z'
'blackode'
```

If the element to subtract is not present in the list then it simply returns the list.

## Running Multiple Mix Tasks

```elixir
mix do deps.get,compile
```

You can run multiple tasks by separating them with coma `,`.
How ever you can also create aliases in your mix project in a file called `mix.exs`.
The project definition looks like the following way when you create one using a `mix` tool.

```elixir
def project do
    [app: :project_name,
     version: "0.1.0",
     elixir: "~> 1.4-rc",
     build_embedded: Mix.env == :prod,
     start_permanent: Mix.env == :prod,
     deps: deps()]
  end
```

You are also allowed to add some extra fields…
Here you have to add the aliases field.

```elixir
[
 aliases: aliases()
]
```

Don’t forget to add , at the end when you add this in the middle of list .

The aliases() should return the key-value list.

```elixir
defp aliases do
[
"ecto.setup": ["ecto.create", "ecto.migrate", "ecto.seed"]
]
end
```

So, whenever you run the mix ecto.setup the three tasks ecto.create, ecto.migrate and ecto.seed will run one after the other.
You can also add them directly as following unlike I did with private function.

```elixir
def project do
  [app: :project_name,
  version: "0.1.0",
  aliases: ["ecto.setup": ["ecto.create", "ecto.migrate", "ecto.seed"]]
  #.....
end
```
