# 9. Циклы и алгоритмы

<p class="reading-time">Чтение: 7 минут</p>

Цикл повторяет действие, а алгоритм задает точную последовательность шагов и условие завершения.

## `while` и `for`

```js
let index = 0;
while (index < colors.length) {
  console.log(colors[index]);
  index += 1;
}

for (let i = 0; i < colors.length; i += 1) {
  console.log(colors[i]);
}
```

`while` удобен, когда заранее неизвестно число повторов. `for` компактно хранит счетчик, условие и шаг. `break` завершает цикл, `continue` переходит к следующей итерации.

## Линейный поиск

```js
function findIndex(items, target) {
  for (let i = 0; i < items.length; i += 1) {
    if (items[i] === target) return i;
  }
  return -1;
}
```

В худшем случае алгоритм просмотрит весь массив: сложность `O(n)`.

## Бинарный поиск

```js
function binarySearch(sorted, target) {
  let left = 0;
  let right = sorted.length - 1;

  while (left <= right) {
    const middle = Math.floor((left + right) / 2);
    if (sorted[middle] === target) return middle;
    if (sorted[middle] < target) left = middle + 1;
    else right = middle - 1;
  }
  return -1;
}
```

Он отбрасывает половину области на каждом шаге и работает за `O(log n)`, но требует отсортированного массива.

## Сортировка

```js
const numbers = [12, 3, 25, 8];
const ascending = numbers.toSorted((a, b) => a - b);
```

Без функции сравнения `sort()` сортирует значения как строки. `sort()` изменяет исходный массив, `toSorted()` возвращает новый.

## Где это было

В уроках 16-20 есть пузырьковая сортировка, сортировка выбором, линейный и бинарный поиск, рекурсия и оценка сложности.

## Мини-практика

Найдите максимум массива одним проходом без `Math.max`. Затем объясните сложность решения и поведение на массиве из отрицательных чисел.

## Проверьте себя

1. Когда `while` удобнее `for`?
2. Какое условие обязательно для бинарного поиска?
3. Почему `[2, 10].sort()` может дать неожиданный результат?

[Следующая тема: строки, массивы и объекты →](collections.md)

