# Отслеживание PR в cozystack

Обновлено: **2026-10-08**. Репозиторий по умолчанию — `cozystack/cozystack`, иначе указано явно.

На GitHub у issue и PR **общая нумерация**, по номеру их не отличить. Поэтому здесь issue всегда помечены словом — `issue #4083`, а голый `#4133` означает PR.

## Главное за 07–08.10

- **Ребейз больше не нужен: 07.10 в 17:13–17:14 yankawai закрыл и переоткрыл #4724, #4682, #4766 и #4779.** Переоткрытие пересобирает тестовый merge-коммит на свежем main, не трогая голову PR, поэтому апрувы lexfrei остались на месте. Проверено: у всех четырёх тестовый merge-коммит создан 07.10 в 17:13–17:14 с базой `cbc9d0c4` — это main от 07.10, уже содержащий #4768, который починил сборку тестов `internal/backupcontroller`. Контракт-тест `internal/fluxcontract` починен в main ещё 05.10
- **Теперь всё держит одна кнопка.** Новые прогоны на этих четырёх PR стоят в `action_required` — ждут «Approve and run workflows». Ни одна обязательная проверка (`pre-commit`, `E2E Tests`) на новой базе ещё не запускалась
- **Побочный эффект переоткрытия: снялся auto-merge.** lexfrei включал его на #4724 и #4682 06.10 в 13:17, на #4766 — 06.10 в 19:06; закрытие PR его отключило. `[вывод]` После зелёного прогона auto-merge надо включить заново или мержить руками
- **#4778 — обещанный PR по issue #3740, открыт 06.10 в 18:25** (`kind/breaking-change`, size/XXL): на вариантах с kube-ovn cilium переводится на `ipam.mode: cluster-pool` и берёт по /29 на ноду из последнего /21 от `networking.podCIDR`, а платформа отдаёт тот же диапазон kube-ovn как `--default-exclude-ips`. Блок намеренно внутри pod CIDR: с пулом `100.65.0.0/16` nodeport через Gateway API отвечал 200 только на ноде бэкенда (3/9), с блоком из pod CIDR — 9/9. Для живых кластеров есть post-upgrade хук миграции. lexfrei 06.10 в 19:01 запросил две правки в хуке и `!` в заголовке, yankawai закрыл их в 19:11, lexfrei одобрил в 19:16. **Апрув снят 07.10 в 17:11 собственным коммитом автора** — он догружает `test_helper` в bats-сьют, потому что после #3849 main гоняет `hack/*.bats` под настоящим bats со `set -u`. Нужен повторный апрув и запуск CI
- **#4779 и его бэкпорт #4780 — новые в трекинге** (открыты 06.10): любая установка или апгрейд mongodb с `bootstrap.enabled: true` падает в helm, потому что чарт рендерит `PerconaServerMongoDBRestore` без обязательного `spec.backupSource.s3.bucket`. APPROVED lexfrei 06.10 в 19:18
- **#4789 и #4812 — новые в трекинге** (открыты 07.10), хвосты уже смерженного: #4789 убирает вкладку ConfigMaps, когда resource map стала нечитаемой после удачного чтения (follow-up #4612), #4812 закрывает две слепые зоны guard'а severity алертов (follow-up #3799). Ревью пока нет ни у одного
- **Ручные бэкпорты есть у всего, что просили пометить `kind/backport`.** В release-1.6 открыто девять ручных бэкпортов yankawai: #4516, #4614, #4677, #4683, #4684, #4725, #4739, #4740, #4780. У всех ревью нет и CI в `action_required`. Прошлая просьба повесить `kind/backport` на #3799, #4738 и #4675 была ошибочной — у них уже есть #4740, #4739 и #4677, лейбл дал бы дубль
- **Без изменений**: ботовский #4609 (конфликтный, закрывается), community#25 без движения с 03.09, issue #3950, issue #3022, issue #3740, issue #4258, issue #4342. Нового стабильного релиза после v1.6.4 (28.09) нет; 29.09 вышел pre-release v1.7.0-alpha.3

## Требует действия

Всё в таблице — действия мейнтейнеров.

| Что | Где | Кому и что делать |
|---|---|---|
| **«Approve and run workflows»** | #4724, #4682, #4766, #4779 | Мейнтейнер: все четыре одобрены lexfrei, 07.10 переоткрыты автором — тестовый merge-коммит уже на main с обоими фиксами. Прогоны висят в `action_required`, ребейз не нужен. `[вывод]` Auto-merge снялся при закрытии, после зелёного его надо включить заново |
| **Повторный апрув и запуск CI** | #4778 | Мейнтейнер: cilium берёт адреса из блока, который kube-ovn не выдаёт — фикс issue #3740 на наших трёх кластерах. lexfrei одобрил 06.10 в 19:16, апрув снят коммитом с догрузкой `test_helper` 07.10 в 17:11. Ломающее изменение: `networking.podCIDR` на вариантах с kube-ovn теперь не уже /20 |
| **Ревью и запуск CI бэкпортов** | #4516, #4614, #4677, #4683, #4684, #4725, #4739, #4740, #4780 | Мейнтейнер: девять ручных бэкпортов в release-1.6, у всех ревью нет и CI не стартовал. Ботовский #4609 дублирует #4614 конфликтным draft'ом — закрыть. `kind/backport` на оригиналы не вешать: ручные бэкпорты уже есть, будет дубль |
| **Ревью** | #4789, #4812 | Мейнтейнер: хвосты смерженных #4612 и #3799, ревью нет, CI в `action_required` |
| **Ревью proposal** | community#25 | Любой мейнтейнер: второй драфт с 21.08 без единого ревью, наш отчёт о прогоне миграции с 03.09 без ответа |
| **Закрыть issue** | issue #3022 | Мейнтейнер: по словам автора #4026 решён мержем #4152 и #2682 |

**В одобренные PR ничего не пушить** — новый коммит снимет апрув (`dismiss_stale_reviews_on_push: true`). Именно так 07.10 снялся апрув на #4778. Переоткрытие PR апрув не снимает и даёт свежую базу — на #4724, #4682, #4766 и #4779 это проверено.

## Наши PR — открытые

| PR | Автор | Название | Что решает | Состояние |
|---|---|---|---|---|
| [#4778](https://github.com/cozystack/cozystack/pull/4778) | yankawai | fix(cilium)!: take cilium's own addresses from a block kube-ovn does not hand out | На вариантах с kube-ovn cilium не выдаёт адреса подам, но сам берёт router IP (`cilium_host`) и Ingress IP для Gateway API из своего CIDR — при `ipam.mode: kubernetes` это `Node.spec.podCIDR`, то есть /24 из того же /16, откуда kube-ovn выдаёт подам. Рано или поздно под получает адрес, который уже держит cilium. PR переводит эти варианты на `ipam.mode: cluster-pool` с /29 на ноду из последнего /21 от `networking.podCIDR` и отдаёт тот же диапазон kube-ovn как `--default-exclude-ips`. Блок намеренно внутри pod CIDR: kube-ovn маскарадит трафик с хоста только если источник в `ovn40subnets`, поэтому с пулом `100.65.0.0/16` nodeport через Gateway API отвечал 200 лишь на ноде бэкенда — 3/9 против 9/9. Живые кластеры переводит post-upgrade хук, который до любых изменений проверяет ёмкость пула по числу CiliumNode и что блок лежит внутри дефолтной подсети. `fixes issue #3740` | **REVIEW_REQUIRED** — апрув был и снялся. lexfrei 06.10 в 19:01 запросил правки: хук чистил CIDR ноды до проверки ёмкости (на 257-й ноде агент терял сеть подов), пул вне дефолтной подсети всё равно переводил агентов, и нужен `!` в заголовке. yankawai закрыл всё в 19:11, lexfrei одобрил 06.10 в 19:16. **07.10 в 17:11 апрув снят** собственным коммитом автора, догружающим `test_helper` в bats-сьют: после #3849 main гоняет `hack/*.bats` под настоящим bats со `set -u`. Обязательные проверки на этой голове не стартовали — `action_required`. Ломающее: `networking.podCIDR` на вариантах с kube-ovn теперь не уже /20, и потолок таких вариантов — 256 нод |
| [#4766](https://github.com/cozystack/cozystack/pull/4766) | yankawai | feat(openbao): optional static-key auto-unseal and apiserver egress label | Тенантный OpenBAO поднимается запечатанным и после каждого рестарта ждёт ручного `bao operator unseal`, а HA-поды зависают на старте без egress к apiserver. PR добавляет opt-in `seal.type: static` (ключ в Secret, который создаёт администратор кластера; чарт ключ не генерирует), защиту от молчаливой смены seal, лейбл `allow-to-apiserver` и ограничения на значения: `keyId` уходит в `tpl` внутри helm-controller с правами cluster-admin. Продолжение застрявшего #4168: первый коммит — его работа, автор txmazing | **APPROVED** — lexfrei 06.10 в 15:07 и повторно в 19:06 после доработки документации; auto-merge включался 06.10 в 19:06 и снялся при переоткрытии. 07.10 в 17:13 автор закрыл и переоткрыл PR: тестовый merge-коммит пересобран на main `cbc9d0c4`, где сборка тестов `internal/backupcontroller` уже починена #4768. Прежний красный был от неё, PR её не трогает. Прогон ждёт «Approve and run workflows». Закроет issue #2483 и issue #2793 |
| [#4724](https://github.com/cozystack/cozystack/pull/4724) | yankawai | fix(postgres): cap the default shared_buffers at a quarter of the memory limit | Чарт не задаёт `shared_buffers`, и CNPG не задаёт — PostgreSQL стартует со встроенными 128MB: на `t1.nano` это весь лимит памяти, на дефолтном `t1.micro` — половина. Дамп, большой скан или догоняющая реплика заполняют пул — инстанс получает OOMKill. Ниже лимита 512Mi чарт ставит четверть лимита, но не больше memory request; явный `shared_buffers` побеждает | **APPROVED** — lexfrei 06.10 в 13:14, после того как правка от 05.10 ограничила `shared_buffers` memory request: иначе вебхук CloudNativePG отвергал бы Cluster при `memory-allocation-ratio` выше 4. auto-merge включался 06.10 в 13:17 и снялся при переоткрытии. 07.10 в 17:13 автор закрыл и переоткрыл PR — база merge-коммита теперь main `cbc9d0c4`, где контракт-тест `internal/fluxcontract` починен ещё 05.10. Прогон ждёт одобрения. Бэкпорт #4725 синхронизирован 06.10 вечером |
| [#4682](https://github.com/cozystack/cozystack/pull/4682) | yankawai | fix(dashboard): allow editing free-form objects in application forms | В Form-режиме консоли у free-form полей нет редактора — они рендерятся пустым fieldset. По словам lexfrei, это не только NATS `config.merge`: то же с десятью `addons.*.valuesOverride` у Kubernetes, Kafka `topics[].config`, `talos.registryMirrors`, etcd `affinity`. PR добавляет JSON-редактор с сохранением типов и блокировкой submit при невалидном вводе. `Fixes issue #4676` | **APPROVED** — lexfrei 06.10 в 13:14, после добавленных скриншотов; auto-merge включался 06.10 в 13:17 и снялся при переоткрытии. 07.10 в 17:13 автор закрыл и переоткрыл PR — база свежая, прежний красный был от уже починенного `internal/fluxcontract`. Прогон ждёт одобрения. CodeRabbit оставил minor про валидацию перед переключением в YAML |
| [#4779](https://github.com/cozystack/cozystack/pull/4779) | yankawai | fix(mongodb): set the s3 bucket and prefix on the bootstrap restore | Любая установка или апгрейд mongodb с `bootstrap.enabled: true` падает в helm: чарт рендерит `PerconaServerMongoDBRestore` без `spec.backupSource.s3.bucket`, а CRD оператора это поле требует — apiserver отвергает объект. Бакет не единственная дыра: при restore из `backupSource` оператор 1.22.0 собирает pbm-хранилище только из `backupSource.s3`, а из `destination` берёт лишь имя бэкапа, поэтому без префикса искал бы бэкап в корне бакета. PR достаёт бакет и префикс из `backup.destinationPath` теми же выражениями, что уже использует хранилище бэкапов, и квотирует значения — числовой бакет вроде `s3://2024/10` раньше рендерился int'ом. Багу столько же лет, сколько чарту | **APPROVED** — lexfrei 06.10 в 19:18. 07.10 в 17:14 автор закрыл и переоткрыл PR; прежний красный подтверждён логом прогона — `FAIL github.com/cozystack/cozystack/internal/backupcontroller [build failed]`, то есть поломка main, починенная #4768. Прогон на свежей базе ждёт одобрения. Бэкпорт — #4780 |
| [#4789](https://github.com/cozystack/cozystack/pull/4789) | yankawai | fix(dashboard): drop the resource map ConfigMaps once the map is unreadable | Хвост #4612. Вкладка ConfigMaps берёт имена из resource map приложения, а React Query держит последний удачный список, когда refetch падает: после того как map стала запрещённой или исчезла, вкладка оставалась с прежними именами, и на каждом ConfigMap показывалась ошибка доступа. Теперь 403 или 404 на resource map сбрасывает имена независимо от того, было ли удачное чтение, и вкладка исчезает — как у приложения, у которого map не читалась с самого начала. Прочие ошибки по-прежнему видны на вкладке | **REVIEW_REQUIRED** с 07.10 08:02, ревью нет. Обязательные проверки в `action_required`, не стартовали. Четыре новых теста падают на main и проходят здесь; `pnpm typecheck`, `pnpm test` (451 тест) и `pnpm lint` зелёные |
| [#4812](https://github.com/cozystack/cozystack/pull/4812) | yankawai | test(monitoring): stop the alert severity guard from missing rule sources | Хвост #3799, который добавил `hack/alert-severity-contract.bats`. У guard'а было две слепые зоны: файлы он искал по точной строке `kind: PrometheusRule`, поэтому закавыченный `kind` или `kind` с комментарием выводил весь файл из проверки; а группа, все правила которой приходят из template-действий (например, include хелпера), после вырезания действий оказывается пустой и проходила, ничего не проверив. Теперь поиск ловит и закавыченный, и закомментированный `kind`, а документ без групп и группа, в которой после вырезания не осталось ни alert-, ни recording-правил, валят прогон с именем файла и группы. На текущем дереве поведение не меняется: те же 43 файла, пустых групп нет | **REVIEW_REQUIRED** с 07.10 17:24, ревью нет. Обязательные проверки в `action_required`. Против старого guard'а два из трёх новых кейсов падают |

### Бэкпорты в release-1.6 — открытые

Девять ручных бэкпортов yankawai плюс один ботовский. У всех ручных ревью нет, и ни на одном обязательные проверки не стартовали — прогоны стоят в `action_required`. `[вывод]` Это следующий патч-релиз 1.6.x: v1.6.4 от 28.09 их не содержит.

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

| PR | Автор | Название | Что решает | Состояние |
|---|---|---|---|---|

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
| только «Approve and run workflows» — одобрены, база свежая после переоткрытия | #4724, #4682, #4766, #4779 |
| повторный апрув после снятого пушем, плюс запуск CI | #4778 |
| ревью и запуск CI | #4789, #4812 |
| ревью и запуск CI бэкпортов в release-1.6 | #4516, #4614, #4677, #4683, #4684, #4725, #4739, #4740, #4780 |

## Связь PR и issue

`Fixes #NNNN` в теле PR — стандартный синтаксис GitHub: при мерже указанный issue закрывается автоматически. Четыре issue #4083–#4086 завёл yankawai 05.09, 07.09 выкатил на них PR — **18.09 закрыт последний из четырёх**.

| Issue | Название | PR | Состояние |
|---|---|---|---|
| [#4083](https://github.com/cozystack/cozystack/issues/4083) | clickhouse: Keeper is never created when the application name is longer than 15 characters | #4133 | **закрыт** мержем 17.09 |
| [#4084](https://github.com/cozystack/cozystack/issues/4084) | nats: config.merge.accounts duplicates the generated accounts key and breaks the install | #4134 | **закрыт** мержем 18.09 |
| [#4085](https://github.com/cozystack/cozystack/issues/4085) | harbor/clickhouse: cleanup hook Role lacks watch on persistentvolumeclaims | #4135 | **закрыт** мержем 18.09 |
| [#4086](https://github.com/cozystack/cozystack/issues/4086) | opensearch: data PVCs are left behind after the application is removed | #4136 | **закрыт** мержем 18.09 |
| [#2483](https://github.com/cozystack/cozystack/issues/2483) | OpenBAO: can't init/unseal without kubectl exec | #4766 | открыт — закроется мержем |
| [#2793](https://github.com/cozystack/cozystack/issues/2793) | OpenBAO HA (raft) pods hang at startup due to hardcoded service_registration "kubernetes" | #4766 | открыт — закроется мержем |
| [#3740](https://github.com/cozystack/cozystack/issues/3740) | networking: cilium takes its per-node infrastructure IPs from the pod CIDR kube-ovn allocates pods from | #4778 | открыт — `fixes` в теле PR, закроется мержем |
| [#4676](https://github.com/cozystack/cozystack/issues/4676) | bug(dashboard): NATS free-form configuration fields have no editor in Form mode | #4682 | открыт — `Fixes` в теле PR, закроется мержем |

## Issues

| Issue | Автор | Название | Состояние |
|---|---|---|---|
| [#3950](https://github.com/cozystack/cozystack/issues/3950) | IvanHunters | Platform-wide defaults for tenant Talos worker settings | OPEN, **23.09 lexfrei принял в работу**: `triage/accepted` и `priority/important-longterm` вместо `triage/needs-triage`. Смерженный 24.09 #4171 закрыл каталожную сторону — платформенный ключ `kubernetesWorkerImage` позволяет направить импорт golden-образа в зеркало, — но потребляющая сторона дефолтов (`imageFactoryURL`, `installerRepository`, `registryMirrors` на `KubernetesNodes`) осталась. В треде готовый разбор: наш комментарий 07.09 о create-time дефолтинге, ответ myasnikovdaniil 08.09 (двум полям платформенный дефолт безопасен, `imageFactoryURL` — прокат флота), предупреждение lexfrei 01.09 о невозможности plain default chain в Helm. Шов на write-пути — #3956, **смержен 07.10** |
| [#3022](https://github.com/cozystack/cozystack/issues/3022) | lexfrei | OpenSearch fails to start in tenant namespaces: privileged init-sysctl violates baseline PodSecurity | OPEN, но **по сути решён**: #4152 поднимает `vm.max_map_count` DaemonSet'ом, #2682 выключил `setVMMaxMapCount`. Автор #4026 предложил закрыть |
| [#4073](https://github.com/cozystack/cozystack/issues/4073) | lexfrei | opensearch-operator: dnsBase stays cluster.local | **закрыт** 17.09 — фикс в #4185 |
| [#3793](https://github.com/cozystack/cozystack/issues/3793) | IvanHunters | Deleting a tenant with a Kafka app hangs the namespace in Terminating (KafkaTopic strimzi.io/topic-operator finalizer) | **закрыт** 23.09 lexfrei — как мы и просили: fixed by #3938, плюс #4280 для кредов хука; бэкпорт в release-1.6 — #4421 |
| [#4676](https://github.com/cozystack/cozystack/issues/4676) | yankawai | bug(dashboard): NATS free-form configuration fields have no editor in Form mode | OPEN с 01.10, `triage/needs-triage`. Фикс — #4682, одобрен lexfrei 06.10 и ждёт только запуска CI. По ревью lexfrei, баг шире NATS — касается всех free-form объектов в формах |
| [#4611](https://github.com/cozystack/cozystack/issues/4611) | yankawai | FoundationDB connection string is not visible to tenants in the dashboard | **закрыт 30.09** мержем #4612 — в день открытия. Хвост #4148: RBAC появился, но консоль не умела показывать ConfigMap |
| [#1966](https://github.com/cozystack/cozystack/issues/1966) | lllamnyp | tcp-balancer: HAProxy 3.3 breaks due to frontend/backend name collision | **закрыт 25.09** мержем #4366 — спустя почти восемь месяцев. Провисел с 03.02 с `priority/important-soon` и успел получить `lifecycle/stale`; первый фикс #2321 застрял на авторе, довёл задачу #4366 |
| [#3740](https://github.com/cozystack/cozystack/issues/3740) | lexfrei | networking: cilium takes its per-node infrastructure IPs from the pod CIDR kube-ovn allocates pods from | OPEN с 10.08, `triage/accepted`, `priority/important-soon`. Kube-OVN выдаёт адреса подам из всего `networking.podCIDR`, а cilium берёт по два адреса на ноду (router IP и Ingress IP) из `Node.spec.podCIDR` — по дефолтам это один диапазон, и под рано или поздно получает уже занятый адрес. **06.10 yankawai задокументировал четыре случая на наших кластерах** (v1.6.4, дефолты kubeovn-cilium, 186–212 занятых адресов на кластер): свежий postgres не поднялся — пробы kubelet попадали в `cilium_host`; хук очистки mariadb упал на `putEndpointIdInvalid` и оставил PVC; virt-operator и keycloak-operator держат адреса cilium чужих нод. Обход — адреса cilium в `excludeIps` у `ovn-default`, но его надо повторять на каждую новую ноду. Вариант `cluster-pool` из `100.65.0.0/16` ломает Gateway API между нодами: источник вне `ovn40subnets` не маскарадится. **Фикс — #4778, открыт 06.10 в 18:25**: `cluster-pool` с /29 на ноду из последнего /21 от `networking.podCIDR`, тот же диапазон уходит kube-ovn как `--default-exclude-ips`, живые кластеры переводит post-upgrade хук. lexfrei 06.10 в 19:01 запросил две правки в хуке и `!` в заголовке, в 19:16 одобрил; 07.10 апрув снялся пушем автора |
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
| [cozystack/community#25](https://github.com/cozystack/community/pull/25) | myasnikovdaniil | [proposal] Per-cluster etcd | OPEN, `REVIEW_REQUIRED`, без движения с 03.09. Наше предложение минимального API принято целиком: на CR только `etcd.replicas`. **03.09 отправили отчёт о прогоне миграции** — ответа нет |

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

| PR | Автор | Что решил | Смержен | Релиз |
|---|---|---|---|---|
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
