# Сборка и качество кода

<p class="reading-time">Чтение: 6 минут</p>

Сборщик берет исходные модули и ресурсы проекта, строит граф зависимостей и создает файлы для публикации. В материалах потока Vite используется в `Desserts`, `GitHub User Finder` и React-уроке. Программа также включает Webpack и Babel.

## Основные части конфигурации

```js
export default {
  mode: "production",
  entry: "./src/index.js",
  output: {
    filename: "bundle.js",
    clean: true,
  },
  module: {
    rules: [
      {
        test: /\.css$/i,
        use: ["style-loader", "css-loader"],
      },
    ],
  },
};
```

- `entry` задает входной модуль;
- `output` описывает результат;
- loaders учат Webpack обрабатывать не только JavaScript;
- plugins выполняют более общие действия;
- development удобен для работы, production оптимизирует результат.

## npm scripts

```json
{
  "scripts": {
    "start": "webpack serve --open",
    "build": "webpack",
    "lint": "eslint src"
  }
}
```

Команды проекта должны быть одинаковыми для всей команды. `package-lock.json` фиксирует версии зависимостей и тоже хранится в Git.

## Разные инструменты

- Prettier форматирует код;
- ESLint ищет потенциальные ошибки и нарушения правил;
- Husky запускает проверки на Git-событиях;
- source map связывает собранный код с исходниками;
- GitHub Pages публикует статический результат.

## Vite, Webpack и Babel

- Vite быстро отдает модули в разработке и собирает production-версию;
- Webpack дает детальную настройку графа, loaders, plugins и чанков;
- Babel преобразует синтаксис JavaScript под целевые браузеры;
- ESLint ищет проблемы в коде, Prettier отвечает за форматирование.

## Мини-практика

Объясните путь `src/main.js` до файлов в `dist`. Затем соберите `Desserts` через `npm run build` и проверьте production preview.

## Проверьте себя

1. Для чего нужны `entry` и `output`?
2. Чем ESLint отличается от Prettier?
3. Почему `node_modules` не коммитят, а lock-файл коммитят?

[Следующая тема: данные и условия →](js-basics.md)
