# Отслеживание PR в cozystack

Обновлено: **2026-10-10**. Репозиторий по умолчанию — `cozystack/cozystack`, иначе указано явно.

На GitHub у issue и PR **общая нумерация**, по номеру их не отличить. Поэтому здесь issue всегда помечены словом — `issue #4083`, а голый `#4133` означает PR.

## Главное за 09–10.10

- **На #4778 пришёл независимый подтверждающий отчёт и прямой вопрос про бэкпорт.** 09.10 в 22:34 yrahman82 описал ту же коллизию на свежей установке v1.6.4 (три ноды, вариант kubeovn-cilium): под получил `CiliumInternalIP` ноды и после этого перестал доходить до любого порта на хосте этой ноды, включая apiserver и node-exporter. Как обход он держит текущие router и ingress IP cilium в `excludeIps` дефолтного сабнета — это ровно первый шаг нашего миграционного хука. Вопрос адресован нам: планируется ли бэкпорт в линию v1.6 или цель — v1.7. Ответа пока нет
- **У community#25 появилась реализация: draft #4817 от lllamnyp, открыт 08.10 в 10:30.** Фазы 1 и 2 проекта per-cluster etcd: шаблоны etcd-чарта переезжают в `cozy-lib`, новый тенантный кластер рендерит свой `EtcdCluster` и Kamaji `DataStore`, уже работающие кластеры остаются на общем etcd предка, миграция 60 → 61 только перечисляет их в логе. На CR ровно одно поле — `etcd.replicas` (3 по умолчанию либо 1), то самое минимальное API, которое мы предлагали в обсуждении proposal'а. На 10.10 — draft, ревью нет, с момента открытия без активности
- **Всё остальное без изменений с 09.10.** Девять бэкпортов в release-1.6 (#4516, #4614, #4677, #4683, #4684, #4725, #4739, #4740, #4780) — ревью нет ни у одного, обязательные прогоны по-прежнему в `action_required`, последнее касание головы 07.10; ботовский #4609 — всё ещё конфликтный draft; сам community#25 — `REVIEW_REQUIRED`, ноль ревью, последняя активность 03.09; issue #2483, issue #2793, issue #3950, issue #3022, issue #4258, issue #4342 — все OPEN; бэкпортов для семи смерженных 08.10 никто не подал, лейбла `kind/backport` ни на одном из семи нет; нового стабильного релиза после v1.6.4 (28.09) нет, последний pre-release — v1.7.0-alpha.3 от 29.09

### Предыдущий заход — 08.10

- **08.10 lexfrei смержил все семь наших открытых PR в main.** По времени мержа: #4812 в 11:14, #4789 в 13:50, #4778 в 18:41, #4779 в 20:54, #4724 в 21:10, #4766 в 23:17, #4682 в 23:21. У всех семи на смерженной голове `pre-commit` и `E2E Tests` зелёные — прогоны форка одобрили, и они прошли
- **#4778 получил повторный апрув lexfrei 08.10 в 10:11** — тот, что снялся 07.10 коммитом с догрузкой `test_helper`, — и в 18:41 уехал в main. Это ломающее изменение: на вариантах с kube-ovn cilium переведён на `ipam.mode: cluster-pool` с /29 на ноду из последнего /21 от `networking.podCIDR`, `networking.podCIDR` теперь не уже /20, потолок таких вариантов — 256 нод
- **#4789 и #4812 прошли весь путь за сутки**: открыты 07.10, APPROVED lexfrei 08.10 в 10:13 и 10:14, auto-merge включён тут же, смержены в 13:50 и 11:14
- **Два issue закрылись автоматически при мерже**: issue #3740 (cilium берёт адреса из pod CIDR kube-ovn) — мержем #4778 в 18:41, issue #4676 (нет редактора free-form полей в Form-режиме) — мержем #4682 в 23:21
- **Ребейз в этот раз сделал сам lexfrei, и апрувы не снялись.** 08.10 в 19:39 он force-push'нул головы #4682, #4724, #4766 и #4779 — ребейз на main, уже содержащий #4778, — и в 19:41 включил на всех четырёх auto-merge. Проверено: в таймлайне этих PR за 08.10 нет ни одного события `review_dismissed`, а единственный апрув у каждого — lexfrei от 06.10. `[предположение]` Причина — в правах того, кто пушит: у мейнтейнера с bypass ruleset'а правило `dismiss_stale_reviews_on_push` не сработало. Для нас вывод прежний: в одобренные PR не пушить самим
- **Бэкпортов для только что смерженного пока нет.** Для #4682, #4766, #4778, #4789 и #4812 ручных бэкпортов в release-1.6 никто не открывал, лейбла `kind/backport` ни на одном из семи нет. Это на нашей стороне. `[вывод]` #4789 и #4812 имеет смысл подавать после мержа #4684 и #4740 — они хвосты #4612 и #3799
- **issue #2483 и issue #2793 мерж #4766 не закрыл.** В теле PR они упомянуты без закрывающего ключевого слова, поэтому оба OPEN. Закрыть их теперь может только мейнтейнер руками
- **Девять бэкпортов в release-1.6 — без единого изменения.** #4516, #4614, #4677, #4683, #4684, #4725, #4739, #4740, #4780: ревью нет ни у одного, `pre-commit` и `Pull Request` на всех висят в `action_required`
- **Без изменений**: ботовский #4609 (конфликтный draft, закрывается), community#25 без движения с 03.09, issue #3950, issue #3022, issue #4258, issue #4342. Нового стабильного релиза после v1.6.4 (28.09) нет; последний pre-release — v1.7.0-alpha.3 от 29.09

## Требует действия

Всё в таблице — действия мейнтейнеров.

| Что | Где | Кому и что делать |
|---|---|---|
| **Ревью и запуск CI бэкпортов** | #4516, #4614, #4677, #4683, #4684, #4725, #4739, #4740, #4780 | Мейнтейнер: девять ручных бэкпортов в release-1.6, у всех ревью нет, `pre-commit` и `Pull Request` в `action_required`. Это единственное, что осталось от нашего скоупа. Ботовский #4609 дублирует #4614 конфликтным draft'ом — закрыть. `kind/backport` на оригиналы не вешать: ручные бэкпорты уже есть, будет дубль |
| **Закрыть issue** | issue #2483, issue #2793 | Мейнтейнер: оба решены мержем #4766 от 08.10 — static-seal auto-unseal и лейбл `allow-to-apiserver`. В теле PR они упомянуты без закрывающего ключевого слова, поэтому автоматом не закрылись |
| **Ревью proposal** | community#25 | Любой мейнтейнер: второй драфт с 21.08 без единого ревью, наш отчёт о прогоне миграции с 03.09 без ответа. 08.10 lllamnyp открыл draft #4817 — реализацию фаз 1 и 2 по этому proposal'у; `[вывод]` решение по документу стало срочнее, код уже пишется под него |
| **Закрыть issue** | issue #3022 | Мейнтейнер: по словам автора #4026 решён мержем #4152 и #2682 |

На нашей стороне: ответить yrahman82 в треде #4778 про бэкпорт в линию v1.6 — вопрос задан 09.10 в 22:34 и висит без ответа. И ручные бэкпорты в release-1.6 для #4682, #4766, #4789 и #4812 ещё не открыты. `[вывод]` #4789 и #4812 стоит подавать после мержа #4684 и #4740 — иначе черри-пик хвоста не приложится к родителю, которого в release-1.6 нет. `[предположение]` #4778 в патч-релиз 1.6.x не поедет вовсе: это `kind/breaking-change`.

**В одобренные PR ничего не пушить** — новый коммит снимает апрув (`dismiss_stale_reviews_on_push: true`). Именно так 07.10 снялся апрув на #4778. Два исключения проверены: переоткрытие PR апрув не снимает (07.10 на #4724, #4682, #4766, #4779) и force-push самого мейнтейнера тоже не снял (08.10 в 19:39 на тех же четырёх).

## Наши PR — открытые

В `main` открытых наших PR нет: 08.10 lexfrei смержил последние семь — #4812, #4789, #4778, #4779, #4724, #4766 и #4682. Их описания переехали в таблицу «Смержено в main» в конце страницы. Открытыми остались только бэкпорты в release-1.6.

### Бэкпорты в release-1.6 — открытые

Девять ручных бэкпортов yankawai плюс один ботовский — после 08.10 это весь наш открытый скоуп. У всех ручных ревью нет, и ни на одном обязательные проверки не стартовали: `pre-commit` и `Pull Request` в `action_required`, зелёные только `PR Auto-Label`, `PR size label`, `DCO` и CodeRabbit. `[вывод]` Это следующий патч-релиз 1.6.x: v1.6.4 от 28.09 их не содержит.

Для смерженного 08.10 бэкпортов пока нет: ни #4682, ни #4766, ни #4789, ни #4812 в release-1.6 не поданы, лейбла `kind/backport` ни на одном из семи нет. Это на нашей стороне.

| PR | Оригинал | Что переносит | Состояние |
|---|---|---|---|
| [#4780](https://github.com/cozystack/cozystack/pull/4780) | #4779 | mongodb: бакет и префикс на bootstrap-restore | открыт 06.10, ревью нет, CI не стартовал |
| [#4740](https://github.com/cozystack/cozystack/pull/4740) | #3799 | linstor и keda: severity, которые Alerta не отбрасывает | открыт 04.10, ревью нет, CI не стартовал. Голова обновлена 07.10 в 17:23 |
| [#4739](https://github.com/cozystack/cozystack/pull/4739) | #4738 | platform: легаси etcd-оператор остаётся в нуле после adoption | открыт 03.10, ревью нет, CI не стартовал |
| [#4725](https://github.com/cozystack/cozystack/pull/4725) | #4724 | postgres: cap `shared_buffers` | открыт 02.10, синхронизирован с оригиналом 06.10 в 21:48, ревью нет, CI не стартовал |
| [#4684](https://github.com/cozystack/cozystack/pull/4684) | #4612 | dashboard и foundationdb: вкладка ConfigMaps тенанту | открыт 01.10, голова обновлена 07.10 в 08:00, ревью нет, CI не стартовал |
| [#4683](https://github.com/cozystack/cozystack/pull/4683) | #4610 | cozystack-api: GC может финализировать Application | открыт 01.10, голова обновлена 06.10 в 21:56, ревью нет, CI не стартовал |
| [#4677](https://github.com/cozystack/cozystack/pull/4677) | #4675 | nats: JetStream для сгенерированного аккаунта | открыт 01.10, ревью нет, CI не стартовал |
| [#4614](https://github.com/cozystack/cozystack/pull/4614) | #3800 | monitoring: почтовый receiver в Alertmanager | открыт 30.09, чистый ручной бэкпорт, ревью нет, CI не стартовал |
| [#4609](https://github.com/cozystack/cozystack/pull/4609) | #3800 | то же, ботовский по лейблу `kind/backport` | конфликтнул — draft с закоммиченными маркерами, DCO в `action_required`. Дубль #4614, подлежит закрытию |
| [#4516](https://github.com/cozystack/cozystack/pull/4516) | #4366 | tcp-balancer: переименование бэкендов и пин HAProxy | открыт 26.09, ревью нет, CI не стартовал |

## Чужие PR — открытые

Взят в трекинг 10.10: реализация per-cluster etcd по community#25 — proposal, в обсуждении которого принято наше предложение минимального API.

| PR | Автор | Название | Что решает | Состояние |
|---|---|---|---|---|
| [#4817](https://github.com/cozystack/cozystack/pull/4817) | lllamnyp | feat(kubernetes): give each tenant Kubernetes cluster its own etcd | Тенантный кластер больше не ждёт, пока у предка появится `spec.etcd: true`, и не делит один etcd, один CA и один диск со всеми соседями по поддереву: у каждого нового кластера свой `EtcdCluster` и Kamaji `DataStore`, создаются и удаляются вместе с кластером, состояние `awaiting-etcd` исчезает. На CR одно поле `etcd.replicas` — 3 или 1 | Открыт 08.10 в 10:30, **draft**, ревью нет, с момента открытия без активности. Фазы 1 и 2 из community#25; фазы 3 и 4, бэкап собственного etcd и opt-in миграция — в follow-up |

## Живая миграция VM — вся связка в main

15.09 yankawai открыл пачку из четырёх PR вокруг одной темы — чтобы VM переезжали между хостами живьём, а не убивались. 18.09 в main доехал последний из них.

| PR | Слой | Что даёт | Состояние |
|---|---|---|---|
| #4254 | платформа → KubeVirt | настраивать миграции платформенными values | **смержен 18.09** |
| website#697 | документация | описание #4254 | **смержен 17.09** |
| #4253 | сеть, Kube-OVN | миграция не зависает на настройке сети целевого пода | **смержен 18.09** |
| #4291 | Cluster API, CAPK | при выводе хоста VM сначала мигрирует, потом дренируется | **смержен 18.09** |

Итог: миграции настраиваются через `kubevirt.migrations`, сеть целевого пода не теряется, а CAPK при выводе хоста мигрирует VM вместо удаления.

## E2E падал одинаково — эпизод закрыт

17.09 шесть несвязанных PR падали на одном наборе сьютов: backup-roundtrip для redis, rabbitmq, postgres, mongodb, плюс kafka и opensearch. 18.09 всё разрешилось:

- перезапуски E2E прошли на всех этих PR, включая сьют opensearch: #4100 и #3935, красные ещё днём, к вечеру смержены с зелёным E2E
- #4254, #4184 и #4291 lexfrei смержил, не дожидаясь зелёного E2E

`[вывод]` И массовое падение backup-roundtrip, и державшееся дольше падение opensearch были нестабильностью тестового стенда, а не регрессиями PR: одни и те же коммиты падали и проходили без изменений кода.

Красные статусы, висевшие на одобренных PR 05–07.10, были от двух поломок main, уже починенных там: контракт-тест `internal/fluxcontract` (05.10 в 15:10) и сборка тестов `internal/backupcontroller` (#4768, 07.10 в 04:36). Повторный запуск упавшего прогона идёт на том же тестовом merge-коммите и от такой поломки не спасает; пуш базу обновляет, но снимает апрувы. 07.10 в 17:13–17:14 автор обошёл это переоткрытием PR — закрыть и сразу открыть, GitHub пересобирает merge-коммит на текущем main, голова PR не меняется, апрувы остаются. Проверено: у #4724, #4682, #4766 и #4779 тестовые merge-коммиты созданы в эти минуты на main `cbc9d0c4`, где оба фикса есть, и `reviewDecision` у всех четырёх по-прежнему `APPROVED`. Платой идут две вещи: закрытие снимает auto-merge, а новый прогон для форка снова ждёт «Approve and run workflows».

**Эпизод закрыт мержем.** 08.10 у всех семи PR `pre-commit` и `E2E Tests` на смерженной голове зелёные — прогоны форка одобрили, и они прошли, мерж мимо красной проверки никому не понадобился. Обнаружился и четвёртый способ обновить базу: 08.10 в 19:39 lexfrei отребейзил головы #4682, #4724, #4766 и #4779 force-push'ем сам, и апрувы при этом не снялись — событий `review_dismissed` за 08.10 нет. `[предположение]` Правило `dismiss_stale_reviews_on_push` не сработало из-за bypass-прав пушащего; на наши собственные пуши это не распространяется.

Из двух issue про backup round-trip issue #4262 закрыт 23.09 lexfrei, issue #4258 открыт.

### Зоны CODEOWNERS

Для файла действует последнее совпавшее правило. Пример на scooby87 — именно на нём 17.09 было видно, как работает требование:

| Путь | scooby87 владелец? |
|---|---|
| `/packages/apps/` | **да** |
| `/packages/system/kubevirt*/`, `/packages/system/vm-*/`, `gpu-operator`, `hami` | **да** |
| `/packages/system/` — всё остальное, включая `linstor`, `kubeovn`, Cluster API | нет |
| `/packages/core/` — платформа | нет |
| `/hack/` — в том числе e2e | нет |
| `/api/`, `/cmd/`, `/internal/`, `/pkg/` | нет |

Основная пятёрка — kvaps, lllamnyp, lexfrei, myasnikovdaniil, IvanHunters — владеет всем. Для `packages/system/` также sircthulhu и mattia-eleuteri. У `types.go`, `zz_generated.deepcopy.go` и `cozyrds/` владельцы не указаны вовсе — это снятие владения со сгенерированных файлов.

### Что держит мерж сейчас

| Что держит | PR |
|---|---|
| ревью и запуск CI бэкпортов в release-1.6 | #4516, #4614, #4677, #4683, #4684, #4725, #4739, #4740, #4780 |
| закрыть как дубль #4614 | #4609 |
| апрув proposal | community#25 |
| вывод из draft и ревью | #4817 |

Своих открытых PR в `main` у нас нет: #4817 — чужой, lllamnyp, и это реализация community#25.

## Связь PR и issue

`Fixes #NNNN` в теле PR — стандартный синтаксис GitHub: при мерже указанный issue закрывается автоматически. Четыре issue #4083–#4086 завёл yankawai 05.09, 07.09 выкатил на них PR — **18.09 закрыт последний из четырёх**.

| Issue | Название | PR | Состояние |
|---|---|---|---|
| [#4083](https://github.com/cozystack/cozystack/issues/4083) | clickhouse: Keeper is never created when the application name is longer than 15 characters | #4133 | **закрыт** мержем 17.09 |
| [#4084](https://github.com/cozystack/cozystack/issues/4084) | nats: config.merge.accounts duplicates the generated accounts key and breaks the install | #4134 | **закрыт** мержем 18.09 |
| [#4085](https://github.com/cozystack/cozystack/issues/4085) | harbor/clickhouse: cleanup hook Role lacks watch on persistentvolumeclaims | #4135 | **закрыт** мержем 18.09 |
| [#4086](https://github.com/cozystack/cozystack/issues/4086) | opensearch: data PVCs are left behind after the application is removed | #4136 | **закрыт** мержем 18.09 |
| [#3740](https://github.com/cozystack/cozystack/issues/3740) | networking: cilium takes its per-node infrastructure IPs from the pod CIDR kube-ovn allocates pods from | #4778 | **закрыт** мержем 08.10 в 18:41 — по `fixes` в теле PR |
| [#4676](https://github.com/cozystack/cozystack/issues/4676) | bug(dashboard): NATS free-form configuration fields have no editor in Form mode | #4682 | **закрыт** мержем 08.10 в 23:21 — по `Fixes` в теле PR |
| [#2483](https://github.com/cozystack/cozystack/issues/2483) | OpenBAO: can't init/unseal without kubectl exec | #4766 | **всё ещё открыт** — в теле #4766 упомянут без закрывающего слова, автоматом не закрылся. Решён смерженным 08.10 #4766, закрыть руками |
| [#2793](https://github.com/cozystack/cozystack/issues/2793) | OpenBAO HA (raft) pods hang at startup due to hardcoded service_registration "kubernetes" | #4766 | **всё ещё открыт** — та же причина. Решён мержем #4766, закрыть руками |

## Issues

| Issue | Автор | Название | Состояние |
|---|---|---|---|
| [#3950](https://github.com/cozystack/cozystack/issues/3950) | IvanHunters | Platform-wide defaults for tenant Talos worker settings | OPEN, **23.09 lexfrei принял в работу**: `triage/accepted` и `priority/important-longterm` вместо `triage/needs-triage`. Смерженный 24.09 #4171 закрыл каталожную сторону — платформенный ключ `kubernetesWorkerImage` позволяет направить импорт golden-образа в зеркало, — но потребляющая сторона дефолтов (`imageFactoryURL`, `installerRepository`, `registryMirrors` на `KubernetesNodes`) осталась. В треде готовый разбор: наш комментарий 07.09 о create-time дефолтинге, ответ myasnikovdaniil 08.09 (двум полям платформенный дефолт безопасен, `imageFactoryURL` — прокат флота), предупреждение lexfrei 01.09 о невозможности plain default chain в Helm. Шов на write-пути — #3956, **смержен 07.10** |
| [#3022](https://github.com/cozystack/cozystack/issues/3022) | lexfrei | OpenSearch fails to start in tenant namespaces: privileged init-sysctl violates baseline PodSecurity | OPEN, но **по сути решён**: #4152 поднимает `vm.max_map_count` DaemonSet'ом, #2682 выключил `setVMMaxMapCount`. Автор #4026 предложил закрыть |
| [#2483](https://github.com/cozystack/cozystack/issues/2483) | Arsolitt | OpenBAO: can't init/unseal without kubectl exec | OPEN, `triage/accepted`, `kind/feature`. Решён смерженным 08.10 #4766: opt-in `seal.type: static` снимает ручной `bao operator unseal` после каждого рестарта. В теле PR issue упомянут без закрывающего слова, поэтому автоматом не закрылся — нужен мейнтейнер |
| [#2793](https://github.com/cozystack/cozystack/issues/2793) | myasnikovdaniil | OpenBAO HA (raft) pods hang at startup due to hardcoded service_registration "kubernetes" | OPEN, `triage/accepted`, `priority/important-longterm`. Решён тем же #4766: поды получили лейбл `policy.cozystack.io/allow-to-apiserver`, и `service_registration "kubernetes"` дотягивается до apiserver. Закрыть руками |
| [#4073](https://github.com/cozystack/cozystack/issues/4073) | lexfrei | opensearch-operator: dnsBase stays cluster.local | **закрыт** 17.09 — фикс в #4185 |
| [#3793](https://github.com/cozystack/cozystack/issues/3793) | IvanHunters | Deleting a tenant with a Kafka app hangs the namespace in Terminating (KafkaTopic strimzi.io/topic-operator finalizer) | **закрыт** 23.09 lexfrei — как мы и просили: fixed by #3938, плюс #4280 для кредов хука; бэкпорт в release-1.6 — #4421 |
| [#4676](https://github.com/cozystack/cozystack/issues/4676) | yankawai | bug(dashboard): NATS free-form configuration fields have no editor in Form mode | **закрыт 08.10 в 23:21** мержем #4682 — через неделю после открытия. По ревью lexfrei, баг был шире NATS: касался всех free-form объектов в формах, и фикс сделан общим |
| [#4611](https://github.com/cozystack/cozystack/issues/4611) | yankawai | FoundationDB connection string is not visible to tenants in the dashboard | **закрыт 30.09** мержем #4612 — в день открытия. Хвост #4148: RBAC появился, но консоль не умела показывать ConfigMap |
| [#1966](https://github.com/cozystack/cozystack/issues/1966) | lllamnyp | tcp-balancer: HAProxy 3.3 breaks due to frontend/backend name collision | **закрыт 25.09** мержем #4366 — спустя почти восемь месяцев. Провисел с 03.02 с `priority/important-soon` и успел получить `lifecycle/stale`; первый фикс #2321 застрял на авторе, довёл задачу #4366 |
| [#3740](https://github.com/cozystack/cozystack/issues/3740) | lexfrei | networking: cilium takes its per-node infrastructure IPs from the pod CIDR kube-ovn allocates pods from | **CLOSED 08.10**, открыт был с 10.08, `triage/accepted`, `priority/important-soon`. Kube-OVN выдаёт адреса подам из всего `networking.podCIDR`, а cilium берёт по два адреса на ноду (router IP и Ingress IP) из `Node.spec.podCIDR` — по дефолтам это один диапазон, и под рано или поздно получает уже занятый адрес. **06.10 yankawai задокументировал четыре случая на наших кластерах** (v1.6.4, дефолты kubeovn-cilium, 186–212 занятых адресов на кластер): свежий postgres не поднялся — пробы kubelet попадали в `cilium_host`; хук очистки mariadb упал на `putEndpointIdInvalid` и оставил PVC; virt-operator и keycloak-operator держат адреса cilium чужих нод. Обход — адреса cilium в `excludeIps` у `ovn-default`, но его надо повторять на каждую новую ноду. Вариант `cluster-pool` из `100.65.0.0/16` ломает Gateway API между нодами: источник вне `ovn40subnets` не маскарадится. **Фикс — #4778, открыт 06.10 в 18:25**: `cluster-pool` с /29 на ноду из последнего /21 от `networking.podCIDR`, тот же диапазон уходит kube-ovn как `--default-exclude-ips`, живые кластеры переводит post-upgrade хук. lexfrei 06.10 в 19:01 запросил две правки в хуке и `!` в заголовке, в 19:16 одобрил; 07.10 апрув снялся пушем автора, 08.10 в 10:11 выдан повторно. **issue закрыт 08.10 в 18:41** мержем #4778 — спустя два месяца после открытия |
| [#4342](https://github.com/cozystack/cozystack/issues/4342) | ghostrider0470 | cozystack-api: spec defaults are applied when an Application is read but not when it is created, so its first update upgrades the Helm release | OPEN с 18.09, `triage/needs-triage`. Обобщение механизма из ревью #3936 до платформенного бага, с репродукцией на VMDisk: дефолты подставляются на чтении, `Update` начинается с чтения — первая запись любого рода запекает дефолты в `spec.values`, Flux делает незапрошенный upgrade, а смена дефолта чарта до тронутых приложений уже не доезжает. Ссылается на #3936 и #3956; для VMInstance такой upgrade ещё и виснет — его комментарий в issue #3734 называет причину: `lookup` kube-ovn IP в шаблоне vm.yaml |

## LINSTOR

| PR | Автор | Название | Состояние |
|---|---|---|---|
| [LINBIT/linstor-server#528](https://github.com/LINBIT/linstor-server/pull/528) | yankawai | satellite: wait for probe devices before reading block information | **MERGED** 08.09 в `master`, коммит `6e5557839`, вошёл в релиз v1.35.1 от 09.09. Закрывает issue LINBIT#527. **В cozystack приехал 18.09 мержем #4100** — патчем поверх пина 1.33.3 |

Пакет `packages/system/linstor` собирает `piraeus-server` **из исходников**: берёт тег из `LINSTOR_VERSION ?= 1.33.3` и накладывает патчи из `images/piraeus-server/patches/`.

| | Версия |
|---|---|
| Прибито в cozystack | **1.33.3** |
| Актуальный релиз LINSTOR | **1.35.1** от 09.09 |
| Разрыв | **448 коммитов**, два минорных релиза |

У LINSTOR **нет релизных веток** — только `master` и рабочие ветки. Бэкпортов в 1.33.x не бывает, поэтому на текущем пине патч из #4100 был единственным способом получить фикс — теперь он в дереве.

- у патча **определённое условие снятия**: при подъёме `LINSTOR_VERSION` до 1.35.1+ его надо удалить, иначе сборка образа упадёт. Механизма, который это заметит, в репозитории нет
- **запроса на бамп версии не существует**. Похожий issue #3857 — про `linstor-csi`, это другой компонент
- при бампе патчи с уже выпущенными апстрим-коммитами устареют:

| Патч | В v1.35.1 |
|---|---|
| `allow-toggle-disk-retry.diff` | да, устареет |
| `fix-luks-header-size.diff` | да, устареет |
| бэкпорт #528 из #4100 | да, устареет |
| `fix-duplicate-tcp-ports.diff` | нет |
| `retry-adjust-after-stale-bitmap.diff` | нет |
| `retry-secondary-after-mkfs.diff` | нет, апстрим-коммита вообще нет |

Коммиты трёх нижних патчей в репозитории LINBIT существуют, но в релиз не входят. `[вывод]` Висят на ветках несмерженных PR.

## Proposals

| PR | Автор | Название | Состояние |
|---|---|---|---|
| [cozystack/community#25](https://github.com/cozystack/community/pull/25) | myasnikovdaniil | [proposal] Per-cluster etcd | OPEN, `REVIEW_REQUIRED`, ноль ревью, последняя активность в треде — наша, 03.09. Наше предложение минимального API принято целиком: на CR только `etcd.replicas`. **03.09 отправили отчёт о прогоне миграции** — ответа нет. **08.10 lllamnyp открыл реализацию — draft #4817** на фазы 1 и 2, с тем же `etcd.replicas` на CR. `[вывод]` Документ теперь догоняет код: апрув proposal'а стал срочнее |

### Наш отчёт по community#25

Живой прогон миграции datastore на кластере с Talos-воркерами (CozyStack v1.6.1, Kamaji v1.6.1-rc.1, три воркера KubeVirt). Механизм Kamaji работает и быстр: копирование около 20 с, от патча до Ready около 43 с, ноль отказов на чтение за всю сессию. Три находки:

1. **Направление миграции ограничено топологией сети тенанта.** CP-поды живут в неймспейсе кластера, и OVN пропускает только к etcd предка, не соседа. При переключении на соседа apiserver умирает на таймауте, кластер заморожен на запись. Для per-cluster etcd в том же неймспейсе это не проблема, но runbook должен назвать правило достижимости.
2. **Webhook заморозки может пережить успешную миграцию.** Лизы в `kube-node-lease` из-под него выведены, поэтому ноды остаются Ready, а кластер при этом заблокирован на запись. Критерий успеха для e2e — канареечная **запись**, а не статус Ready.
3. **Kubelet'ы глохнут к новым назначениям, пока их не перезапустить.** Новый под висит `Pending` с назначенной нодой (наблюдали 16 минут). Перезапуск дешёвый: Talos делает graceful cordon, канарейка не потеряла ни одной записи на трёх ребутах.

Отдельно: у оператора нет пути к Talos API воркеров тенанта — чарт заводит per-cluster `talos-ca`, но клиентских кредов не выпускает.

## Закрытые без мержа

| PR | Автор | Что было | Итог |
|---|---|---|---|
| [#2321](https://github.com/cozystack/cozystack/pull/2321) | officialasishkumar | Первый фикс tcp-balancer для HAProxy 3.3+ | Закрыт 29.09 lexfrei как superseded: доведён смерженным #4366, где авторство переименования сохранено; в комментарии закрытия — благодарность автору |
| [#4026](https://github.com/cozystack/cozystack/pull/4026) | yankawai | Ручка `setVMMaxMapCount` для OpenSearch | Закрыт автором 17.09: main обогнал — #4152 поднимает sysctl DaemonSet'ом, #2682 выключил `setVMMaxMapCount`. Ребейз вернул бы дефолт `true`, обратный смерженному |
| [#3357](https://github.com/cozystack/cozystack/pull/3357) | fuad00 | DNS_BASE для opensearch-operator | Закрыт 10.09: коммит с сохранением авторства довели через #4185, маршрут изменён на `opensearch-operator.manager.dnsBase` |
| [#3294](https://github.com/cozystack/cozystack/pull/3294) | myasnikovdaniil | Golden Talos image, первая версия | Смержен 07.09 в ветку `test/drop-ghcr-mirror`, до main не доехал. Работа доведена в #4171 — смержен 24.09 |
| [#4007](https://github.com/cozystack/cozystack/pull/4007) | myasnikovdaniil | Удаление ghcr.io pull-through mirror из e2e | Закрыт 09.09 как пустой: #4020 уже всё удалил |
| [#3949](https://github.com/cozystack/cozystack/pull/3949) | IvanHunters | Поля `talos.*` необязательными в схеме | Закрыт 24.08 |
| [#2751](https://github.com/cozystack/cozystack/pull/2751) | SerjioTT | SMTP для Grafana | Закрыт 13.08 stale-ботом, осознанно: #3800 решает задачу на правильном слое |

## Смержено в main

PR из отслеживаемого скоупа, которые уже в `main` своего репозитория. Часть из них подробнее описана выше.

**28.09 вышел патч-релиз [v1.6.4](https://github.com/cozystack/cozystack/releases/tag/v1.6.4)** — 19 отслеживаемых PR доехали до пользователей release-1.6 бэкпортами (с учётом добавленного задним числом #4220), плюс website#697 в notes; всего в notes релиза 60 пунктов. В контрибьюторах yankawai отмечен как «First contribution». #4366 и #4171 в notes отсутствуют — `[вывод]` смержены в main позже отсечки и поедут следующим релизом. #2682, #4185, #4152 и #4095 не входят ни в один релиз 1.6.x — ждут минорного.

Нового стабильного релиза после v1.6.4 нет. 29.09 вышел pre-release v1.7.0-alpha.3. Девять открытых ручных бэкпортов — #4516, #4614, #4677, #4683, #4684, #4725, #4739, #4740, #4780 — `[вывод]` поедут следующим патч-релизом 1.6.x, когда их отревьюят и прогонят.

**08.10 в main уехали последние семь наших открытых PR** — #4812, #4789, #4778, #4779, #4724, #4766, #4682, все смержены lexfrei с зелёными `pre-commit` и `E2E Tests`. Бэкпортов в release-1.6 у четырёх из них ещё нет (#4725 и #4780 были поданы заранее, для #4778 `[предположение]` бэкпорта не будет — `kind/breaking-change`).

| PR | Автор | Что решил | Смержен | Релиз |
|---|---|---|---|---|
| [#4682](https://github.com/cozystack/cozystack/pull/4682) | yankawai | dashboard: у free-form объектов в Form-режиме появился JSON-редактор с сохранением типов и блокировкой submit при невалидном вводе. Это не только NATS `config.merge`: то же было с `addons.*.valuesOverride` у Kubernetes, Kafka `topics[].config`, `talos.registryMirrors`, etcd `affinity`. Закрыл issue #4676 | 08.10 в 23:21 | бэкпорта в release-1.6 ещё нет |
| [#4766](https://github.com/cozystack/cozystack/pull/4766) | yankawai | openbao: opt-in `seal.type: static` — тенантный OpenBAO больше не ждёт ручного `bao operator unseal` после каждого рестарта, HA-поды не зависают на старте (лейбл `allow-to-apiserver`). Ключ в Secret, который создаёт администратор кластера; чарт его не генерирует, молчаливая смена seal заблокирована. Продолжение застрявшего #4168, первый коммит — работа txmazing. Решает issue #2483 и issue #2793, но оба остались открытыми — в теле PR нет закрывающего слова | 08.10 в 23:17 | бэкпорта в release-1.6 ещё нет |
| [#4724](https://github.com/cozystack/cozystack/pull/4724) | yankawai | postgres: `shared_buffers` ниже лимита 512Mi ставится в четверть лимита, но не больше memory request — PostgreSQL больше не стартует со встроенными 128MB, которые на `t1.nano` занимали весь лимит памяти. Явный `shared_buffers` побеждает | 08.10 в 21:10 | ручной бэкпорт #4725 открыт 02.10, ревью нет |
| [#4779](https://github.com/cozystack/cozystack/pull/4779) | yankawai | mongodb: установка и апгрейд с `bootstrap.enabled: true` больше не падают в helm — бакет и префикс для `PerconaServerMongoDBRestore` берутся из `backup.destinationPath` и квотируются. Багу было столько же лет, сколько чарту | 08.10 в 20:54 | ручной бэкпорт #4780 открыт 06.10, ревью нет |
| [#4778](https://github.com/cozystack/cozystack/pull/4778) | yankawai | cilium: на вариантах с kube-ovn cilium берёт свой router IP и Ingress IP не из pod CIDR, откуда kube-ovn выдаёт адреса подам, а из выделенного блока — `ipam.mode: cluster-pool` с /29 на ноду из последнего /21 от `networking.podCIDR`, и тот же диапазон уходит kube-ovn как `--default-exclude-ips`. Блок внутри pod CIDR намеренно: с пулом `100.65.0.0/16` nodeport через Gateway API отвечал 200 только на ноде бэкенда (3/9), с блоком из pod CIDR — 9/9. Живые кластеры переводит post-upgrade хук с проверками ёмкости и вложенности до любых изменений. Закрыл issue #3740. **Ломающее**: `networking.podCIDR` на этих вариантах теперь не уже /20, потолок — 256 нод. 09.10 в 22:34 yrahman82 подтвердил ту же коллизию на свежей v1.6.4 (три ноды, kubeovn-cilium): под получил `CiliumInternalIP` ноды и потерял доступ ко всем портам её хоста, включая apiserver и node-exporter | 08.10 в 18:41 | `[предположение]` бэкпорта не будет — `kind/breaking-change`. 09.10 yrahman82 спросил прямо, цель v1.6 или v1.7 — ответить надо нам |
| [#4789](https://github.com/cozystack/cozystack/pull/4789) | yankawai | dashboard: 403 или 404 на resource map сбрасывает имена ConfigMap независимо от того, было ли удачное чтение, — вкладка ConfigMaps больше не остаётся с прежними именами и ошибкой доступа на каждом из них. Хвост #4612 | 08.10 в 13:50 | бэкпорта ещё нет; `[вывод]` подавать после мержа #4684 |
| [#4812](https://github.com/cozystack/cozystack/pull/4812) | yankawai | monitoring: guard severity алертов больше не пропускает источники правил — поиск ловит закавыченный и закомментированный `kind`, а документ без групп и группа, пустая после вырезания template-действий, валят прогон с именем файла и группы. Хвост #3799, на текущем дереве поведение не меняется | 08.10 в 11:14 | бэкпорта ещё нет; `[вывод]` подавать после мержа #4740 |
| [#3956](https://github.com/cozystack/cozystack/pull/3956) | myasnikovdaniil | api: пустое обязательное поле в спеке чинится вместо падения всего релиза; nulls без применимых дефолтов чарта сохраняются. Шов на write-пути для create-time дефолтинга из issue #3950. Запрос IvanHunters от 01.09 висел месяц, снят им самим 30.09 | 07.10 | пока только main |
| [#4675](https://github.com/cozystack/cozystack/pull/4675) | yankawai | nats: JetStream включается для сгенерированного аккаунта `A`, когда он включён на сервере и нет явной настройки — `$JS.API.INFO` больше не отвечает ошибкой 10039 | 05.10 | ручной бэкпорт #4677 открыт 01.10, ревью нет |
| [#4738](https://github.com/cozystack/cozystack/pull/4738) | yankawai | platform: миграция 50 оставляет легаси etcd-оператор в нуле после adoption — тенантные etcd больше не теряют DNS пиров при апгрейде на 1.6.x. Найдено на проде при апгрейде 1.4.3 → 1.6.4 | 05.10 | ручной бэкпорт #4739 открыт 03.10, ревью нет |
| [#3800](https://github.com/cozystack/cozystack/pull/3800) | yankawai | monitoring: почтовый канал алертинга через Alertmanager рядом с Alerta — реализовано наше предложение (`alertnames`, настраиваемые `severities`); пароль SMTP только из константного Secret, адреса и `smarthost` валидируются строже `net/mail`. Пять кругов ревью 18–30.09 | 30.09 | бэкпорт в release-1.6 в пути: ботовский #4609 конфликтнул (draft с маркерами), чистый ручной #4614 с 30.09 ждёт ревью |
| [#4612](https://github.com/cozystack/cozystack/pull/4612) | yankawai | dashboard + foundationdb: в консоли у приложения появилась вкладка ConfigMaps — строка подключения FoundationDB теперь видна тенанту в дашборде. #4148 дал RBAC на ConfigMap, но у консоли не было представления для него; вкладка generic, но resource map пока только у foundationdb. Закрыл issue #4611 в день открытия | 30.09 | ручной бэкпорт #4684 открыт 01.10, ревью нет |
| [#4610](https://github.com/cozystack/cozystack/pull/4610) | yankawai | cozystack-api: сборщик мусора может финализировать Application — удаление с `--cascade=foreground` или orphan-пропагацией больше не зависает навсегда. Application теперь отдаёт `foregroundDeletion`/`orphan` в `metadata.finalizers` и проносит их изменения до HelmRelease; у пары общий UID, и GC финализировал узел через Application-эндпоинт, который finalizers не показывал. Найдено на v1.6.4 с Bucket и OpenBAO | 30.09 | ручной бэкпорт #4683 открыт 01.10, ревью нет |
| [#3799](https://github.com/cozystack/cozystack/pull/3799) | yankawai | linstor + keda: severity `warning`/`informational` вместо `warn`/`info` — восемь алертов больше не отбрасываются Alerta с ошибкой 500; severity закреплены тестом-контрактом по helm-шаблонам | 30.09 | ручной бэкпорт #4740 открыт 04.10, ревью нет |
| [#4366](https://github.com/cozystack/cozystack/pull/4366) | yankawai | tcp-balancer: бэкенды переименованы (rename взят из #2321, автор указан), образ запинен на `haproxy:3.4.4` LTS вместо `latest`, whitelist-guard, helm-unittest сьюты. Свежая установка снова стартует. Закрыл issue #1966, висевший с 03.02 | 25.09 | ручной бэкпорт #4516 открыт 26.09, ревью нет |
| [#4171](https://github.com/cozystack/cozystack/pull/4171) | myasnikovdaniil | kubernetes: воркеры тенантных кластеров грузятся CDI-клоном общего golden-образа Talos вместо ~4 GiB по HTTP на каждого; выбор per-pool через `osImage.builtin` | 24.09 | пока только main |
| [#3936](https://github.com/cozystack/cozystack/pull/3936) | yankawai | rabbitmq: дефолтный пресет поднят с `t1.nano` до `s1.nano` — брокер на дефолтах больше не получает OOMKill. Популяция, остающаяся на `t1.nano` после read-modify-write, описана в release note (см. issue #4342) | 22.09 | v1.6.4 — бэкпорт #4393 |
| [#4291](https://github.com/cozystack/cozystack/pull/4291) | yankawai | cluster-api: бэкпорт живой миграции перед выводом хоста — CAPK мигрирует VM, а не удаляет (CAPK #374, #389, #392) | 18.09 | v1.6.4 — бэкпорт #4474 |
| [#3937](https://github.com/cozystack/cozystack/pull/3937) | yankawai | registry: `kubectl apply --dry-run=server` больше не делает настоящую запись, коды ошибок бэкенда сохраняются | 18.09 | v1.6.4 — бэкпорт #4370 |
| [#4220](https://github.com/cozystack/cozystack/pull/4220) | yankawai | backups: у barman-cloud сайдкара свои ресурсы (100m/256Mi, потолок 1Gi) — раньше в тенантных неймспейсах он наследовал 128Mi из `LimitRange` и получал OOMKill во время бэкапов, пока `ObjectStore` и `Cluster` выглядели здоровыми. Взят в трекинг задним числом 30.09 | 18.09 | v1.6.4 — бэкпорт #4336 |
| [#4100](https://github.com/cozystack/cozystack/pull/4100) | yankawai | linstor: ожидание временных probe-устройств — тома на ZFS 4K-пулах снова набирают реплики (бэкпорт LINBIT#528) | 18.09 | v1.6.4 — бэкпорт #4335 |
| [#3935](https://github.com/cozystack/cozystack/pull/3935) | yankawai | kafka: `topics[].config` необязателен, как и у Strimzi | 18.09 | v1.6.4 — бэкпорт #4334 |
| [#4014](https://github.com/cozystack/cozystack/pull/4014) | yankawai | mongodb: креды дашборда заполняются при первой установке, живой пароль не ротируется | 18.09 | v1.6.4 — бэкпорт #4330 |
| [#4136](https://github.com/cozystack/cozystack/pull/4136) | yankawai | opensearch: post-delete хук удаляет PVC данных — квота тенанта освобождается. Закрыл issue #4086 | 18.09 | v1.6.4 — бэкпорт #4368 |
| [#4253](https://github.com/cozystack/cozystack/pull/4253) | yankawai | kubeovn: ретрай настройки сети миграции — живая миграция VM не зависает | 18.09 | v1.6.4 — бэкпорт #4328 |
| [#4135](https://github.com/cozystack/cozystack/pull/4135) | yankawai | apps: хуки очистки harbor, clickhouse, mariadb и qdrant получили `watch` и лимит ожидания — рапортуют о завершении после реального удаления PVC. Закрыл issue #4085 | 18.09 | v1.6.4 — бэкпорт #4369 |
| [#4134](https://github.com/cozystack/cozystack/pull/4134) | yankawai | nats: `config.merge.accounts` сливается со сгенерированной картой аккаунтов, установка с `users` не падает. Закрыл issue #4084 | 18.09 | v1.6.4 — бэкпорт #4321 |
| [#4292](https://github.com/cozystack/cozystack/pull/4292) | yankawai | linstor: опциональный graceful shutdown сателлита на Talos — Secondary-ресурсы DRBD отпускаются при штатном выключении ноды | 18.09 | v1.6.4 — бэкпорт #4401 |
| [#4254](https://github.com/cozystack/cozystack/pull/4254) | yankawai | kubevirt: платформенный ключ `kubevirt.migrations` доезжает до `spec.configuration.migrations` | 18.09 | v1.6.4 — бэкпорт #4402 |
| [#4184](https://github.com/cozystack/cozystack/pull/4184) | yankawai | linstor: plunger переподключает только зависший peer — отключённая реплика возвращается | 18.09 | v1.6.4 — бэкпорт #4319 |
| [#4148](https://github.com/cozystack/cozystack/pull/4148) | yankawai | foundationdb: тенант читает ConfigMap со строкой подключения своей базы | 18.09 | v1.6.4 — бэкпорт #4315 |
| [#4133](https://github.com/cozystack/cozystack/pull/4133) | yankawai | clickhouse: Keeper создаётся и при имени приложения длиннее 15 символов. Закрыл issue #4083 | 17.09 | v1.6.4 — бэкпорт #4314 |
| [#3934](https://github.com/cozystack/cozystack/pull/3934) | yankawai | kubernetes: сайдкары KubeVirt CSI запинены на версии SIG Storage — контроллер CSI в тенанте больше не падает на CPU без x86-64-v3 | 17.09 | v1.6.4 — бэкпорт #4313 |
| [website#697](https://github.com/cozystack/website/pull/697) | yankawai | Документация `kubevirt.migrations` в разделе `next` сайта | 17.09 | в notes v1.6.4 |
| [#3938](https://github.com/cozystack/cozystack/pull/3938) | yankawai | kafka: топики релиза удаляются до снятия topic operator, переустановка больше не падает. Закрыл issue #3793 (23.09, вместе с #4280) | 15.09 | v1.6.4 — бэкпорт #4456, вместе с #4280 |
| [#2682](https://github.com/cozystack/cozystack/pull/2682) | Arsolitt | opensearch: TLS для HTTP API и Dashboards через cert-manager, `setVMMaxMapCount` выключен | 11.09 | пока только main |
| [#4185](https://github.com/cozystack/cozystack/pull/4185) | lexfrei | opensearch-operator: `DNS_BASE` следует домену платформы — securityadmin на `cozy.local` работает | 10.09 | пока только main |
| [#4152](https://github.com/cozystack/cozystack/pull/4152) | lexfrei | opensearch-operator: DaemonSet поднимает `vm.max_map_count` на всех нодах, privileged init больше не нужен | 08.09 | пока только main |
| [#3920](https://github.com/cozystack/cozystack/pull/3920) | lexfrei | CDI v1.66.1: HTTP-импорты в block-тома на 4Kn снова работают, нет шторма `resourceVersion` на prime-PVC | 07.09 | v1.6.4 — бэкпорт #4156 |
| [#4095](https://github.com/cozystack/cozystack/pull/4095) | IvanHunters | harbor: явные ресурсы для nginx-прокси, переживает LimitRange тенанта | 06.09 | пока только main |
