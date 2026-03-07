# Telegram Bots Book

[![book](https://img.shields.io/badge/TelegramBots-Book-blue.svg?style=flat)](https://telegrambots.github.io/book/)
[![master](https://github.com/TelegramBots/book/actions/workflows/ci.yml/badge.svg)](https://github.com/TelegramBots/book/actions/workflows/ci.yml)

Этот репозиторий содержит документацию для [TelegramBots](https://github.com/TelegramBots) проекта.
Эта книга - большой учебник к созданию Telegram ботов в экосистеме .NET 🤖.

## 🔨 Сборка & тестирование ✔

Эта книга является web приложением, сгенерированным из файлов markdown с использованием [mdBook].
Все markdown файлы, упомянутые в [SUMMARY](src/SUMMARY.md) буту отображаться как HTML страницы.

1. Install [mdBook]:
    - You can download a [mdBook binary]
    - Or install it using Rust package manager

      ```bash
      cargo install mdbook --vers "^0.4.28"
      ```

1. Run locally at [localhost:3000](http://localhost:3000):

    ```bash
    mdbook serve --open
    ```

## Contribute 👋

**Your contribution is welcome!** 🙂
See [Contribution Guidelines].

<!-- -->

[mdBook]: https://github.com/rust-lang/mdBook
[mdBook binary]: https://github.com/rust-lang/mdBook/releases
[Contribution Guidelines]: CONTRIBUTING.md
