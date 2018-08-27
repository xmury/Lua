# Знакомство с Lua за 15 минут
```lua
-- две черты начинают однострочный комментарий
--[[
    Двойные \[\[ \]\] делают 
    коменнтарий многострочным
]]

----------------------------------------------------
-- 1. Переменные и управление потоком.
----------------------------------------------------

num = 42 - Все числа дробные.
- Не волнуйся, 64-битные числа имеют 52 бит для
- сохранение точных значений int; точность машины
- не проблема для int, которым требуется < 52 бит.

s = 'walternate' - неизменяемые строки, такие же как в Python.
t = "двойные кавычки так прекрасны"
u = [[  Двойные скобки
        начало и конец
        многострочные строки.]]
t = nil -- undefined t; У Lua есть сбор мусора.

-- Блоки обозначаются такими ключевыми словами, как do / end:
while num < 50 do
   num = num + 1 -- Отсутствуют операторы ++ или +=.
end

-- If условия:
if num > 40 then
  print('over 40')
elseif s ~= 'walternate' then  -- ~= не равно.
  -- Проверка равенства  == like Python; ok for strs.
  io.write('not over 40\n')  -- Стандартный вывод.
else
  -- Переменные глобальны по умолчанию.
  thisIsGlobal = 5  -- CamelCase Распространнён.
 -- https://ru.wikipedia.org/wiki/CamelCase 

  -- Создадим локальную переменную:
  local line = io.read()  -- Читает следующую строку stdin.

  -- Для склеивания строк используется оператор: ..
  print('Зима близко, ' .. line)
end

-- Неопределенная переменная возвращает nil.
-- Это не ошибка:
foo = anUnknownVariable  -- Сейчас foo = nil.

aBoolValue = false

-- Только nil и false ложны; 0 и '' правда!
if not aBoolValue then print('twas false') end

-- 'или' и 'и' укорочены.
-- Это похоже на оператор a?C:c в C/js:
ans = aBoolValue and 'yes' or 'no'  --> 'no'

karlSum = 0
for i = 1, 100 do  -- Диапазон включает оба конца.
  karlSum = karlSum + i
end

-- Используйте "100, 1, -1" в качестве диапазона для обратного отсчета:
fredSum = 0
for j = 100, 1, -1 do fredSum = fredSum + j end

-- В общем, диапазон начинается, end [, step].

-- Еще одна конструкция цикла:
repeat
  print('the way of the future')
  num = num - 1
until num == 0

----------------------------------------------------
-- 2. Функции.
----------------------------------------------------

function fib(n)
  if n < 2 then return 1 end
  return fib(n - 2) + fib(n - 1)
end

-- Замыкания и анонимные функции это нормально:
function adder(x)
  -- Возвращаемая функция создается, когда adder
  -- вызывается и запоминает значение x:
  return function (y) return x + y end
end
a1 = adder(9)
a2 = adder(36)
print(a1(16))  --> 25
print(a2(64))  --> 100

-- Возвраты, функциональные вызовы и назначения все работают
-- со списками, которые могут быть несовместимы по длине.
-- Несогласованные приемники равны нулю;
-- несогласованные отправители отбрасываются.

x, y, z = 1, 2, 3, 4
-- Теперь x = 1, y = 2, z = 3 и 4 выбрасывается.

function bar(a, b, c)
  print(a, b, c)
  return 4, 8, 15, 16, 23, 42
end

x, y = bar('zaphod')  --> выводит "zaphod  nil nil"
-- Теперь x = 4, y = 8, значения 15..42 отбрасываются.

-- Функции являются  first-class, могут быть локальными / глобальными.
-- То же самое:
function f(x) return x * x end
f = function (x) return x * x end

-- И так:
local function g(x) return math.sin(x) end
local g; g  = function (x) return math.sin(x) end
-- the 'local g' decl makes g-self-references ok.

-- Между прочим, тригонометрические функции работают в радианах.

-- Скобки с одним параметром строки не нужны.
print 'hello'  -- Работает отлично.

----------------------------------------------------
-- 3. Таблицы.
----------------------------------------------------

-- Таблицы = единственная составная структура данных Lua;
-- они являются ассоциативными массивами.
-- Подобно php-массивам или js-объектам, они являются хеш 
-- словарями, поэтому их также можно использовать в качестве списков.
-- Использование таблиц в качестве словаре/карт: 

-- Ключи словаря по умолчанию являются строками:
t = {key1 = 'value1', key2 = false}

-- Строковые ключи могут использовать js-подобную точечную нотацию:
print(t.key1)  -- Выведет 'value1'.
t.newKey = {}  -- Добавляет новую пару ключ / значение.
t.key2 = nil   -- Удаляет ключ2 из таблицы.

-- Literal notation for any (non-nil) value as key:
u = {['@!#'] = 'qbert', [{}] = 1729, [6.28] = 'tau'}
print(u[6.28])  -- prints "tau"

-- Key matching is basically by value for numbers
-- and strings, but by identity for tables.
a = u['@!#']  -- Now a = 'qbert'.
b = u[{}]     -- We might expect 1729, but it's nil:
-- b = nil since the lookup fails. It fails
-- because the key we used is not the same object
-- as the one used to store the original value. So
-- strings & numbers are more portable keys.

-- A one-table-param function call needs no parens:
function h(x) print(x.key1) end
h{key1 = 'Sonmi~451'}  -- Prints 'Sonmi~451'.

for key, val in pairs(u) do  -- Table iteration.
  print(key, val)
end

-- _G is a special table of all globals.
print(_G['_G'] == _G)  -- Prints 'true'.

-- Using tables as lists / arrays:

-- List literals implicitly set up int keys:
v = {'value1', 'value2', 1.21, 'gigawatts'}
for i = 1, #v do  -- #v is the size of v for lists.
  print(v[i])  -- Indices start at 1 !! SO CRAZY!
end
-- A 'list' is not a real type. v is just a table
-- with consecutive integer keys, treated as a list.
```