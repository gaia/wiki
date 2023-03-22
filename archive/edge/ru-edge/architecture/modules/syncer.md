---
id: syncer
title: Syncer
description: Объяснение к модулю Syncer в Polygon Edge.
keywords:
  - docs
  - polygon
  - edge
  - architecture
  - module
  - synchronization
---

## Обзор {#overview}

Это модуль содержит логику для протокола синхронизации. Он используется для синхронизации нового нода с работающей сетью или для валидации и вставки новых блоков для нодов, которые не участвуют в консенсусе (не валидаторы).

Polygon Edge использует **libp2p** как сетевой уровень, а поверх него используется **gRPC**.

В Polygon Edge есть 2 типа синхронизации:
* Bulk Sync — синхронизация большого количества блоков за раз
* Watch Sync — синхронизация по отдельным блокам

### Bulk Sync {#bulk-sync}

Шаги процедуры Bulk Sync простые и предназначены для достижения цели процедуры Bulk Sync — синхронизировать как можно больше блоков (доступных) от других узлов для максимального быстрой актуализации.

Выполнение процесса Bulk Sync выглядит так:

1. ** Определить, требуется ли ноду синхронизация Bulk Sync **: на этом шаге нод проверяет карту узлов, чтобы определить, существуют ли другие ноды с большим количеством блоков, чем у этого нода (локально)
2. ** Найти лучший нод (используя карту синхронизации узлов) **: на этом шаге нод находит лучший узел для синхронизации, основываясь на критериях, указанных в примере выше.
3. ** Открыть поток массовой синхронизации **: на этом шаге нод открывает поток gRPC для лучшего узла для пакетной синхронизации блоков с общим количеством блоков
4. ** Лучший узел закрывает поток после завершения массовой отправки**: на этом шаге лучший узел, с которым нод выполняет синхронизацию, закрывает поток после отправки всех доступных на нем блоков
5. ** После завершения пакетной синхронизации следует проверить, является ли нод валидатором **: на этом шаге лучший узел закрывает поток, и нод проверяет, является ли он валидатором после пакетной синхронизации.
  * Если нод становится валидатором, он выходит из состояния синхронизации и начинает участвовать в консенсусе
  * Если нет, они продолжают участвовать в синхронизации ** Watch Sync **

### Watch Sync {#watch-sync}

:::info

Шаг синхронизации Watch Sync выполняется, только если нод не является валидатором, а является обычным нодом сети, который прослушивает поступающие блоки.

:::

Процедура Watch Sync допольно простая, поскольку нод уже синхронизирован с остальной сетью и ему нужно только обрабатывать новые поступающие блоки.

Это шаги, из которых состоит процесс Watch Sync:

1. ** Добавьте новый блок при обновлении статуса узла **: на этом шаге ноды прослушивают новые события блоков, а при поступлении нового блока он вызывает функцию gRPC, получает блок и обновляет локальное состояние.
2. ** После синхронизации последнего блока проверьте, является ли нод валидатором **
   * Если да, он выводится из состояния синхронизации
   * Если нет, он продолжает прослушивать новые события блока

## Отчет о производительности {#perfomance-report}

:::info

Измерение производительности производилось на локальном компьютере посредством синхронизации ** миллиона блоков **

:::

| Название | Результат |
|----------------------|----------------|
| Синхронизация 1 млн блоков | ~ 25 мин |
| Передача 1 млн блоков | ~ 1 мин |
| Количество вызовов GRPC | 2 |
| Охват теста | ~ 93% |