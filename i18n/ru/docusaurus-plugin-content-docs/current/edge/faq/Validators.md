---
id: validators
title: Часто задаваемые вопросы о валидаторах
description: "Часто задаваемые вопросы о валидаторах Polygon Edge"
keywords:
  - docs
  - polygon
  - edge
  - FAQ
  - validators

---

## Как добавить/удалить валидатор? {#how-to-add-remove-a-validator}

### PoA {#poa}
Добавление/удаление валидаторов осуществляется путем голосования. Вы можете найти руководство по этому поводу [здесь](/docs/edge/consensus/poa).

### PoS {#pos}
Вы можете найти руководство о том, как поставить средства [здесь](/docs/edge/consensus/pos-stake-unstake), чтобы нод мог стать валидатором, и как отказаться от стейка (удалить валидатор).

Просим обратить ваше внимание на следующее:
- Вы можете использовать генезис-флаг `--max-validator-count`, чтобы установить максимальное количество стейкеров, которые могут присоединиться к набору валидаторов.
- Вы можете использовать генезис-флаг `--min-validator-count `, чтобы установить минимальное количество стейкеров, необходимых для присоединения к набору валидаторов (по умолчанию `1`).
- Когда максимальное количество валидаторов достигнуто, вы больше не можете добавлять валидаторы до тех пор, пока не удалите уже существующий валидатор из набора (не имеет значения, если размер стейка выше у нового валидатора). Если вы удалите валидатор, количество оставшихся валидаторов не может быть ниже `--min-validator-count`.
- Для получения статуса валидатора по умолчанию установлен порог равный `1`единиц валюты нативной сети (газа).



## Сколько пространства на диске рекомендуется для валидатора? {#how-much-disk-space-is-recommended-for-a-validator}

Мы рекомендуем начать с 100G в качестве консервативно оцененной дорожки и убедиться, что впоследствии можно будет масштабировать диск.


## Существует ли лимит на количество валидаторов? {#is-there-a-limit-to-the-number-of-validators}

Если говорить о технических ограничениях, то в Polygon Edge отсутствует явное ограничение на количество нодов, которые вы можете иметь в сети. Вы можете установить лимиты соединений (количество входящих/исходящих соединений) для каждого нода.

Если говорить о практических ограничениях, то производительность кластера из 100 нодов будет более низкой, чем у кластера из 10 нодов. Увеличивая количество нодов в кластере, вы увеличиваете сложность взаимодействия и накладные расходы на сеть в целом. Все зависит от того, на какой сети вы работаете и какая у вас топология сети.

## Как переключиться с PoA на PoS? {#how-to-switch-from-poa-to-pos}

PoA — это механизм консенсуса по умолчанию. Для нового кластера, чтобы переключиться на PoS, необходимо добавить флаг `--pos` при генерации генезис-файла. Если у вас есть запущенный кластер, то инструкции, как переключиться, можно найти [здесь](/docs/edge/consensus/migration-to-pos). В [разделе о консенсусе](/docs/edge/consensus/poa) можно найти всю информацию о нашем механизме консенсуса и настройке.

## Как обновить ноды при внесении серьезных изменений? {#how-do-i-update-my-nodes-when-there-s-a-breaking-change}

Подробное руководство можно найти [здесь](/docs/edge/validator-hosting#update).

## Можно ли настроить минимальный размер стейка для PoS Edge? {#is-the-minimum-staking-amount-configurable-for-pos-edge}

Минимальный размер стейка по умолчанию составляет `1 ETH`, и его нельзя настраивать.

## Почему такие команды JSON RPC, как `eth_getBlockByNumber`и `eth_getBlockByHash`, не возвращают адрес майнера? {#not-return-the-miner-s-address}

В настоящее время в Polygon Edge используется консенсус IBFT 2.0, который, в свою очередь, основан на механизме голосования, описанном в Clique PoA: [ethereum/EIPs#225](https://github.com/ethereum/EIPs/issues/225).

Если посмотреть на EIP-225 (Clique PoA), это соответствующая часть, которая объясняет, для чего используется  `miner`( `beneficiary`):

<blockquote>Мы перепрофилируем поля заголовка ethash следующим образом:<ul>
<li><b>бенефициар/майнер:</b> адрес для предложений о внесении изменений в список авторизованных подписчиков.</li>
<ul>
<li>Обычно заполняется нулями, изменяется только во время голосования.</li>
<li>Тем не менее, произвольные значения разрешаются (даже бессмысленные, такие как голосование против тех, кто не подписался), чтобы избежать дополнительных сложностей в реализации механики голосования. </li>
<li>Необходимо заполнить нулями в блоках контрольной точки (т. е. перехода эпохи).</li>
</ul>

</ul>

</blockquote>

Таким образом, поле `miner` используется для предложения голосования за определенный адрес, а не для того, чтобы показать автора предложения блока.

Информацию об авторе предложения блока можно найти путем восстановления pubkey из Seal, поля дополнительных данных Istanbul в заголовках блоков, закодированных RLP.

## Какие части и значения Genesis можно смело изменить? {#which-parts-and-values-of-genesis-can-safely-be-modified}

:::caution

Перед попыткой его редактировать, пожалуйста, убедитесь, что вручную создать файл genesis.json. Кроме того, вся цепочка должна быть остановлена, прежде чем редактировать файл genesis.json. После изменения файла генеза его необходимо распространить на все ноды не валидатора и валидатора.

:::

**Только раздел загрузки файла genesis может быть безопасно изменен**, где можно добавить или удалить bootnodes из списка.