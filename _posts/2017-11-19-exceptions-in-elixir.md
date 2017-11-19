---
layout: post
title: "Обрабока исключений в Эликсир"
desc: "Learn the Erlang's Let if fail pholosophy and how to work with exceptions in Elixir."
keywords: "Elixir, exceptions, fail fast, let it fail, raise, throw, try, rescue, catch, error"
tags: [Elixir, Перевод]
author: "Vitaly Tatarintsev (ck3g)"
---

Эрланг поддерживает философию "Пусть что-то пойдет не так" (или "быстрого отказа"). Первое объяснение, что нет необходимости писать программы думая постоянно о надежности. Другая идея заключается в том, что если программа ожидает отказа, возможно это еще не катастрофа всего приложения :). 

Хочется упомянуть [Джо Амстронга](https://en.wikipedia.org/wiki/Joe_Armstrong_(programming)), который является основным изобретателем языка Эрланг. Его цитата находится на странице о [быстром отказе](http://wiki.c2.com/?FailFast) :

> Философия "быстрого отказа" занимает центральное место в языке Эрланг - девиз Эрланга "это всего лишь сбой" и "разреши другому процессу выполнить восстановление после ошибки" - было использовано в очень надежных производственных системах с миллионами строк кода -- Джо Амстронг

Эликсир также разделяет философию, что ошибки должны быть фатальными, а исключения предназначены для вещей, которые обычно никогда не должны происходить. Обычно в приложениях на Эликсир исключения выбрасываются, но иногда попадаются.

That does not mean, that you should ignore them completely. As soon as Elixir application usually consists of multiple processes, you may let them (processes) crash. Also, you should design your application in the way, that even if a process fails you application still keeps running. Of course crashed processes can have unfinished work, but you can design your application in a way to minimize these cases.

Теперь, после краткого введения, давайте посмотрим, что у нас есть.

### Конструкция raise/rescue

Для начала, мы можем вызвать исключение используя функцию `raise/1` :

```elixir
iex> raise "Something went wrong"
** (RuntimeError) Something went wrong
```

or, if we want to provide the type of exceptions we can do it using `raise/2` function:

```elixir
iex> raise RuntimeError, message: "Something went wrong"
** (RuntimeError) Something went wrong
```
Once an error has been raised and you want to handle it, then you can use `try/rescue` construct.
In the `rescue` block you can either use the error itself or just expose it as you want.

```elixir
iex> try do
...>   raise "Something went wrong"
...> rescue
...>   e in RuntimeError -> e
...> end
%RuntimeError{message: "Something went wrong"}

iex> try do
...>   raise "Something went wrong"
...> rescue
...>   RuntimeError -> "Error!!!"
...> end
"Error!!!"
```

### Последовательность after

There is also `after` block available for us. It will be executed as a final step regardless of raised exception or not.

```elixir
iex> try do
...>   raise "Something went wrong"
...> rescue
...>   e in RuntimeError -> e
...> after
...>   IO.puts("Cleaning up...")
...> end
Cleaning up...
%RuntimeError{message: "Something went wrong"}
```

You can use `after` for clean up operations, closing open files or any other termination tasks.


### Методы throw/catch

In Elixir there is a way to `throw` a value and then `catch` it later. Unlike `raise` you cannot `rescue` a value thrown earlier. So it has to be `catch`.

```elixir
iex> try do
...>   throw {:some, :value}
...> catch
...>   :throw, value -> "Here is your value: #{inspect value}"
...> end
"Here is your value: {:some, :value}"
```
Similar to `rescue` it is possible to use `after` block here.

```elixir
iex> try do
...>   throw {:some, :value}
...> catch
...>   :throw, value -> "Here is your value: #{inspect value}"
...> after
...>   IO.puts("Cleaning up...")
...> end
Cleaning up...
"Here is your value: {:some, :value}"
```

Similar to throw some processes can [exit/1](https://hexdocs.pm/elixir/Kernel.html#exit/1) or have an `:erlang.error/1`.
They both can be caught similar to `throw`.

### Определим собственные исключения

The same way as we define [Structs](http://whatdidilearn.info/2017/11/06/more-on-maps-and-structs-in-elixir.html#structs) we can define our own exceptions by using `defmodule` + `defexception`. It is also possible to define custom functions as we did with Structures.

```elixir
iex> defmodule MySpecialError do
...>   defexception message: "Something special went wrong"
...>
...>   def full_message(error) do
...>     "General failure: #{error.message}"
...>   end
...> end
{:module, MySpecialError,
 <<70, 79, 82, 49, 0, 0, 15, 180, 66, 69, 65, 77, 65, 116, 85, 56, 0, 0, 1, 130,
   0, 0, 0, 35, 21, 69, 108, 105, 120, 105, 114, 46, 77, 121, 83, 112, 101, 99,
   105, 97, 108, 69, 114, 114, 111, 114, 8, ...>>, {:full_message, 1}}
```

And then use it to raise from some place

```elixir
iex> try do
...>   raise MySpecialError
...> rescue
...>   e in MySpecialError -> IO.puts(MySpecialError.full_message(e))
...> end
General failure: Something special went wrong
:ok
```

## Подведем итоги

На этом мы заканчиваем введение в обработку исключений на Эликсире. Теперь мы знаем, как вызывать и обрабатывать несколько типов исключений и как определять свои собственные. 

[Источник](http://whatdidilearn.info/2017/11/19/exceptions-in-elixir.html)