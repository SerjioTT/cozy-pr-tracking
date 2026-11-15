# Отслеживание PR в cozystack

Обновлено: **2026-11-15**. Репозиторий по умолчанию — `cozystack/cozystack`, иначе указано явно.

На GitHub у issue и PR **общая нумерация**, по номеру их не отличить. Поэтому здесь issue всегда помечены словом — `issue #4083`, а голый `#4133` означает PR.

## Главное за 15.11

- **13.11 lexfrei разобрал почти всю нашу очередь в `main`: восемнадцать PR смержены за один день.** С 11:07 до 11:22 он влил `main` в ветки всех наших PR (комментарий в каждом: «I merged main into your branch … Your commits are unchanged»), обязательные прогоны впервые пошли и прошли, и с 12:14 до 15:25 смержены — все merge-коммитом, все им самим: #5100, #5094, #4986, #5249, #4960, #5102, #4948, #5098, #5119, #4993, #5096, #5114, #4945, #4946, #5011, #5009, #4963, #4965. Семь из девяти одобренных IvanHunters 23.10 ушли с тем апрувом плюс апрув lexfrei; на #4986 вернулся и myasnikovdaniil (`APPROVED` 13.11). Автоматически закрылись issue #5002 и issue #5004 (мерж #5011 и #5009 в 15:25), issue #3759 (#5114, 15:12) и issue #5248 (#5249, 12:41). Плановый проход 13.11 шёл в 12:15 UTC и застал только начало — в снимке 13.11 это не записано.
- **Пять PR получили `CHANGES_REQUESTED` от lexfrei, один — комментарий с двумя пунктами; все шесть — работа на нашей стороне.** #4947: нужна ручка opt-out `master.topologySpreadPolicy: hard | soft` по образцу opensearch, `hard` по умолчанию, тainted-нода и ручка — в release note (`nodeTaintsPolicy: Ignore` считает недоступную мастерам ноду за пустую, и третий мастер на двух годных нодах не планируется никогда). #4989 и #4990: код принят, нужна release note с выходом для застрявшей реплики — `scheduling.globalAppTopologySpreadConstraints` с `ScheduleAnyway` до апгрейда либо пересоздание PVC с потерей данных этой реплики; у #4990 ещё случай «меньше годных нод, чем sentinel'ов», при котором оператор пропускает все проверки. #5040: ресурс алерта должен быть `<cluster>/<namespace>` (из `$labels.cluster` и `$labels.namespace`) — Alerta дедуплицирует по environment/resource/event и склеивает два кластера в один алерт. #5121: после слияния с `main` красный `Unit & controller tests` (`hack/mariadb-first-boot-waits.bats`, тест 1201), таймаут op в `chainsaw-test.yaml` поднять минимум до 3180s, ссылку на `docs/operations/backup-classes.md` заменить на website#788. #5097 (`COMMENTED`): конфликт с `main` в `platform/templates/apps.yaml`, `platform/values.yaml`, `cozystack-values-secret.yaml` и тесте — `main` теперь так же гейтит `_namespace.ingress`; оставить оба гейта и переписать комментарий и имена тестов; в release note добавить, что slim-кластер с SeaweedFS на `Tenant/root` должен выставить `seaweedfs.rootEnabled: true`. **#4875 за это время стал `DIRTY`/`CONFLICTING`** — нужен ребейз, lexfrei к нему после 09.11 не возвращался. Не тронуты вовсе #5110, #5183, #5189: прогоны в `action_required`, ревью нет.
- **13.11 в 12:54 вышел стабильный v1.7.1** — промоут v1.7.1-rc.2, тег `c0b53abb` на голове `release-1.7`, через #5313. Наши #4985 и #5005 в нём и названы в release notes (tcp-balancer больше не OOM'ится, etcd-алерт не считает Canceled). Восемнадцать мержей 13.11 в v1.7.1 **не вошли** — rc.2 снят 12.11, до них. `release-1.7` после тега не двигался.
- **Что с ними сделает батч — проверено локально черри-пиком собственных коммитов каждого PR (как пикает батч) на головы `release-1.7` `c0b53abb` и `release-1.6` `bf4dfdd0`.** Ближайший плановый прогон — понедельник 17.11 в 06:23 UTC (cron `23 6 * * 1,4`), он продолжит открытую пачку #5311. В 1.7 из восемнадцати шестнадцать — `fix` после отсечки, их батч возьмёт сам; четырнадцать ложатся чисто, **#5096** (`workloadmonitor_controller_test.go`) и **#5119** (`postgres.yaml`) конфликтуют и попадут в «Deferred conflicts» — их разрешает мейнтейнер на пачке. `feat` #4993 и ломающий #5009 батч пропустит по правилам 2 и 3 — им нужен `backport/include`, как и `test` #5025, который #5311 уже держит в Skipped. В 1.6 те же PR подпадают под политику (смержены после 26.10) и поедут **после** мержа пачки 1.7 (правило 6): чисто ложатся #4946, #4948, #4963, #4965, #5094, #5098, #5100, #5102, #5114 и, при лейбле, #4993 и #5009; конфликтуют #4945, #4960, #4986, #5011, #5096, #5119, #5249 (#4960 к тому же опирается на #4796, которого нет ни в одной линии — `[вывод]` батч его запаркует). Новый раздел «Бэкпорты наших PR — что сделает батч» ниже — по каждому PR.
- **Поправка к записи от 09.11: `backport/include` обходит и дату старта линии.** В `docs/release/backports.md` (и на `7097cc5c`, и на `main`) лейбл «takes it … although it merged before its line started under this policy»; не обходится только точка ветвления (release-1.6 отведена 22.07) и парковка. То есть для 1.6 наши смерженные до 26.10 #4985, #5005, #5025 и чужие #4289, #4427, #3633, #4095 может привезти батч — по лейблу, без ручных PR. **Ставить лейблы можем только мейнтейнеры**: у нас на репозитории `triage: false`, у yankawai `author_association: CONTRIBUTOR`; все три существующих `backport/skip` поставил myasnikovdaniil 10.11, `backport/include` пока не ставил никто. Ручной PR против `release-*` от автора без triage-прав `pr-labeler.yaml` закрывает при открытии — `[вывод]` новые ручные бэкпорты yankawai подавать больше нельзя, уже открытые 24 живы.
- **Из 24 наших ручных PR в `release-1.6` семь уже лишние.** Их оригиналы смержены 13.11 и ложатся на 1.6 чисто — батч довезёт сам: #4964, #4966, #5095, #5099, #5101, #5103, #5115. Ещё два станут лишними, как только на оригинал поставят `backport/include`: #4994 (#4993) и #5010 (#5009). Три нужны как готовое разрешение конфликта: #4988 (#4986), #5012 (#5011), #5120 (#5119). Пять ждут мержа своего оригинала, у которого на 1.6 конфликт: #4991, #4992, #5041, #5111, #5122. Шесть переносят оригиналы до 26.10 — только `backport/include` или мерж руками: #4967, #4987, #5001, #5006, #5007, #5026. #4961 — фикс только ветки, `APPROVED` myasnikovdaniil с 22.10. Таблица — в разделе «release-1.6». Пачка 1.6 #5312 стоит draft с красным `Unit & controller tests`; `release-1.6` на `bf4dfdd0` с 02.11.
- **Остальное.** `main` за 13–15.11 ушёл далеко вперёд (голова `5a5d5cd3` 15.11 03:00, сверх наших — десятки чужих `fix` и `feat`). Пачка #5311 в 1.7: draft, два оригинала (#5152, #5226), CI зелёный. Прогоны батча: 09.11 ×2, 12.11 ×2, 13.11 08:53 (`workflow_dispatch`, myasnikovdaniil). #5107 — всё так же `APPROVED`, ждёт ответа автора lllamnyp'у; community#25, #4817 — без движения. Новых PR и issue у yankawai после 09.11 нет. На наших ручных PR в 1.6 за это время никто не появлялся.

### Предыдущий заход — 13.11

- **12.11 в 12:38 вышел v1.7.1-rc.2 — наши #4985 и #5005 и в нём.** Тег на `12705a1c` — merge-коммите пачки #5290, rc.1 — его предок. Сверх rc.1 в rc.2 один фикс — чужой #5284 (`fix(seaweedfs)`: COSI-драйвер поднят до v0.4.0 с фиксом потерянного гранта). Чужие #5150 и #5147 не вошли и в rc.2 — оба в линии по-прежнему открыты. `[вывод]` Стабильный v1.7.1 будет промоутом одного из rc — наши фиксы доедут до стабильного с ним.
- **Пачка #5290 прошла линию 1.7 за 18 минут — внеплановым прогоном батча.** 12.11 в 11:31:44 myasnikovdaniil смержил #5284 в `main` и в 11:31:57 сам запустил «Backport batch» (`workflow_dispatch`); бот открыл #5290 в 11:33 с единственным оригиналом #5284, мейнтейнер одобрил её в 11:51:21 и смержил в 11:51:46, а в 12:38 на merge-коммите вышел rc.2. `[вывод]` Прогон запускали руками ради COSI-фикса перед rc.2. Тот же прогон прошёл и по 1.6: на #5247 новый иммутабельный отчёт в 11:34 — состав тот же, девять чужих оригиналов, голова `f77860df` не менялась, красный helm-unittest mariadb-operator остался; `backport/skip` на #5083, #4973 и #5180 держатся.
- **13.11 в 08:14 смержен #5269 — батч стал строже к неполным переносам.** Оригинал, часть изменений которого не имеет эффекта на линии (правка или удаление файла, которого там нет), паркуется как partial целиком — вместо переноса половины фикса, как случилось в 1.6; оригинал, опирающийся на код, которого линия не несёт, паркуется или ждёт пререквизита; `backport/include` не снимает ни парк, ни ожидание, ни partial; пик, отличный от оригинала, помечается `changed`, его release note уходит под «Check these notes». Для нас выводы не меняются: в 1.6 наши ручные PR остаются единственным путём.
- **`main` +10 мержей после прохода 12.11, наших нет**: 12.11 — #5282 (lexfrei, `test(release)`), #5284, #5226 (`fix(platform)`: пустой ingress у tenant-root без собственного), #5228 (`fix(seaweedfs)`: lifecycle-правила S3-бакетов) и #5232 (`fix(clickhouse)`: креды system-bucket у работающего бэкап-сайдкара) — три последних scooby87; 13.11 — #5152 (mattia-eleuteri, `fix(monitoring-agents)`: деинсталляция релиза без оператора VictoriaMetrics), #5221 (`fix(postgres)`: бэкап-Job'ы не дерутся за WAL-архив) и #5229 (`fix(clickhouse)`: упавшие попытки бэкапа не попадают в бакет) от scooby87, #5288 (`ci(e2e)`) и #5269 (`feat(release)`). Голова — `0f169540`. Семь `fix` возьмут следующие прогоны батча обеих линий; `test`, `ci` и `feat` без `backport/include` правила не берут.
- **На нашем без движения.** Девять одобренных — тот же `APPROVED`; #4875 и #4986 ждут возврата ревьюеров; #5107 одобрен и ждёт ответа myasnikovdaniil на вопрос lllamnyp; #5119 — `CONFLICTING`, ребейз за yankawai; бэкпорты в 1.6 на прежних головах. Нового у yankawai после 09.11 нет — поиск по `created` пуст. Отслеживаемые открытые issue не менялись; `release-1.6` — `bf4dfdd0`.

## Требует действия

Сначала то, что на нашей стороне, потом — мейнтейнеры.

| Что | Где | Кому и что делать |
|---|---|---|
| **Правки по ревью lexfrei от 13.11** | #4947, #4989, #4990, #5040, #5121, #5097 | **yankawai**: #4947 — ручка `master.topologySpreadPolicy: hard / soft` (`hard` по умолчанию, на `soft` пустые `topologySpreadConstraints`), tainted-нода и ручка в release note; #4989 и #4990 — release note с выходом для застрявшей реплики (`scheduling.globalAppTopologySpreadConstraints` → `ScheduleAnyway` до апгрейда либо пересоздание PVC с потерей её данных), у #4990 ещё случай «годных нод меньше, чем sentinel'ов»; #5040 — ресурс `<cluster>/<namespace>` (из `$labels.cluster` и `$labels.namespace`) в правиле и в ожидаемых labels теста; #5121 — починить `hack/mariadb-first-boot-waits.bats` тест 1201, поднять таймаут op в `chainsaw-test.yaml` до ≥3180s, ссылку на `docs/operations/backup-classes.md` заменить на website#788; #5097 — разрешить конфликт с `main` (оба гейта, переписать комментарий и имена тестов про безусловность) и добавить в release note `seaweedfs.rootEnabled: true` для slim-кластеров с SeaweedFS на `Tenant/root`. Пушить можно — ни на одном нет действующего апрува |
| **Ребейз #4875** | #4875 | **yankawai**: PR стал `DIRTY`/`CONFLICTING`; после ребейза — снова ждать lexfrei, его `CHANGES_REQUESTED` от 08.11 закрыт правками 09.11 |
| **Закрыть семь лишних ручных бэкпортов в 1.6** | #4964, #4966, #5095, #5099, #5101, #5103, #5115 | **yankawai**: оригиналы смержены 13.11 и ложатся на `release-1.6` чисто — батч перенесёт сам после мержа пачки 1.7. После `backport/include` на #4993 и #5009 туда же — #4994 и #5010 |
| **«Approve and run workflows» и первое ревью** | #5110, #5183, #5189 | Мейнтейнеры: единственные три наших PR в `main`, которых 13.11 не коснулись. #5110 — `feat`, шаблон тем и тел писем алертов; #5183 — алерты тенантного etcd только по своему релизу; #5189 — квота hugepages тенанта только как requests (#5184 уже в `main`). Прогоны в `action_required` |
| **`backport/include` на смерженное, которое батч не возьмёт сам** | #4993, #5009, #5025 | Мейнтейнеры: `feat` #4993 и ломающий #5009 пропускаются правилами 2 и 3, `test` #5025 уже в Skipped у #5311. Лейбл действует на обе линии; на 1.6 все три ложатся чисто. Позже то же для #5040 и #5110 при их мерже |
| **`backport/include` для 1.6 на оригиналы до 26.10** | #4985, #5005, #4095, #4289, #4427, #3633 | Мейнтейнеры: по политике лейбл обходит дату старта линии, и батч привезёт их в 1.6 сам (#4985 и #5005 уже в 1.7); иначе — смержить ручные #4987, #5006, #5026, #4967, #5001, #5007 как есть. Решение за вами, нам годится любое |
| **Провести пачку #5311 в 1.7 после прогона 17.11 и разрешить два конфликта** | #5311, #5096, #5119 | myasnikovdaniil: прогон 17.11 добавит в пачку шестнадцать наших `fix`; #5096 и #5119 упадут в Deferred conflicts — разрешаются на пачке по `backport-tasks/resolve-conflict.md`. В 1.6 те же конфликты плюс #4945, #4960, #4986, #5011, #5249; для #4986, #5011, #5119 готовые решения — наши #4988, #5012, #5120 |
| **Провести пачку #5312 в 1.6** | #5312 | myasnikovdaniil: draft с 13.11, красный `Unit & controller tests`; пока она не смержена, в 1.6 не приедет ничего — правило 6 и очередь |
| **Смержить #4961** | #4961 | myasnikovdaniil: фикс только `release-1.6` без оригинала в `main`, `APPROVED` вами 22.10; батч такие не возит |
| **Ответить на вопрос lllamnyp по #5107** | #5107 | myasnikovdaniil: `APPROVED` IvanHunters 09.11, вопрос от 11.11 про спецификацию installation defaults; за ним — путь к issue #3950 |
| **Затриажить issue** | issue #5070 | Мейнтейнеры: дефолтный тенантный Kubernetes не влезает в тенанта с `resourceQuotas.cpu 24`, с 22.10 без комментариев |
| **Ответить на два вопроса в issue** | issue #4962 | Мейнтейнеры: выкачка образов при апгрейде роняет управляющий etcd в перевыборы; pre-pull в гайде или механизм в релизе |
| **Закрыть issue** | issue #2483 | Мейнтейнер: решён #4766, парный issue #2793 закрыт 06.11 |
| **Правки по ревью в proposal** | community#25 | myasnikovdaniil: ревью lexfrei от 12.10, пункт 6 — наш runbook живой миграции, текст готовы прислать |
| **Правки по ревью** | #4817 | lllamnyp: `CHANGES_REQUESTED` lexfrei 12.10, draft с 14.10; кусок B1 (флаги defrag) — за нами, как только PR выйдет из draft |

**Ручные бэкпорты в `release-*` больше не подаём.** По `docs/release/backports.md` их открывают только мейнтейнеры, PR от автора без triage-прав закрывается при открытии; всё, что смержено в `main` как `fix`, батч возит сам. Исключения — `feat`, ломающие и `test` — решаются лейблом `backport/include`, который ставит мейнтейнер.

## Наши PR — открытые

В `main` у нас десять открытых PR — после того как 13.11 lexfrei смержил восемнадцать (строки переехали в «Смержено в main»): #4875 от 11.10 — `CHANGES_REQUESTED` lexfrei 08.11, правки 09.11, с 13.11 `CONFLICTING`; #4947 от 18.10, #4989 и #4990 от 20.10, #5040 от 21.10, #5121 от 26.10 — `CHANGES_REQUESTED` lexfrei 13.11 после того, как он влил `main` в ветки и прогоны прошли; #5097 от 25.10 — `COMMENTED` lexfrei 13.11 с двумя пунктами; #5110 от 26.10, #5183 от 02.11 и #5189 от 03.11 — не тронуты: ревью нет, прогоны в `action_required`. Апрув IvanHunters от 23.10 остался только на #4947, но его перекрывает `CHANGES_REQUESTED`. Ни на одном из десяти нет действующего апрува — пушить правки можно.

**[#4875](https://github.com/cozystack/cozystack/pull/4875)** — yankawai — fix(cozy-lib): delete the Secrets cert-manager writes when an app is deleted

**Что решает.** Удаление приложения больше не оставляет в неймспейсе тенанта Secret'ы с приватными ключами, которые cert-manager написал под его Certificate: post-delete хук удаляет их по точному списку имён, им подключены qdrant, redis, rabbitmq, nats, opensearch, плюс Secret'ы ingress-сертификатов harbor, bucket и bootbox. E2E-guard из закрытого #4876 валит сьюту, которая оставила такой Secret. Решает issue #4704

**Состояние.** Открыт 11.10 в 17:33. **16.10 в 21:40:41 force-push: оба пункта ревью внесены, ветка ребейзнута на main `aed7fa89` и стала `MERGEABLE`.** Семь коммитов: `57d2e750` даёт имя `printf "%s-secrets-cleanup" $root.Release.Name | trunc 63 | trimSuffix "-"` Job'у, ServiceAccount, Role и RoleBinding, с тремя кейсами cozy-lib-tests на 53-символьном релизе — `helm template` redis с таким релизом рендерит 63 символа вместо прежних 69; `cb1891c6` проводит `clastix/kubectl` через `cozy-lib.image` и добавляет `_hooks.tpl` в renovate-менеджер пина. Конфликт в `hack/e2e-chainsaw/README.md` разведён, обе стороны сохранены. По отчёту автора: helm unittest зелёный на cozy-lib-tests 87/87, qdrant 112/112, redis 74/74, rabbitmq 109/109, nats 81/81, opensearch 159/159, harbor 46/46, bucket 12/12, bootbox 12/12. **08.11 в 08:50 второе ревью lexfrei — снова `CHANGES_REQUESTED`, NOT LGTM**: хук очистки удаляет настроенный общий wildcard Secret — `publishing.certificates.wildcardSecretName` (пример `bootbox-tls`) всегда передаётся хелперу очистки, в wildcard-режиме приложение своего TLS-Secret не создаёт, и uninstall сносит операторский. Просит сохранять настроенный общий wildcard Secret и удалять только app-owned цели, включая отдельные хуки harbor и bucket, оставив очистку ключей от прежних TLS-ревизий; прежние блокеры признаны закрытыми. **09.11 в 11:48 правки внесены восьмым коммитом `c2f1d03e` `fix(apps): preserve shared wildcard secrets during cleanup` — нужен возврат ревьюера.** Обязательные прогоны на новой голове в `action_required`. **13–15.11:** lexfrei к PR не возвращался, в `main` 13.11 влиты десятки PR — голова `c2f1d03e` стала `DIRTY`/`CONFLICTING`; ребейз за нами. Прогоны по-прежнему `action_required`

**[#4947](https://github.com/cozystack/cozystack/pull/4947)** — yankawai — fix(seaweedfs): keep the masters on separate nodes through a drain

**Что решает.** У StatefulSet мастеров SeaweedFS только `preferred` анти-аффинити, которое решает, где мастер стартует, но не куда его вернёт дренаж: на трёхнодовом кластере вытесненный мастер стартует рядом с другим и после возврата ноды никуда не уходит — найдено на двух кластерах v1.6.4, причём на одном оба оказались там в одну секунду во время обслуживания ноды. Потеря такой ноды уносит два из трёх raft-членов и кворум. PR даёт мастерам hostname-спред с `whenUnsatisfiable: DoNotSchedule`, `maxSkew: 1` и `nodeTaintsPolicy: Ignore`: закордоненная нода продолжает считаться, замена ждёт Pending и возвращается на неё при uncordon, а на четырёх и более нодах уезжает на свободную

**Состояние.** Открыт 18.10 в 21:04:13, не draft, **21.10 в 10:19:57 force-push: коммит про monitoring снят, остался один `b47fed73`**, а PR переименован с `fix(seaweedfs, monitoring)`; лейбл сменился с `size/L` на `size/M`, файлов два — `values.yaml` на +25 и `master_affinity_test.yaml` на +46. `MERGEABLE`, `REVIEW_REQUIRED`, ревью нет, обязательные прогоны в `action_required`. Спред реплик monitoring теперь целиком за #4989, выбирать между двумя подходами мейнтейнерам больше не нужно. Автор сам называет две цены: нода, на которой мастера не живут (например тейнтованный control plane), всё равно считается, поэтому при двух пригодных нодах рядом с такой третий мастер остаётся Pending и апгрейд там встаёт; на двухнодовом кластере дренаж ноды с двумя мастерами оставит оба Pending до её возврата. Проверено в локальном kwok-кластере на kube-scheduler v1.34.12. **В теле два прямых вопроса мейнтейнерам** — приемлем ли случай тейнтованной ноды как задокументированный и нужна ли ручка opt-out вроде `topologySpreadPolicy` у opensearch. **13.11:** lexfrei в 11:21 влил `main` в ветку (`943a1d7b`, наши коммиты без изменений), прогоны прошли зелёными, в 11:48 — `CHANGES_REQUESTED` «NOT LGTM, one fix inline»: нужен opt-out `master.topologySpreadPolicy: hard / soft` по образцу `topologySpreadPolicy` у opensearch — на `soft` чарту передаётся пустой `topologySpreadConstraints` и остаётся только preferred анти-аффинити; `hard` по умолчанию; tainted-нода и ручка — в release note. Причина: `nodeTaintsPolicy: Ignore` считает недоступную мастерам ноду (kubeadm control plane, tainted GPU-воркер) за ноду с нулём мастеров, и на двух годных нодах рядом с одной такой третий мастер не планируется никогда — свежая установка не становится ready, апгрейд виснет на первом мастере; это тот же hang, что чинил #3197. Апрув IvanHunters от 23.10 остался, но решает `CHANGES_REQUESTED`

**[#4989](https://github.com/cozystack/cozystack/pull/4989)** — yankawai — fix(monitoring): spread monitoring replicas over nodes

**Что решает.** Чарт monitoring рендерит VMCluster, VLCluster, VTCluster и VMAlertmanager без `affinity` и `topologySpreadConstraints`, а vm-operator v0.68.4 своих правил не добавляет. На трёх трёхнодовых кластерах v1.6.4 и v1.6.5 реплики vmstorage, vmselect и vlstorage делили одну ноду. Хранящие тиры и Alertmanager получают жёсткий спред `DoNotSchedule`, stateless — `ScheduleAnyway`

**Состояние.** **Открыт 20.10 в 10:30**, три коммита, ревью нет, прогоны в `action_required`. 20.10 в 19:15 автор добавил третий коммит `33783b4c` и разобрал его в комментарии в 21:55: `merge` в тернарнике дописывал labels селектора политики в общую map `$selector`, и у политики со смешанными constraints чужие labels расползались по всем; теперь слияние идёт в глубокую копию constraint'а. Бэкпорт #4991 подан сразу, тот же коммит там как `40f5d5c9`. **Альтернатива второму коммиту #4947**, который даёт тем же тирам мягкое анти-аффинити: по тексту PR, если мейнтейнеры выберут этот подход, коммит про monitoring в #4947 не нужен. **13.11:** lexfrei в 11:10 влил `main` (`44633a0f`), прогоны зелёные, в 12:15 — `CHANGES_REQUESTED`: «The code is fine and CI is green; only the release note needs a fix» — note говорит, что реплика висит Pending «until that replica is moved», но не как; нужно расписать выход: `scheduling.globalAppTopologySpreadConstraints` с политикой `ScheduleAnyway` до апгрейда, либо пересоздать PVC застрявшей реплики на ноде без соседа с потерей её данных

**[#4990](https://github.com/cozystack/cozystack/pull/4990)** — yankawai — fix(redis, valkey, harbor): spread sentinels and data pods over nodes

**Что решает.** RedisFailover приложений redis и valkey и Harbor не задают размещения, и оператор ставит лишь мягкое анти-аффинити. На v1.6.4 и v1.6.5 два sentinel'а из трёх стояли на одной ноде и у тенантного Redis, и у Redis в Harbor: падение этой ноды оставляет один sentinel — ниже кворума, failover невозможен. Sentinel'ы и data-поды получают спред `DoNotSchedule`

**Состояние.** **Открыт 20.10 в 10:32**, четыре коммита, голова `a2576bcf`, ревью нет, прогоны в `action_required`. 20.10 в 18:44 добавлен `2119b7ac` — селектор проставляется каждому платформенному constraint'у, та же находка, что в #4989. **09.11, вслед за чужим #5151 (у Redis с одной репликой один Sentinel): в 11:51 в ветку влит `main` merge-коммитом `0d389f4` с сообщением `fix(redis): preserve sentinel replica policy with strict spread` — спред должен действовать и при одно-sentinel'ной политике реплик, — в 12:20 добавлен `a2576bc` — null-селектор политики спреда читается как незаданный, вместо «не матчит ничего»**. Бэкпорт #4992 подан сразу (`2119b7ac` там как `8a667f21`) и 09.11 дополнен коммитом про null-селекторы — правка про политику реплик в 1.6 не нужна, #5151 там нет. **13.11:** lexfrei в 11:22 влил `main` (`10fc1f41`), прогоны зелёные, в 11:55 — `CHANGES_REQUESTED`: код принят, в release note нужен ещё один случай — при `nodeTaintsPolicy: Ignore` нода, которую поды не толерируют, считается пустой, и на кластере с меньшим числом годных нод, чем sentinel'ов, последний sentinel остаётся Pending, а пока любой под Pending, оператор пропускает все проверки (`checker.go#L101-L111` v3.3.5): новый инстанс не получает мастера, апгрейд виснет. В note — этот случай, opt-out (`scheduling.globalAppTopologySpreadConstraints` → `ScheduleAnyway` до апгрейда) и конкретные шаги для застрявшей реплики; опционально поправить сообщение коммита `902051e7`, что «an instance with more pods than nodes still starts» — на таком кластере это неверно

**[#5040](https://github.com/cozystack/cozystack/pull/5040)** — yankawai — feat(monitoring): alert when a namespace stays in Terminating

**Что решает.** На застрявший в Terminating неймспейс в репозитории не горит ни одно правило: kube-state-metrics отдаёт `kube_namespace_status_phase`, дашборд `capacity-planning.json` его читает, а алерта нет. На кластере v1.6.5 тенантный неймспейс, удалённый напрямую, провисел так около 28 часов на финализаторах Kamaji из issue #5002, и единственным сигналом был мигающий `HelmReleaseNotReady` тенантного релиза. Новый warning `KubeNamespaceStuckTerminating` в новой группе `namespace-deletion` зажигается, когда `max by (cluster, namespace)` по фазе `Terminating` держится 30 минут; группировка по `cluster` и `namespace`, а не по инстансу, чтобы таймер не сбрасывался при переезде пода kube-state-metrics на другую ноду, а описание называет неймспейс, кластер и указывает на условия статуса с оставшимися типами и финализаторами

**Состояние.** Открыт 21.10 в 12:53:23, не draft, один коммит `79627dc2` после force-push 21.10 в 21:31, три файла: само правило, `hack/namespace-deletion-alerts.bats` и `hack/testdata/namespace-deletion-alerts/unittest.yaml`. `MERGEABLE`, `REVIEW_REQUIRED`, ревью нет, обязательные прогоны в `action_required`. Порог 30 минут выбран по замерам: здоровое удаление тенанта укладывалось в минуту в 61 случае из 61, а с Kubernetes, VM и Postgres внутри — около пяти минут. Сьюта гоняет `.spec` VMRule через `vmalert-tool` v1.136.0, тот же стенд, что у #4963, #4985 и #4993: 5 кейсов и 11 проверок, включая переезд kube-state-metrics на другую ноду на пятнадцатой минуте и один алерт на кластер при совпадающих именах. 18 Bats-сьют дают 325 из 325. **13.11:** lexfrei в 11:10 влил `main` (`bb8d9e5c`), CI и E2E зелёные, в 12:53 — `CHANGES_REQUESTED`, одна правка: Alerta строит resource из `exported_instance` и дедуплицирует по environment/resource/event без тега `cluster`, поэтому два алерта из кейса «одно имя в двух кластерах» склеиваются в один, и resolve из одного кластера закрывает его, пока неймспейс в другом ещё застрял. Нужно `<cluster>/<namespace>` (из `$labels.cluster` и `$labels.namespace`) в `namespace-deletion.yaml` и то же значение в ожидаемых labels теста

**[#5097](https://github.com/cozystack/cozystack/pull/5097)** — yankawai — fix(platform): stop handing tenants a root S3 the root tenant does not run

**Что решает.** `cozystack-basics` безусловно пишет `seaweedfs: tenant-root` в Secret `cozystack-values` корневого тенанта, поэтому каждый тенант без своего SeaweedFS наследует `tenant-root` как S3 — даже когда у `Tenant/root` стоит `spec.seaweedfs: false`. На v1.6.5 это дало 8796 записей `bucketclasses ... not found` за три дня. Новое платформенное значение `seaweedfs.rootEnabled` доезжает в cozystack-basics как `_cluster.seaweedfs-enabled` — тем же путём, каким `monitoring.rootEnabled` доезжает для #3181, — и `_namespace.seaweedfs` пишется только при `true`; slim-варианты ставят `false`. Без S3 по цепочке чарты bucket и harbor теперь отказываются рендериться с сообщением, называющим проблему, вместо создания claim'а под класс, которого не будет

**Состояние.** OPEN с 25.10 в 13:54:19, один коммит `5a6460d9`, четыре новых набора тестов, 288 из 288 в платформе и 69 из 69 в cozystack-basics. **Бэкпорта в 1.6 нет.** Ревью нет, прогоны в `action_required`. **13.11 в 10:41 — `COMMENTED` lexfrei, два пункта до мержа:** (1) конфликт с `main` в `platform/templates/apps.yaml`, `platform/values.yaml`, `cozystack-basics/templates/cozystack-values-secret.yaml` и его тесте — `main` теперь так же гейтит `_namespace.ingress` на `_cluster.ingress-enabled`; оставить оба гейта и переписать комментарий и имена тестов, где etcd, ingress и monitoring названы безусловными (после слияния безусловны только etcd и monitoring); «I didn't merge it myself because those sentences need your wording»; (2) release note: slim-варианты ставят `seaweedfs.rootEnabled: false`, но `values-isp-slim.yaml` разрешает включить SeaweedFS на `Tenant/root` — такой кластер на апгрейде теряет `_namespace.seaweedfs`, и его Buckets и Harbor падают на рендере; написать, что slim-кластер с корневым SeaweedFS должен выставить `seaweedfs.rootEnabled: true`. `main` в ветку он не вливал, прогоны в `action_required`

**[#5110](https://github.com/cozystack/cozystack/pull/5110)** — yankawai — feat(monitoring): take an Alertmanager template file for alert emails

**Что решает.** Почтовый receiver из #3800 шлёт дефолтные тему и HTML-тело Alertmanager: тема склеивает все общие labels группы — для застрявшего DaemonSet это строка на пол-экрана, — а тело оказывается таблицей labels и annotations; шаблонов темы и тела в values #3800 не оставил. Новое поле `alerta.alerts.email.template` принимает текст файла шаблонов Alertmanager: чарт кладёт его в ConfigMap `alertmanager-email-template` и перечисляет в `VMAlertmanager.spec.templates`, оператор v0.68.4 монтирует файл, добавляет путь в `templates` конфигурации и отдаёт reloader'у, так что неразбираемый шаблон валит загрузку конфигурации. Текст обязан содержать `define`, нестроковое значение валит рендер. По тексту PR, ту же схему руками держат на трёх кластерах с Alertmanager v0.31.0

**Состояние.** **Открыт 26.10 в 14:09**, +136/−3, лейблы `kind/feature`, `size/L`, `area/monitoring`, голова `fc1fef92`, последнее обновление 26.10 в 15:53. Ревью нет, `REVIEW_REQUIRED`, `MERGEABLE`/`BLOCKED`; `Pull Request`, `Pre-Commit Checks`, `API Review Gate` и CodeQL в `action_required`. Бэкпорт в 1.6 — #5111; **в 1.7 бэкпорта нет**. **13–15.11:** единственный из PR от 26.10, которого lexfrei не коснулся — `main` не влит, прогоны `action_required`, ревью нет. Бэкпорт #5111 открыт; черри-пик на обе линии конфликтует в `cozyrds/monitoring.yaml`, а как `feat` без `backport/include` батч его не возьмёт

**[#5121](https://github.com/cozystack/cozystack/pull/5121)** — yankawai — fix(mongodb): restore a backup into another application with its own users

**Что решает.** Восстановление бэкапа MongoDB в другое приложение ломает цель: логический restore psmdb переигрывает дамп целиком вместе с `admin.system.users`, у цели оказываются пользователи источника, её Secret'ы `<release>-percona-server-mongodb-users` и `internal-<release>-users` перестают аутентифицировать, и оператор теряет вход в только что восстановленный кластер — так и с `RestoreJob` to-copy из issue #3832, и с `bootstrap` чарта. Новое `bootstrap.databases` включает селективный restore (`spec.selective`, `withUsersAndRoles: false`): данные из бэкапа, учётки — свои. Кроме того, bootstrap-restore ждёт готовности кластера — оператор 1.22.0 иначе валит его навсегда — и читает из своего `bootstrap.destinationPath`; всего двенадцать коммитов, четыре из них про `RestoreJob`, а двенадцатый добавляет проверку учёток после restore. В теле `Fixes #3832`

**Состояние.** **Открыт 26.10 в 17:53**, двенадцать коммитов: 01.11 в 08:41 добавлен `3b77a992` `fix(mongodb): verify credentials after restore` — контроллерная часть в `internal/backupcontroller`, в чартовый бэкпорт #5122 не входит. 04.11 в 22:06 в ветку влит `main`: голова — merge-коммит `cadfc7f4`, тринадцать коммитов, +2231/−118, `kind/bug`, `size/XXL`, `area/database`. Ревью нет, `REVIEW_REQUIRED`; `Pull Request`, `Pre-Commit Checks`, `API Review Gate`, `Codegen Drift Check` и CodeQL в `action_required`. Бэкпорт в 1.6 — #5122, только чартовая часть; **в 1.7 бэкпорта нет**. **13.11:** lexfrei в 11:10 влил `main` (`0521a9a8`), и в 11:34 — `CHANGES_REQUESTED` «NOT LGTM, one fix inline»: слияние с текущим `main` валит `Unit & controller tests` — `hack/mariadb-first-boot-waits.bats`, тест 1201 ([job 131850043628](https://github.com/cozystack/cozystack/actions/runs/43414592544/job/131850043628)); inline: с включённым restore-плечом op в `chainsaw-test.yaml` слишком короток для трёх ожиданий HelmRelease по 660s — поднять таймаут минимум до 53m (3180s), резерв 1200s не трогать; вне диффа: раздел 9 описания ссылается на `docs/operations/backup-classes.md`, которого в `main` больше нет — ссылаться на website#788. Голова `0521a9a8`, 13 коммитов

**[#5183](https://github.com/cozystack/cozystack/pull/5183)** — yankawai — fix(etcd): scope tenant alerts to their scrape job

**Что решает.** Алерты тенантного etcd в `packages/extra/etcd/templates/prometheus-rules.yaml` не ограничены своим релизом: каждый релиз etcd оценивает правила по всем тенантам, и один сбойный член etcd поднимает алерты и от чужих релизов. PR ограничивает каждое правило задачей сбора метрик своего релиза; RTT между членами тенанта считают только тенантные правила, а в глобальных `packages/system/monitoring-agents/alerts/etcd.yaml` он больше не дублируется; остальные глобальные проверки etcd остаются, пороги не меняются. По тексту PR, имитация потери лидера по существующим метрикам до правки выбирала 18 членов в шести неймспейсах, после — только три нужных; тесты обоих чартов проходят, 46 тестов в 17 сьютах

**Состояние.** **Открыт 02.11 в 22:55**, один коммит `81fd5fa8`, +55/−16, четыре файла, `kind/bug`, `size/M`, `area/database`. Ревью нет, `REVIEW_REQUIRED`, `MERGEABLE`; `Pull Request`, `Pre-Commit Checks`, `API Review Gate` и CodeQL в `action_required`. **Бэкпортов нет — ни в 1.6, ни в 1.7.** Черри-пик на обе ветки конфликтует в `prometheus-rules.yaml`: PR правит строки, изменённые нашим #4985, которого в ветках нет; поверх #4985 ложится чисто на обе. **13–15.11:** lexfrei не коснулся — `main` не влит, прогоны `action_required`, ревью нет. С мержем #4985 в 1.7 (пачка #5246) черри-пик на `release-1.7` стал чистым; на 1.6 конфликт в `prometheus-rules.yaml` остаётся — там #4985 нет

**[#5189](https://github.com/cozystack/cozystack/pull/5189)** — yankawai — fix(cozy-lib): render tenant hugepages quota only as requests

**Что решает.** Тенант с `hugepages-2Mi` или `hugepages-1Gi` в `resourceQuotas` получает в `tenant-quota` и `requests.hugepages-<size>`, и `limits.hugepages-<size>`, а Kubernetes принимает huge pages в ResourceQuota только как `hugepages-<size>` или `requests.hugepages-<size>` — apiserver отклоняет `tenant-quota`, и релиз тенанта не ставится и не обновляется. Huge pages не входят и в `limits.memory`, поэтому ВМ на huge pages сейчас вообще не ограничена квотой тенанта. `cozy-lib.resources.flattenResource` теперь отбрасывает сторону limits для `hugepages-*`, как уже делает для `storage`; `sanitize` не меняется — поду по-прежнему нужны равные requests и limits. По тексту PR, `helm unittest` проходит 69 из 69 в `packages/tests/cozy-lib-tests` и 78 из 78 в `packages/apps/tenant`, новый кейс падает на `main`

**Состояние.** **Открыт 03.11 в 20:22**, один коммит `38a32445`, +36/−0, три файла, `kind/bug`, `size/M`, `area/platform`. Ревью мейнтейнеров нет, `REVIEW_REQUIRED`, `MERGEABLE`/`BLOCKED`; `Pull Request`, `Pre-Commit Checks`, `API Review Gate` и CodeQL в `action_required`. Бот-ревьюер Codex 03.11 в 20:30 оставил три замечания: проверка квоты дочернего тенанта в `pkg/registry/apps/application/quota.go` читает использование hugepages родителя из `limits.*` и после этого PR увидит ноль; нет трейлеров `Signed-off-by` и `Assisted-by`; тело коммита пересказывает diff. yankawai 04.11 в 22:03 ответил: первое чинит чужой #5184 lexfrei, оба должны ехать вместе или #5184 первым, регрессии между ними нет — до этого PR тенант с квотой hugepages не ставился вовсе; трейлеры на `38a32445` есть. **09.11 в 14:23:40 #5184 смержен в `main` (апрув kvaps) — порядковый блокер снят, PR едет сам.** **Бэкпортов нет — ни в 1.6, ни в 1.7**, локальный черри-пик чистый на обе ветки. **13–15.11:** lexfrei не коснулся — прогоны `action_required`, ревью нет; `MERGEABLE`. #5184, с которым его просили мержить вместе или после, в `main` с 09.11 — порядковый блокер снят. Черри-пик чистый на обе линии

### Бэкпорты наших PR — что сделает батч

С 09.11 бэкпорты возит батч `cmd/backport-batch` по `docs/release/backports.md`; раздел с 15.11 отвечает не «есть ли бэкпорт-PR», а «что батч сделает с каждым нашим PR в каждой линии». Строки — все наши открытые PR в `main` и всё наше, смерженное в `main` после отсечки `release-1.7` (12.10, `38d6ac7e`). Значения в колонках линий: **в линии** (с релизом), **возьмёт сам** (`fix` после старта линии, пик чистый), **конфликт** (батч отложит в Deferred, разрешает мейнтейнер на пачке — или готовый ручной PR), **нужен лейбл** (`feat`/`test`/ломающий или смержен до старта линии — только с `backport/include`), **парк** (опирается на то, чего в линии нет). Пики проверены 15.11 локальным `cherry-pick -x` собственных коммитов PR (у смерженных — коммитов `M^1..M^2`, как пикает батч) на `release-1.7` `c0b53abb` и `release-1.6` `bf4dfdd0`; «чисто» — без конфликтов, сборку и тесты не гоняли. В 1.6 всё едет только после того, как пачка 1.7 с тем же оригиналом смержена (правило 6).

**Ближайший прогон — понедельник 17.11 06:23 UTC**, он продолжит открытые #5311 (1.7) и #5312 (1.6). Лейблы ставят только мейнтейнеры: у нас `triage: false`, yankawai — `CONTRIBUTOR`. Открытые PR в таблице едут только после мержа в `main`.

| PR в `main` | Что | Состояние в `main` | `release-1.7` | `release-1.6` |
|---|---|---|---|---|
| [#4875](https://github.com/cozystack/cozystack/pull/4875) | cozy-lib: Secret'ы cert-manager удаляются вместе с приложением | открыт, `CHANGES_REQUESTED` lexfrei 08.11, правки 09.11, с 13.11 `CONFLICTING` — ребейз за нами | после мержа **возьмёт сам**; пик чистый | после мержа **возьмёт сам** (после 1.7); пик чистый |
| [#4947](https://github.com/cozystack/cozystack/pull/4947) | seaweedfs: мастера на разных нодах через drain | открыт, `CHANGES_REQUESTED` lexfrei 13.11 — ручка opt-out | после мержа **возьмёт сам**; пик чистый | после мержа **возьмёт сам**; пик чистый |
| [#4989](https://github.com/cozystack/cozystack/pull/4989) | monitoring: спред реплик по нодам | открыт, `CHANGES_REQUESTED` lexfrei 13.11 — release note | после мержа **возьмёт сам**; пик чистый | **конфликт** (`vtraces.yaml`); ручной #4991 — готовое решение |
| [#4990](https://github.com/cozystack/cozystack/pull/4990) | redis, valkey, harbor: спред sentinel'ов и data-подов | открыт, `CHANGES_REQUESTED` lexfrei 13.11 — release note | **конфликт** (`redisfailover.yaml`, в линии уже #5151) — Deferred | **конфликт** (`redisfailover.yaml`, `valkeyfailover.yaml`, `Makefile`); ручной #4992 — без `-x`, батч его за перенос не признает |
| [#5040](https://github.com/cozystack/cozystack/pull/5040) | monitoring-agents: алерт `KubeNamespaceStuckTerminating`, `feat` | открыт, `CHANGES_REQUESTED` lexfrei 13.11 — resource алерта | `feat` — **нужен лейбл**; пик чистый | `feat` — **нужен лейбл**; пик чистый; ручной #5041 |
| [#5097](https://github.com/cozystack/cozystack/pull/5097) | platform: тенантам не отдаётся корневой S3, которого нет | открыт, `COMMENTED` lexfrei 13.11 — конфликт с `main` и release note | после мержа **возьмёт сам**; пик текущей головы чистый | **конфликт** (`apps.yaml`, три `values-isp-slim*.yaml`); ручного нет |
| [#5110](https://github.com/cozystack/cozystack/pull/5110) | monitoring: файл шаблонов Alertmanager для писем, `feat` | открыт, ревью нет, прогоны не запускались | `feat` — **нужен лейбл**, и **конфликт** в сгенерированном `cozyrds/monitoring.yaml` | то же; ручной #5111 — единственный готовый путь |
| [#5121](https://github.com/cozystack/cozystack/pull/5121) | mongodb: restore в другое приложение со своими пользователями | открыт, `CHANGES_REQUESTED` lexfrei 13.11 — красный bats, таймаут, ссылка | после мержа **возьмёт сам**; все 12 коммитов чисто | **конфликт** (`cozyrds/mongodb.yaml`); ручной #5122 — только чартовая часть |
| [#5183](https://github.com/cozystack/cozystack/pull/5183) | etcd: алерты тенанта только по своему релизу | открыт, ревью нет, прогоны не запускались | после мержа **возьмёт сам**; пик чистый — #4985 в линии | **конфликт** (`prometheus-rules.yaml`): нужен сначала #4985 — лейбл на него или ручной #4987 |
| [#5189](https://github.com/cozystack/cozystack/pull/5189) | cozy-lib: квота hugepages тенанта только как requests | открыт, ревью нет, прогоны не запускались | после мержа **возьмёт сам**; пик чистый | после мержа **возьмёт сам**; пик чистый |
| [#4945](https://github.com/cozystack/cozystack/pull/4945) | apps: имена Job и CronJob в лимитах Kubernetes | **смержен 13.11 в 15:18** | **возьмёт сам** 17.11; чисто | **конфликт** (`migration-hook.yaml`) |
| [#4946](https://github.com/cozystack/cozystack/pull/4946) | keycloak, foundationdb: CRD операторов обновляются на апгрейде | **смержен 13.11 в 15:24** | **возьмёт сам** 17.11; чисто | **возьмёт сам** после 1.7; чисто |
| [#4948](https://github.com/cozystack/cozystack/pull/4948) | monitoring: `InfoInhibitor` уходит в blackhole | **смержен 13.11 в 12:56** | **возьмёт сам** 17.11; чисто | **возьмёт сам** после 1.7; чисто |
| [#4960](https://github.com/cozystack/cozystack/pull/4960) | vm-disk: отказ диску меньше golden-образа | **смержен 13.11 в 12:41** | **возьмёт сам** 17.11; пик чистый, но опирается на #4796, которого в линии нет — `[вывод]` **парк** | **конфликт** (`dv.yaml`) и та же зависимость — `[вывод]` **парк** |
| [#4963](https://github.com/cozystack/cozystack/pull/4963) | piraeus-operator: алерты по error-report'ам LINSTOR | **смержен 13.11 в 15:18** | **возьмёт сам** 17.11; чисто | **возьмёт сам** после 1.7; чисто; ручной #4964 лишний |
| [#4965](https://github.com/cozystack/cozystack/pull/4965) | tcp-balancer: порты kubernetes и talos только при `kubernetesAndTalos` | **смержен 13.11 в 15:24** | **возьмёт сам** 17.11; чисто | **возьмёт сам** после 1.7; чисто; ручной #4966 лишний |
| [#4986](https://github.com/cozystack/cozystack/pull/4986) | etcd: жёсткий спред членов по нодам | **смержен 13.11 в 12:39** | **возьмёт сам** 17.11; чисто | **конфликт** (`etcd-cluster.yaml`, `values.yaml`, `values.schema.json`, `README.md`, `etcd.yaml`); ручной #4988 — готовое решение |
| [#4993](https://github.com/cozystack/cozystack/pull/4993) | monitoring-agents: алерт `PodReplicasColocated`, `feat` | **смержен 13.11 в 14:38** | `feat` — **нужен лейбл**; чисто | `feat` — **нужен лейбл**; чисто; с лейблом ручной #4994 лишний |
| [#5009](https://github.com/cozystack/cozystack/pull/5009) | cozy-lib: память меньше 1Mi отклоняется, ломающий | **смержен 13.11 в 15:25** | ломающий `!` — **нужен лейбл**; чисто | ломающий — **нужен лейбл**; чисто; с лейблом ручной #5010 лишний |
| [#5011](https://github.com/cozystack/cozystack/pull/5011) | cozystack-basics: прямое удаление тенантного неймспейса отклоняется | **смержен 13.11 в 15:25** | **возьмёт сам** 17.11; чисто | **конфликт** (`e2e-install-cozystack.bats`, `external_tls_policy_test.go`); ручной #5012 — готовое решение |
| [#5094](https://github.com/cozystack/cozystack/pull/5094) | monitoring: VMServiceScrape vmalert селектит Service оператора | **смержен 13.11 в 12:16** | **возьмёт сам** 17.11; чисто | **возьмёт сам** после 1.7; чисто; ручной #5095 лишний |
| [#5096](https://github.com/cozystack/cozystack/pull/5096) | workloadmonitor: Bucket не ready, пока BucketClaim не провизионен | **смержен 13.11 в 15:11** | **конфликт** (`workloadmonitor_controller_test.go`) — Deferred | **конфликт** (три файла контроллера); ручного нет |
| [#5098](https://github.com/cozystack/cozystack/pull/5098) | vertical-pod-autoscaler: два admission controller со спредом | **смержен 13.11 в 13:32** | **возьмёт сам** 17.11; чисто | **возьмёт сам** после 1.7; чисто; ручной #5099 лишний |
| [#5100](https://github.com/cozystack/cozystack/pull/5100) | etcd: defrag-CronJob по размеру клиента | **смержен 13.11 в 12:14** | **возьмёт сам** 17.11; чисто | **возьмёт сам** после 1.7; чисто; ручной #5101 лишний |
| [#5102](https://github.com/cozystack/cozystack/pull/5102) | monitoring-agents: тихий Watchdog — тревога в Alerta | **смержен 13.11 в 12:56** | **возьмёт сам** 17.11; чисто | **возьмёт сам** после 1.7; чисто; ручной #5103 лишний |
| [#5114](https://github.com/cozystack/cozystack/pull/5114) | kubevirt: права тенантного админа для `virtctl image-upload` | **смержен 13.11 в 15:12** | **возьмёт сам** 17.11; чисто | **возьмёт сам** после 1.7; чисто; ручной #5115 лишний |
| [#5119](https://github.com/cozystack/cozystack/pull/5119) | postgres: пятипольный cron в `backup.schedule` | **смержен 13.11 в 14:38** | **конфликт** (`postgres.yaml`) — Deferred | **конфликт** (`postgres.yaml`); ручной #5120 — готовое решение |
| [#5249](https://github.com/cozystack/cozystack/pull/5249) | kubernetes: kubevirt-csi отсоединяет hotplug-том при пропавшем infra PVC | **смержен 13.11 в 12:41** | **возьмёт сам** 17.11; чисто | **конфликт** (`controller_test.go`); ручного нет |
| [#4889](https://github.com/cozystack/cozystack/pull/4889) | monitoring: vmalert шлёт алерты каждой реплике Alertmanager | смержен 14.10 | **в линии** — #4943, v1.7.0 | **в линии** — #4890, v1.6.5 |
| [#4985](https://github.com/cozystack/cozystack/pull/4985) | etcd: отменённые Watch-стримы не считаются ошибками | смержен 23.10 | **в линии** — пачка #5246, **v1.7.1** | смержен до старта линии — **нужен лейбл**; ручной #4987 |
| [#5005](https://github.com/cozystack/cozystack/pull/5005) | tcp-balancer: HAProxy по лимитам своего пода | смержен 23.10 | **в линии** — пачка #5246, **v1.7.1** | смержен до старта линии — **нужен лейбл**; ручной #5006 |
| [#5025](https://github.com/cozystack/cozystack/pull/5025) | harbor: тест ресурсов nginx-прокси и портала, `test` | смержен 23.10 | `test` — **нужен лейбл**, #5311 держит в Skipped | `test` и до старта — **нужен лейбл**; ручной #5026 вместе с #4095 |

### release-1.7 — v1.7.1 вышел, восемнадцать наших мержей ждут понедельника

**13.11 в 12:54 вышел стабильный v1.7.1** — промоут v1.7.1-rc.2 через #5313, тег `c0b53abb` на голове `release-1.7`; после тега линия не двигалась. Наши #4985 и #5005 в нём — пришли пачкой #5246 (смержена 11.11, v1.7.1-rc.1) и названы в release notes. Это первый наш релиз, собранный батчем целиком: ни ручных PR, ни лейблов.

**Открытая пачка #5311** (cozystack-ci[bot], 13.11 08:55, draft) несёт два чужих оригинала — #5152 и #5226; CI зелёный (`pre-commit`, `Unit & controller tests`, `Backport batch shape`). Deferred: #5228, #5229; Parked: #5232, #5221; `backport/skip` от myasnikovdaniil 10.11: #5083, #4973, #5180. В Skipped стоит наш `test` #5025 (правило 2).

**Плановый прогон 17.11 в 06:23 UTC продолжит её нашими мержами 13.11**: шестнадцать `fix` возьмёт сам, четырнадцать чисто, #5096 и #5119 — в Deferred conflicts; #4993 и #5009 — в Skipped до `backport/include`. Дальше пачку выводит из draft и мержит мейнтейнер (по #5246 и #5290 — за 18–50 минут), и следующий rc линии их заберёт. `[вывод]` Это и есть путь восемнадцати в v1.7.2.

### release-1.6 — двадцать четыре ручных PR: что с каждым

**Голова `release-1.6` — `bf4dfdd0` с 02.11**, релизов линии после v1.6.5 (16.10) нет. Пачка **#5312** (13.11, draft, 24 коммита, четыре оригинала — #5091, #5123, #5212, #5213) стоит с красным `Unit & controller tests`; в её теле 562 PR отсечены правилом 1, 23 — правилом 2, 6 ждут правила 6. Пока она не смержена, в линию не приедет ничего. Ручные PR в линию мы больше не подаём (см. «Требует действия»); ниже — что делать с теми 24, что уже открыты. Все 24 без ревью (кроме #4961), прогоны в `action_required`; для ветки это норма, не блокер.

**[#4961](https://github.com/cozystack/cozystack/pull/4961)** — оригинала в `main` нет, фикс только ветки

**Что чинит.** kubernetes: Job `talos-reconcile` на `release-1.6` рендерится чартом `kubernetes` и после каждого апгрейда уходит в CrashLoopBackOff по OOMKill на лимите 256Mi — воркерский `TalosConfigTemplate` тенантного кластера не применяется. Фикс — server-side `kubectl apply --server-side --force-conflicts` вместо client-side; в `main` то же сделал #4163, а его бэкпорт #4575 тронул только чарт `kubernetes-nodes`. Одна строка; `helm unittest` на ветке 193 из 193

**Что сделает батч в 1.6.** Батч не возит: оригинала в `main` нет; по политике такие PR открывают мейнтейнеры

**Судьба ручного PR.** **оставить и мержить руками** — `APPROVED` myasnikovdaniil 22.10

**[#4964](https://github.com/cozystack/cozystack/pull/4964)** — оригинал: #4963, смержен 13.11

**Что сделает батч в 1.6.** **возьмёт сам** после мержа пачки 1.7; на 1.6 чисто

**Судьба ручного PR.** **лишний — закрыть**

**[#4966](https://github.com/cozystack/cozystack/pull/4966)** — оригинал: #4965, смержен 13.11

**Что сделает батч в 1.6.** **возьмёт сам** после 1.7; чисто

**Судьба ручного PR.** **лишний — закрыть**

**[#4967](https://github.com/cozystack/cozystack/pull/4967)** — оригинал: #4289 (IvanHunters, до 26.10)

**Что чинит.** bucket (#4289 смержен в `main` 17.09) — hook очистки ACME ходил на `localhost:8080`, потому что `--request-timeout=30s` заставлял kubectl игнорировать внутрикластерные креды; каждый вызов отбивался, hook выходил с нулём, и `hook-succeeded` удалял Job вместе с логом. Плюс гейт `hack/check-hook-request-timeout.bats` на остальные шаблоны. В 1.6 не приезжал ни ботом, ни руками; ручной PR — один коммит, черри-пик без конфликтов с сохранением авторства

**Что сделает батч в 1.6.** **нужен лейбл** `backport/include` на #4289; на 1.6 чисто

**Судьба ручного PR.** Оставить до решения мейнтейнеров: лейбл или мерж ручного

**[#4987](https://github.com/cozystack/cozystack/pull/4987)** — оригинал: #4985, смержен 23.10 (до старта)

**Что сделает батч в 1.6.** **нужен лейбл** на #4985; #4985 в 1.7 — правило 6 пройдёт

**Судьба ручного PR.** Оставить до решения: лейбл или мерж ручного

**[#4988](https://github.com/cozystack/cozystack/pull/4988)** — оригинал: #4986, смержен 13.11

**Что сделает батч в 1.6.** **конфликт** на 1.6 в пяти файлах — Deferred

**Судьба ручного PR.** **оставить** — готовое разрешение конфликта; как его применить (на пачку по `resolve-conflict.md` или смержить PR и `--drop` оригинал) — решает мейнтейнер

**[#4991](https://github.com/cozystack/cozystack/pull/4991)** — оригинал: #4989, открыт (правки за нами)

**Что сделает батч в 1.6.** После мержа — **конфликт** (`vtraces.yaml`)

**Судьба ручного PR.** Оставить, ждёт оригинала

**[#4992](https://github.com/cozystack/cozystack/pull/4992)** — оригинал: #4990, открыт (правки за нами)

**Что сделает батч в 1.6.** После мержа — **конфликт** в трёх файлах; написан под ветку без `-x` — батч его за перенос не признает

**Судьба ручного PR.** Оставить, ждёт оригинала

**[#4994](https://github.com/cozystack/cozystack/pull/4994)** — оригинал: #4993, смержен 13.11, `feat`

**Что сделает батч в 1.6.** **нужен лейбл**; с ним на 1.6 чисто

**Судьба ручного PR.** Лишний после лейбла — закрыть тогда

**[#5001](https://github.com/cozystack/cozystack/pull/5001)** — оригинал: #4427 (IvanHunters, до 26.10)

**Что чинит.** monitoring-agents (#4427 смержен в `main` 24.09) — у kube-apiserver свой лимит размера скрейпа. Чинит `TargetDown` на живом apiserver: на v1.6.5 один таргет отдавал больше глобальных `promscrape.maxScrapeSize: 32MB`, другие на том же кластере были на 28.9 и 12.7 МБ. Три коммита, черри-пики без конфликтов; единственное отличие от исходного диффа — контекстная строка в `values.yaml`, ресурсов fluent-bit из #3495 на ветке нет

**Что сделает батч в 1.6.** **нужен лейбл** на #4427

**Судьба ручного PR.** Оставить до решения

**[#5006](https://github.com/cozystack/cozystack/pull/5006)** — оригинал: #5005, смержен 23.10 (до старта)

**Что сделает батч в 1.6.** **нужен лейбл** на #5005; #5005 в 1.7

**Судьба ручного PR.** Оставить до решения

**[#5007](https://github.com/cozystack/cozystack/pull/5007)** — оригинал: #3633 (lexfrei, до 26.10)

**Что чинит.** tenant (#3633 смержен в `main` 08.08) — релизы `monitoring` и `gateway` тенанта рендерятся с `remediation.retries: -1` без стратегии, упавшая установка не переустанавливается на месте. Апгрейдная сторона поймана на v1.6.5: одна hostname-constraint в платформенном `scheduling.globalAppTopologySpreadConstraints` перерендерила релиз monitoring в `tenant-root`, CNPG покатил `alerta-db` и `grafana-db`, и ревизии 18 и 19 упали по таймауту ожидания `HelmRelease/tenant-root/monitoring-system`. Один коммит, три файла байт в байт как в `a27c3418` на `main`

**Что сделает батч в 1.6.** **нужен лейбл** на #3633

**Судьба ручного PR.** Оставить до решения

**[#5010](https://github.com/cozystack/cozystack/pull/5010)** — оригинал: #5009, смержен 13.11, ломающий

**Что сделает батч в 1.6.** **нужен лейбл**; с ним на 1.6 чисто

**Судьба ручного PR.** Лишний после лейбла — закрыть тогда

**[#5012](https://github.com/cozystack/cozystack/pull/5012)** — оригинал: #5011, смержен 13.11

**Что сделает батч в 1.6.** **конфликт** на 1.6 (`e2e-install-cozystack.bats`, `external_tls_policy_test.go`)

**Судьба ручного PR.** **оставить** — готовое разрешение

**[#5026](https://github.com/cozystack/cozystack/pull/5026)** — оригинал: #4095 (IvanHunters, до 26.10) + `test` #5025

**Что чинит.** harbor (#4095 плюс тесты из #5025) — явные ресурсы nginx-прокси, чтобы он переживал LimitRange тенанта. На ветке прокси рендерится без `resources`, получает дефолт LimitRange 250m и 128Mi, и на кластере v1.6.5 оба пода nginx дефолтного Harbor в тенантном неймспейсе ушли в OOMKilled с 5 и 6 рестартами за минуты. Три коммита — черри-пик `146f711b` из #4095 и два из #5025, без конфликтов, с `cherry picked from`

**Что сделает батч в 1.6.** **нужен лейбл** на оба

**Судьба ручного PR.** Оставить до решения

**[#5041](https://github.com/cozystack/cozystack/pull/5041)** — оригинал: #5040, открыт (правки за нами), `feat`

**Что сделает батч в 1.6.** После мержа — **нужен лейбл**; чисто

**Судьба ручного PR.** Оставить, ждёт оригинала и лейбла

**[#5095](https://github.com/cozystack/cozystack/pull/5095)** — оригинал: #5094, смержен 13.11

**Что сделает батч в 1.6.** **возьмёт сам** после 1.7; чисто

**Судьба ручного PR.** **лишний — закрыть**

**[#5099](https://github.com/cozystack/cozystack/pull/5099)** — оригинал: #5098, смержен 13.11

**Что сделает батч в 1.6.** **возьмёт сам** после 1.7; чисто

**Судьба ручного PR.** **лишний — закрыть**

**[#5101](https://github.com/cozystack/cozystack/pull/5101)** — оригинал: #5100, смержен 13.11

**Что сделает батч в 1.6.** **возьмёт сам** после 1.7; чисто

**Судьба ручного PR.** **лишний — закрыть**

**[#5103](https://github.com/cozystack/cozystack/pull/5103)** — оригинал: #5102, смержен 13.11

**Что сделает батч в 1.6.** **возьмёт сам** после 1.7; чисто

**Судьба ручного PR.** **лишний — закрыть**

**[#5111](https://github.com/cozystack/cozystack/pull/5111)** — оригинал: #5110, открыт, `feat`

**Что сделает батч в 1.6.** **нужен лейбл** и **конфликт** в `cozyrds/monitoring.yaml` на обеих линиях

**Судьба ручного PR.** Оставить — единственный готовый путь в 1.6

**[#5115](https://github.com/cozystack/cozystack/pull/5115)** — оригинал: #5114, смержен 13.11

**Что сделает батч в 1.6.** **возьмёт сам** после 1.7; чисто

**Судьба ручного PR.** **лишний — закрыть**

**[#5120](https://github.com/cozystack/cozystack/pull/5120)** — оригинал: #5119, смержен 13.11

**Что сделает батч в 1.6.** **конфликт** (`postgres.yaml`) на обеих линиях

**Судьба ручного PR.** **оставить** — готовое разрешение

**[#5122](https://github.com/cozystack/cozystack/pull/5122)** — оригинал: #5121, открыт (правки за нами)

**Что сделает батч в 1.6.** После мержа — **конфликт** (`cozyrds/mongodb.yaml`); перенесена только чартовая часть

**Судьба ручного PR.** Оставить, ждёт оригинала

Итого: **семь лишних сейчас** (#4964, #4966, #5095, #5099, #5101, #5103, #5115), **два — после лейбла** (#4994, #5010), **три — готовые разрешения конфликтов** (#4988, #5012, #5120), **пять ждут оригинала** (#4991, #4992, #5041, #5111, #5122), **шесть ждут решения по лейблу на оригиналы до 26.10** (#4967, #4987, #5001, #5006, #5007, #5026), **один — фикс ветки** (#4961). Если ручной PR с `-x`-пиками всех коммитов оригинала смержат раньше батча, батч признает оригинал «уже в линии» и не тронет; закрывать лишние можно не дожидаясь — батч довезёт сам.

### Бэкпорты до 09.11 — архив

До мержа #5106 (09.11) бэкпорты возил лейбловый бот (`kind/backport` → новейшая линия, `kind/backport-previous` → предыдущая) и ручные PR, которые релиз-менеджер мержил пачками перед rc без ревью и зелёного CI: 14.10 (десять наших), 16.10, 22.10, 23.10 и 02.11 (#5089) — последние три без единого нашего. Пре-релизным гейтом был `cmd/backport-audit`, у которого сквошенные ручные бэкпорты давали ложный PARTIAL. Всё это удалено из `main` 09.11; что из этого ещё важно для механики — в CLAUDE.md. Форк в `release-1.6` зелёный `E2E Tests` получить не может: в дереве ветки нет `OCI_EXPORT_DIR` и `e2e-fork.yaml`.

## Чужие PR — открытые

Один PR. #5106 выбыл 09.11 — смержен в `main` в 04:42:39 и переписал механику бэкпортов: лейблы мертвы, бэкпорты возит батч, первые пачки #5246 и #5247 открыты в то же утро; строка перенесена в «Смержено в main». #4897 выбыл 04.11 — закрыт автором без мержа в пользу proposal community#116 с прототипом #5107, запись в «Закрытых без мержа». #5089 выбыл 02.11 — пачка смержена в `release-1.6` в 09:42:37 и стала головой ветки `bf4dfdd0`. #5141 выбыл 30.10 — смержен в `release-1.6` и расчистил красные юнит-тесты всей линии. #4817 взят в трекинг 10.10 — реализация per-cluster etcd по community#25, в обсуждении которого принято наше предложение минимального API.

| PR | Автор | Название | Что решает | Состояние |
|---|---|---|---|---|
| [#4817](https://github.com/cozystack/cozystack/pull/4817) | lllamnyp | feat(kubernetes): give each tenant Kubernetes cluster its own etcd | Тенантный кластер больше не ждёт, пока у предка появится `spec.etcd: true`, и не делит один etcd, один CA и один диск со всеми соседями по поддереву: у каждого нового кластера свой `EtcdCluster` и Kamaji `DataStore`, создаются и удаляются вместе с кластером, состояние `awaiting-etcd` исчезает. На CR одно поле `etcd.replicas` — 3 или 1 | Открыт 08.10 в 10:30, **draft**, **`CHANGES_REQUESTED` от lexfrei 12.10 в 13:31** — «NOT LGTM», четыре блокера: ветка конфликтует с main в трёх файлах (`mergeable: false`); на голове красный `Unit & controller tests` от 08.10, из-за него `E2E (in-tree)` пропущен и живые кейсы не гонялись, падают `hack/kubernetes-pre-delete-hook.bats` и `hack/check-render-matrix.bats`; CronJob `kubernetes-<name>-etcd-defrag` при имени кластера 30–32 символа выходит за апишный потолок 52 символа и кластер не устанавливается; `podscrape.yaml` по `app.kubernetes.io/name: etcd` заставляет общий etcd тенанта скрейпить ещё и etcd кластеров своего неймспейса, и `etcdInsufficientMembers` видит 4 из 6 вместо 2 из 3. B1 упирается в наш #4873: `--compaction=false` и расчёт от реальной квоты надо занести в `cozy-lib`, иначе перенос CronJob молча откатит фикс. **14.10 в 04:45 второе ревью — `COMMENTED` от themoriarti**, с оговоркой «это комментарий, не голос»: все четыре блокера перепроверены на той же голове и подтверждены, плюс пять находок — на кластере с развалившимся etcd падение на клине удаления заставит Flux ретраить вечно, живой класс хранилища побеждает и при переходе с трёх членов на одного, VPA с `updateMode: Initial` пересчитывает размер каждого нового члена, `fromEndpoints` в сетевой политике матчит только поды. Фазы 1 и 2 из community#25; фазы 3 и 4, бэкап собственного etcd и opt-in миграция — в follow-up |

## Живая миграция VM — вся связка в main

15.09 yankawai открыл пачку из трёх PR вокруг одной темы — чтобы VM переезжали между хостами живьём, а не убивались. 18.09 в main доехал последний из них.

| PR | Слой | Что даёт | Состояние |
|---|---|---|---|
| #4254 | платформа → KubeVirt | настраивать миграции платформенными values | **смержен 18.09** |
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
| правки по ревью lexfrei от 13.11 — за yankawai; ни на одном нет апрува, пушить можно | #4947, #4989, #4990, #5040, #5121, #5097 |
| ребейз после конфликта с `main` (за yankawai), затем возврат lexfrei к правкам 09.11 | #4875 |
| «Approve and run workflows» и первое ревью — 13.11 их не коснулись | #5110, #5183, #5189 |
| мерж пачки 1.7 после прогона 17.11 и разрешение двух конфликтов на ней | #5311 (#5096, #5119) |
| `backport/include` — иначе батч не возьмёт ни в одну линию | #4993, #5009, #5025; позже #5040, #5110 |
| `backport/include` на оригиналы до 26.10 — или мерж ручных как есть | #4985, #5005, #4095, #4289, #4427, #3633 (ручные #4987, #5006, #5026, #4967, #5001, #5007) |
| красный `Unit & controller tests` и мерж пачки 1.6 | #5312 |
| мерж руками — фикс только ветки, `APPROVED` 22.10 | #4961 |
| ответ автора lllamnyp'у | #5107 |
| правки автора по ревью lexfrei от 12.10 | community#25 |
| четыре блокера по ревью lexfrei, конфликт с main, вывод из draft | #4817 |

Из открытого в `main` наших десять — #4875, #4947, #4989, #4990, #5040, #5097, #5110, #5121, #5183, #5189; у шести правки за нами, у одного ребейз, три ждут первого прогона и ревью. В `release-1.6` наших открыто двадцать четыре, из них семь уже лишние (батч довезёт сам), ещё два станут лишними после лейбла.

## Связь PR и issue

`Fixes #NNNN` в теле PR — стандартный синтаксис GitHub: при мерже указанный issue закрывается автоматически. Четыре issue #4083–#4086 завёл yankawai 05.09, 07.09 выкатил на них PR — **18.09 закрыт последний из четырёх**.

| Issue | Название | PR | Состояние |
|---|---|---|---|
| [#5002](https://github.com/cozystack/cozystack/issues/5002) | Tenant namespace deleted directly is stuck Terminating on Kamaji finalizers | #5011 | **закрыт 13.11 в 15:25** мержем #5011 — по `Fixes #5002` в теле |
| [#5004](https://github.com/cozystack/cozystack/issues/5004) | Memory without a unit in app resources is applied as bytes | #5009 | **закрыт 13.11 в 15:25** мержем #5009 — по `Fixes #5004` в теле |
| [#3759](https://github.com/cozystack/cozystack/issues/3759) | Tenant admins cannot upload images into VM disks (CDI upload forbidden by tenant RBAC) | #5114 | **закрыт 13.11 в 15:12** мержем #5114 — по `Fixes #3759` в теле; вторая половина (панель загрузки в консоли и сертификат upload-прокси) остаётся в чужем #3847 |
| [#3832](https://github.com/cozystack/cozystack/issues/3832) | bug(backups): MongoDB to-copy restore leaves the target cluster unauthenticatable | #5121, #5122 | OPEN, завёл lexfrei. В теле #5121 (открыт 26.10 в 17:53) написано `Fixes #3832`, так что при мерже issue закроется сам; бэкпорт #5122 ссылается на него через `Refs` |
| [#5248](https://github.com/cozystack/cozystack/issues/5248) | kubevirt-csi leaves hotplug volumes attached when the infra PVC is missing | #5249 | **закрыт 13.11 в 12:41** мержем #5249 — по `Fixes` в теле; заведён yankawai 09.11 |
| [#4006](https://github.com/cozystack/cozystack/issues/4006) | registries.mirrors has no effect on Talos workers | #4897 → #5190 | **Закрыт 09.11 в 14:36:33** мержем #5190 myasnikovdaniil — того самого отдельного PR, что был обещан при закрытии #4897: мёртвое значение `registries` и Secret patch-containerd удалены из платформы, зеркала воркеров — через `talos.registryMirrors` у KubernetesNodes. Просьба kvaps от 20.10 — закрыть дыру и на `release-1.6` или хотя бы предупреждать на рендере — осталась без PR |
| [#4083](https://github.com/cozystack/cozystack/issues/4083) | clickhouse: Keeper is never created when the application name is longer than 15 characters | #4133 | **закрыт** мержем 17.09 |
| [#4084](https://github.com/cozystack/cozystack/issues/4084) | nats: config.merge.accounts duplicates the generated accounts key and breaks the install | #4134 | **закрыт** мержем 18.09 |
| [#4085](https://github.com/cozystack/cozystack/issues/4085) | harbor/clickhouse: cleanup hook Role lacks watch on persistentvolumeclaims | #4135 | **закрыт** мержем 18.09 |
| [#4086](https://github.com/cozystack/cozystack/issues/4086) | opensearch: data PVCs are left behind after the application is removed | #4136 | **закрыт** мержем 18.09 |
| [#3740](https://github.com/cozystack/cozystack/issues/3740) | networking: cilium takes its per-node infrastructure IPs from the pod CIDR kube-ovn allocates pods from | #4778 | **закрыт** мержем 08.10 в 18:41 — по `fixes` в теле PR |
| [#4676](https://github.com/cozystack/cozystack/issues/4676) | bug(dashboard): NATS free-form configuration fields have no editor in Form mode | #4682 | **закрыт** мержем 08.10 в 23:21 — по `Fixes` в теле PR |
| [#2483](https://github.com/cozystack/cozystack/issues/2483) | OpenBAO: can't init/unseal without kubectl exec | #4766 | **всё ещё открыт** — в теле #4766 упомянут без закрывающего слова, автоматом не закрылся. Решён смерженным 08.10 #4766, закрыть руками; 06.11 myasnikovdaniil закрыл парный issue #2793, этот не тронул |
| [#2793](https://github.com/cozystack/cozystack/issues/2793) | OpenBAO HA (raft) pods hang at startup due to hardcoded service_registration "kubernetes" | #4766 | **закрыт 06.11 в 07:55** myasnikovdaniil вручную, с комментарием-ссылкой на #4766 |
| [#3950](https://github.com/cozystack/cozystack/issues/3950) | Platform-wide defaults for tenant Talos worker settings | #4897 → community#116, #5107 | OPEN. **04.11 #4897 закрыт автором без мержа**: тот же случай покрывает proposal community#116 — дефолты установки одним общим путём для любого приложения — с прототипом draft #5107 (оба открыты 26.10). `[вывод]` Автоматически issue теперь не закроется — `closes #3950` был в теле закрытого PR; у #5107 в теле ссылка на issue без закрывающего слова. #5107 одобрен 09.11 и `CLEAN`, но 11.11 lllamnyp попросил доспецифицировать поведение (community#116 смержен 04.11) — мерж ждёт ответа myasnikovdaniil. Шов на write-пути #3956 смержен 07.10, каталожная сторона #4171 — 24.09 |
| [#4345](https://github.com/cozystack/cozystack/issues/4345) | monitoring: vmalert notifies only one of the three Alertmanager replicas | #4889 | **закрыт 06.11 в 07:56** myasnikovdaniil вручную, с комментарием-ссылкой на #4889 |
| [#4704](https://github.com/cozystack/cozystack/issues/4704) | apps: CA and TLS Secrets with private keys stay in the tenant namespace after the app is deleted | #4875 | OPEN. В теле #4875 написано `Addresses #4704` — это не закрывающее слово, `[вывод]` автоматом issue не закроется и после мержа |

## Issues

| Issue | Автор | Название | Состояние |
|---|---|---|---|
| [#5070](https://github.com/cozystack/cozystack/issues/5070) | yankawai | Default tenant Kubernetes control plane does not fit a tenant with resourceQuotas.cpu 24 and deadlocks on creation | OPEN с 22.10 в 11:46:23, `triage/needs-triage`, комментариев нет. На v1.6.5 тенанта создали с `resourceQuotas` `cpu: 24`, `memory: 24Gi`, `storage: 50Gi` и `etcd: true`, затем приложение Kubernetes с дефолтами чарта плюс `addons.ingressNginx.enabled: true` — через 16 часов у кластера нет ни одной ноды. Чарт тенанта отрендерил `tenant-quota` с `requests.cpu: 2400m` (это `cpu`, делённое на платформенный cpu-allocation-ratio 10), а дефолтный кластер в покое просит 2145m: 360m на каждую из двух реплик control plane, 875m на семь контейнеров kcsi-controller, по 125m на kccm и cluster-autoscaler и 300m на три члена тенантного etcd. Девять контейнеров с фиксированными 125m, которых ratio не касается, дают 1125m — почти половину квоты. Deployment control plane получил два ReplicaSet в одну секунду, потому что Kamaji создаёт его без konnectivity-server и дописывает контейнер вторым шагом, а `spec.deployment.strategy` чарт не задаёт и Kamaji берёт свой дефолт `maxSurge: 100%`/`maxUnavailable: 0` — последний старый под держится, пока не поднимутся два новых, второй новый в квоту не влезает, и раскатка не завершается никогда. Отсюда и всё остальное: `ReplicaFailure` на Deployment, `Provisioning` у TenantControlPlane, `initialized: false` у KamajiControlPlane, MachineSet md0 без Machine с `ControlPlaneIsStable preflight check failed`, HelmRelease cilium в `UpgradeFailed` и четыре аддона в `DependencyNotReady`. Квоту не называет ни один из этих статусов — её видно только в condition Deployment'а. Апстримный clastix/kamaji#1372 рендерит konnectivity-server в первой ревизии, смержен в master 21.10 и в тег ещё не вошёл. В теле четыре предложения: привести фиксированные 125m к ratio или сделать их значениями чарта (апстрим kubevirt/csi-driver просит не больше 10m на контейнер, 125m пришли в чарт из #698 и #2614), задать `spec.deployment.strategy` или оставить дефолт и заложить в квоту место ещё под один под control plane, абзац в README тенанта про деление на ratio и реальный запрос дефолтного кластера, показ ошибки квоты в статусе приложения Kubernetes (issue #2733). Отдельно: дизайн резерваций в community#48 с фазой 1 в чужом #4983 (**смержен в `main` 10.11 в 00:26**) снимает проблему целиком, но его таблица накладных расходов считает CSI четырёхконтейнерным, а на v1.6.5 и на main у kcsi-controller семь контейнеров; #4817 добавил бы в этот же счёт членов etcd каждого нового кластера. Побочное наблюдение — около 12,6k no-op `update` в час от capi-kamaji на KamajiControlPlane и 6,3k на TenantControlPlane. Подписки нет |
| [#5002](https://github.com/cozystack/cozystack/issues/5002) | yankawai | Tenant namespace deleted directly is stuck Terminating on Kamaji finalizers | **закрыт 13.11 в 15:25** мержем #5011 — по `Fixes #5002` в теле |
| [#5004](https://github.com/cozystack/cozystack/issues/5004) | yankawai | Memory without a unit in app resources is applied as bytes | **закрыт 13.11 в 15:25** мержем #5009 — по `Fixes #5004` в теле |
| [#4962](https://github.com/cozystack/cozystack/issues/4962) | yankawai | Upgrade image pulls stall the management etcd on Talos nodes | OPEN с 19.10 в 16:41:41, `triage/needs-triage`, комментариев нет. От мейнтейнеров нужен выбор варианта: pre-pull в гайде по апгрейду или механизм, который тянет образы с релизом. Выкачка образов релиза при апгрейде v1.6.4 → v1.6.5 роняет управляющий etcd в перевыборы лидера: на первом кластере четыре перевыбора (термы 272–275) за пять минут после `helm upgrade`, `slow fdatasync` до 5.0 с и `leader failed to send out heartbeat on time`, позже одной выкачки `piraeus-server` (~370 МБ) на ноде-лидере хватило ещё на одни выборы. Кворум держался каждый раз, но записи в API встают на каждых выборах посреди раскатки платформы. На втором кластере 25 изменившихся образов вытянули заранее через `talosctl image pull --namespace cri`, по одной ноде, лидер последним — ни одних выборов ни при установщике, ни при hook миграции IPAM Cilium, ни при роллинге платформы и сателлитов LINSTOR. Среда: Talos 1.13.6, etcd 3.6.12, системный диск общий с хранилищем образов containerd, три control-plane-ноды. **Два вопроса мейнтейнерам**: описать pre-pull в гайде по апгрейду (эту часть автор готов прислать сам) или везти с релизом механизм, который тянет образы на все ноды по одной до раскатки HelmRelease |
| [#3734](https://github.com/cozystack/cozystack/issues/3734) | IvanHunters | vm-instance releases never converge: kstatus observedGeneration pre-check gates healthy KubeVirt VMs | **Закрыт 24.09** lexfrei мержем #3735 (`Closes #3734`) — но только в `main`; `[вывод]` в линии 1.7 фикс тоже есть, потому что `release-1.7` отрезана от main позже 24.09. HelmRelease VMInstance не сходится, хотя VM работает: helm-controller ждёт, пока `observedGeneration` догонит `generation`, а KubeVirt не поднимает его после изменения одних метаданных шаблона — например, аннотаций kube-ovn с IP и MAC, которые чарт добавляет при повторном рендере. В issue lfinmauritius подтвердил баг на v1.6.1, а ghostrider0470 разобрал причину с аннотациями kube-ovn. **02.11 фикс приехал и в 1.6 — пачка #5089 принесла #3735 в ветку** (`fbf69f7aa`, проверено по дереву); наш бэкпорт #5003 стал лишним и конфликтует |
| [#3950](https://github.com/cozystack/cozystack/issues/3950) | IvanHunters | Platform-wide defaults for tenant Talos worker settings | OPEN, **23.09 lexfrei принял в работу**: `triage/accepted` и `priority/important-longterm` вместо `triage/needs-triage`. Смерженный 24.09 #4171 закрыл каталожную сторону — платформенный ключ `kubernetesWorkerImage` позволяет направить импорт golden-образа в зеркало, — но потребляющая сторона дефолтов (`imageFactoryURL`, `installerRepository`, `registryMirrors` на `KubernetesNodes`) осталась. В треде готовый разбор: наш комментарий 07.09 о create-time дефолтинге, ответ myasnikovdaniil 08.09 (двум полям платформенный дефолт безопасен, `imageFactoryURL` — прокат флота), предупреждение lexfrei 01.09 о невозможности plain default chain в Helm. Шов на write-пути — #3956, **смержен 07.10**. **13.10 в 06:40 myasnikovdaniil открыл #4897 с потребляющей стороной** — `kubernetesWorkerImage` стал дефолтом нового `KubernetesNodes`, добавлены `installerRepository` и `kubernetesRegistries`, подстановка только на create, как мы и предлагали 07.09. В теле `closes #3950`. 13.10 #4897 получил `CHANGES_REQUESTED` от IvanHunters и lexfrei, 14.10 вернулся в draft. **04.11 в 06:29:09 #4897 закрыт автором без мержа** в пользу proposal community#116 (installation defaults одним общим путём для любого приложения) с прототипом draft #5107 от 26.10. `[вывод]` Закрывать issue теперь будет путь community#116/#5107, и автоматического `closes` на нём нет. Сам community#116 смержен 04.11 в 08:35; #5107 одобрен IvanHunters 09.11 и `CLEAN`, но 11.11 в 13:43 lllamnyp попросил доспецифицировать поведение дополнением к proposal и в документации — мерж ждёт ответа myasnikovdaniil |
| [#4345](https://github.com/cozystack/cozystack/issues/4345) | ghostrider0470 | monitoring: vmalert notifies only one of the three Alertmanager replicas, so two thirds of queries for firing alerts come back empty | **Закрыт 06.11 в 07:56:46** myasnikovdaniil с комментарием: #4889 — vmalert шлёт алерты каждой реплике Alertmanager напрямую, реплик столько же, сколько у ресурса Alertmanager. Был OPEN с 18.09, `triage/accepted`, `priority/important-longterm`; решён нашим #4889, смерженным 14.10, автоматом не закрылся — в теле PR issue не был упомянут |
| [#3022](https://github.com/cozystack/cozystack/issues/3022) | lexfrei | OpenSearch fails to start in tenant namespaces: privileged init-sysctl violates baseline PodSecurity | **Закрыт 06.11 в 07:56:06** myasnikovdaniil с комментарием: поды тенантного OpenSearch больше не просят привилегированный sysctl-init (#2682), сисктл на нодах держит системный DaemonSet из #4152 — ровно то, что автор #4026 и предлагал |
| [#2483](https://github.com/cozystack/cozystack/issues/2483) | Arsolitt | OpenBAO: can't init/unseal without kubectl exec | OPEN, `triage/accepted`, `kind/feature`. Решён смерженным 08.10 #4766: opt-in `seal.type: static` снимает ручной `bao operator unseal` после каждого рестарта. В теле PR issue упомянут без закрывающего слова, поэтому автоматом не закрылся — нужен мейнтейнер. 06.11 myasnikovdaniil закрыл парный issue #2793, этот остался открытым |
| [#2793](https://github.com/cozystack/cozystack/issues/2793) | myasnikovdaniil | OpenBAO HA (raft) pods hang at startup due to hardcoded service_registration "kubernetes" | **Закрыт 06.11 в 07:55:58** myasnikovdaniil с комментарием: #4766 даёт подам OpenBAO server лейбл egress к apiserver, существующие поды после апгрейда надо пересоздать, чтобы лейбл получить |
| [#4704](https://github.com/cozystack/cozystack/issues/4704) | lexfrei | apps: CA and TLS Secrets with private keys stay in the tenant namespace after the app is deleted | OPEN с 02.10, `triage/needs-triage`, за всё время ни одного комментария. Платформенный cert-manager работает с `enableCertificateOwnerRef: false`, поэтому Secret с приватным ключом переживает и свой Certificate, и релиз, а приложение с тем же именем потом переиспользует найденный ключ. Решается нашим #4875 |
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
| [cozystack/community#25](https://github.com/cozystack/community/pull/25) | myasnikovdaniil | [proposal] Per-cluster etcd | OPEN, **`CHANGES_REQUESTED` от lexfrei 12.10 в 13:24** — первое ревью мейнтейнера за всё время, шесть пунктов и шесть инлайн-замечаний: дискриминатором нового кластера должен быть `spec.dataStoreName` живого `KamajiControlPlane`, а не всегда заполненный `_namespace.etcd`; имя `DataStore` через дефис не уникально; платформенного бэкапа у per-cluster etcd нет; сетевая политика etcd в неймспейсе не изолирует; фаза 4 снимает гейты у 26 шаблонов аддонов; **пункт 6 просит внести в runbook наш прогон от 03.09** — достижимость из неймспейса кластера, канареечная запись вместо Ready, путь к Talos API воркеров. Ревью IvanHunters от 04.07 помечено `DISMISSED`. Наше предложение минимального API принято целиком: на CR только `etcd.replicas`. **03.09 отправили отчёт о прогоне миграции** — ответа нет. **08.10 lllamnyp открыл реализацию — draft #4817** на фазы 1 и 2, с тем же `etcd.replicas` на CR. `[вывод]` Документ теперь догоняет код: апрув proposal'а стал срочнее. **На 15.10 автор на ревью не ответил** — последняя активность на PR 12.10 в 13:24 |

### Наш отчёт по community#25

Живой прогон миграции datastore на кластере с Talos-воркерами (CozyStack v1.6.1, Kamaji v1.6.1-rc.1, три воркера KubeVirt). Механизм Kamaji работает и быстр: копирование около 20 с, от патча до Ready около 43 с, ноль отказов на чтение за всю сессию. Три находки:

1. **Направление миграции ограничено топологией сети тенанта.** CP-поды живут в неймспейсе кластера, и OVN пропускает только к etcd предка, не соседа. При переключении на соседа apiserver умирает на таймауте, кластер заморожен на запись. Для per-cluster etcd в том же неймспейсе это не проблема, но runbook должен назвать правило достижимости.
2. **Webhook заморозки может пережить успешную миграцию.** Лизы в `kube-node-lease` из-под него выведены, поэтому ноды остаются Ready, а кластер при этом заблокирован на запись. Критерий успеха для e2e — канареечная **запись**, а не статус Ready.
3. **Kubelet'ы глохнут к новым назначениям, пока их не перезапустить.** Новый под висит `Pending` с назначенной нодой (наблюдали 16 минут). Перезапуск дешёвый: Talos делает graceful cordon, канарейка не потеряла ни одной записи на трёх ребутах.

Отдельно: у оператора нет пути к Talos API воркеров тенанта — чарт заводит per-cluster `talos-ca`, но клиентских кредов не выпускает.

## Закрытые без мержа

| PR | Автор | Что было | Итог |
|---|---|---|---|
| [#5003](https://github.com/cozystack/cozystack/pull/5003) | yankawai | Ручной бэкпорт #3735 (IvanHunters) в `release-1.6`: снятие ожидания готовности у релиза VMInstance | Закрыт автором 04.11 в 22:58 как лишний: пачка #5089 02.11 принесла #3735 в линию как `fbf69f7aa`, по комментарию автора все девять файлов ветки совпадают с `release-1.6` побайтно |
| [#4897](https://github.com/cozystack/cozystack/pull/4897) | myasnikovdaniil | Платформенные дефолты Talos-воркеров тенанта: `kubernetesWorkerImage`, `installerRepository`, `kubernetesRegistries`, подстановка на create; в теле `closes #3950` и `closes #4006` | Закрыт автором 04.11 в 06:29:09 без мержа: тот же случай покрывает proposal community#116 — дефолты установки одним общим путём для любого приложения. Прототип — draft #5107 от 26.10. TLS-настройки зеркал и мёртвое значение `registries` (issue #4006) — отдельными PR; применение платформенных зеркал к существующим пулам не переносится |
| [#4876](https://github.com/cozystack/cozystack/pull/4876) | yankawai | E2E leak-guard на Secret'ы cert-manager отдельным PR | Закрыт автором 11.10 в 19:03, через 40 минут после открытия: коммит влит в #4875, чтобы проверка не приехала раньше фиксов, которые она проверяет |
| [#4609](https://github.com/cozystack/cozystack/pull/4609) | github-actions[bot] | Ботовский бэкпорт #3800 в release-1.6 по лейблу `kind/backport` | Закрыт 10.10 в 20:05 lexfrei как дубль ручного #4614: ботовская ветка несла только маркеры конфликта, из-за них же DCO висел в `action_required` |
| [#2321](https://github.com/cozystack/cozystack/pull/2321) | officialasishkumar | Первый фикс tcp-balancer для HAProxy 3.3+ | Закрыт 29.09 lexfrei как superseded: доведён смерженным #4366, где авторство переименования сохранено; в комментарии закрытия — благодарность автору |
| [#4026](https://github.com/cozystack/cozystack/pull/4026) | yankawai | Ручка `setVMMaxMapCount` для OpenSearch | Закрыт автором 17.09: main обогнал — #4152 поднимает sysctl DaemonSet'ом, #2682 выключил `setVMMaxMapCount`. Ребейз вернул бы дефолт `true`, обратный смерженному |
| [#3357](https://github.com/cozystack/cozystack/pull/3357) | fuad00 | DNS_BASE для opensearch-operator | Закрыт 10.09: коммит с сохранением авторства довели через #4185, маршрут изменён на `opensearch-operator.manager.dnsBase` |
| [#3294](https://github.com/cozystack/cozystack/pull/3294) | myasnikovdaniil | Golden Talos image, первая версия | Смержен 07.09 в ветку `test/drop-ghcr-mirror`, до main не доехал. Работа доведена в #4171 — смержен 24.09 |
| [#4007](https://github.com/cozystack/cozystack/pull/4007) | myasnikovdaniil | Удаление ghcr.io pull-through mirror из e2e | Закрыт 09.09 как пустой: #4020 уже всё удалил |
| [#3949](https://github.com/cozystack/cozystack/pull/3949) | IvanHunters | Поля `talos.*` необязательными в схеме | Закрыт 24.08 |
| [#2751](https://github.com/cozystack/cozystack/pull/2751) | SerjioTT | SMTP для Grafana | Закрыт 13.08 stale-ботом, осознанно: #3800 решает задачу на правильном слое |

## Смержено в main

PR из отслеживаемого скоупа, которые уже в `main` своего репозитория. Часть из них подробнее описана выше.

**28.09 вышел патч-релиз [v1.6.4](https://github.com/cozystack/cozystack/releases/tag/v1.6.4)** — 19 отслеживаемых PR доехали до пользователей release-1.6 бэкпортами (с учётом добавленного задним числом #4220); всего в notes релиза 60 пунктов. В контрибьюторах yankawai отмечен как «First contribution». #4366 и #4171 в notes отсутствуют — `[вывод]` смержены в main позже отсечки и поедут следующим релизом. #2682, #4185, #4152 и #4095 не входят ни в один релиз 1.6.x — ждут минорного.

**29.10 вышел стабильный v1.7.0** — промоут v1.7.0-rc.3, тег на `a32334ae`. Теги rc.1 и rc.2 — его предки, поэтому всё, что в колонке «Релиз» помечено v1.7.0-rc.N, есть и в v1.7.0.

**11.11 в 08:26 вышел v1.7.1-rc.1** — первый релиз после v1.7.0, тег на `3cd9d4c2`, merge-коммите пачки #5246: наши #4985 и #5005 в нём. **12.11 в 12:38 вышел v1.7.1-rc.2** — тег на `12705a1c`, merge-коммите пачки #5290; сверх rc.1 в нём один чужой #5284.

**16.10 в 10:40 вышел стабильный v1.6.5** — промоут v1.6.5-rc.3 через #4937, смерженный myasnikovdaniil в 10:38; тег на `908b107a`, голове `release-1.6`. Перед этим: 14.10 v1.6.5-rc.1 (15:30, `6688ddf3` — мерж #4918) и v1.6.5-rc.2 (20:02, `234dfdaa`), 16.10 в 07:31 v1.6.5-rc.3 (`5a7531a3` — мерж нашего #4516). В notes v1.6.5 явно названы 15 отслеживаемых PR, #4789 и #4812 приехали внутри #4684 и #4740. Последний pre-release линии 1.7 — v1.7.0-rc.1 от 12.10; **ветка `release-1.7` на репозитории появилась** — её голова всё тот же коммит `38d6ac7e` от 12.10 07:53, в ветку пока ничего не смержено, и у нас в неё открыт бэкпорт #4943 для #4889.

**21.10 прошла первая пачка в `release-1.7`** и забрала наш единственный открытый PR в этой ветке: #4943 (бэкпорт #4889) смержен myasnikovdaniil в 07:02:51, merge-коммит `a739d04d`. Вместе с ним в те же полминуты уехали чужие ботовские #4935 и #4996, а 20.10 в 12:21:55 — ботовский #4995. Ни ревью, ни зелёных обязательных прогонов: у #4943 статус `E2E Tests` на голове был красным. Голова `release-1.7` на тот момент — `37c82f30`. По линии 1.7 после v1.7.0-rc.1 от 12.10 вышли v1.7.0-rc.2 22.10 и v1.7.0-rc.3 27.10, так что #4943 уже у пользователей rc.

**23.10 в `main` уехали три наших PR** — первые после #4889 от 14.10. myasnikovdaniil смержил #5025 в 17:40:29 (merge-коммит `61484797`), #4985 в 17:40:34 (`432d02e2`) и #5005 в 18:53:40 (`55aad6f9`, голова `main`). Все три он одобрил 22.10 в 18:48–18:49, «Approve and run workflows» на них сделали 23.10, и обязательные `pre-commit` и `E2E Tests` на их головах зелёные. В линию 1.6 они переносятся нашими открытыми #4987, #5006 и #5026; backport-лейблов на них нет, поэтому в 1.7 бот бэкпортов не открывал.

**Пачка бэкпортов перед 1.6.5 прошла 14.10** и забрала десять наших из одиннадцати: #4683, #4677, #4725, #4739, #4740, #4780, #4874, #4684, #4614 (07:34:26–07:35:45) и #4890 (13:30:15), все — myasnikovdaniil, ни у одного не было ни ревью, ни зелёных обязательных прогонов. Туда же уехали ботовские #4905 и #4906 — бэкпорты наших #4682 и #4766, созданные по лейблу `kind/backport-previous`, — и ручной #4918 для #4778. Проверено локально: все они предки rc.2. **Последний наш бэкпорт #4516 смержен 16.10 в 06:46:45**, его merge-коммит `5a7531a3` стал головой ветки и тегом v1.6.5-rc.3; открытых бэкпортов у нас больше нет.

**12.10 в 05:57 в main уехал #4873** — открыт 11.10 в 17:25, апрув lexfrei в 04:56, мерж им же через час, все проверки на голове зелёные. Бэкпорт в release-1.6 был подан заранее, 11.10 в 17:26 — это #4874.

**08.10 в main уехали последние семь наших открытых PR** — #4812, #4789, #4778, #4779, #4724, #4766, #4682, все смержены lexfrei с зелёными `pre-commit` и `E2E Tests`. Бэкпорты для всех семи теперь в ветке: #4725 и #4780 были поданы заранее, #4682 и #4766 забрал бот по лейблу (#4905 и #4906), #4778 забэкпортил руками myasnikovdaniil (#4918), содержимое #4812 уехало внутри #4740. Без бэкпорта остался только #4789 — черри-пик конфликтует в одном тесте.


**13.11 lexfrei смержил восемнадцать наших PR за один день** — с 12:14 (#5100) до 15:25 (#5009, #5011), все merge-коммитом после того, как сам влил `main` в каждую ветку и дождался зелёных прогонов. **13.11 в 12:54 вышел стабильный v1.7.1** — промоут v1.7.1-rc.2, тег `c0b53abb`; из нашего в нём #4985 и #5005, восемнадцать мержей того же дня снят после rc и поедут в линию батчем с прогона 17.11.
| PR | Автор | Что решил | Смержен | Релиз |
|---|---|---|---|---|
| [#4945](https://github.com/cozystack/cozystack/pull/4945) | yankawai | apps: имена Job и CronJob в лимитах Kubernetes | **13.11 в 15:18** lexfrei, merge-коммит, после слияния `main` в ветку | в v1.7.1 **не вошёл** (rc.2 снят 12.11); в `release-1.7` поедет батчем с прогона 17.11 → `[вывод]` v1.7.2 |
| [#4946](https://github.com/cozystack/cozystack/pull/4946) | yankawai | keycloak, foundationdb: CRD операторов обновляются на апгрейде | **13.11 в 15:24** lexfrei, merge-коммит, после слияния `main` в ветку | в v1.7.1 **не вошёл** (rc.2 снят 12.11); в `release-1.7` поедет батчем с прогона 17.11 → `[вывод]` v1.7.2 |
| [#4948](https://github.com/cozystack/cozystack/pull/4948) | yankawai | monitoring: `InfoInhibitor` уходит в blackhole | **13.11 в 12:56** lexfrei, merge-коммит, после слияния `main` в ветку | в v1.7.1 **не вошёл** (rc.2 снят 12.11); в `release-1.7` поедет батчем с прогона 17.11 → `[вывод]` v1.7.2 |
| [#4960](https://github.com/cozystack/cozystack/pull/4960) | yankawai | vm-disk: отказ диску меньше golden-образа | **13.11 в 12:41** lexfrei, merge-коммит, после слияния `main` в ветку | в v1.7.1 **не вошёл** (rc.2 снят 12.11); в `release-1.7` поедет батчем с прогона 17.11 → `[вывод]` v1.7.2; опирается на #4796, которого нет в линиях — `[вывод]` батч запаркует |
| [#4963](https://github.com/cozystack/cozystack/pull/4963) | yankawai | piraeus-operator: алерты по error-report'ам LINSTOR | **13.11 в 15:18** lexfrei, merge-коммит, после слияния `main` в ветку | в v1.7.1 **не вошёл** (rc.2 снят 12.11); в `release-1.7` поедет батчем с прогона 17.11 → `[вывод]` v1.7.2 |
| [#4965](https://github.com/cozystack/cozystack/pull/4965) | yankawai | tcp-balancer: порты kubernetes и talos только при `kubernetesAndTalos` | **13.11 в 15:24** lexfrei, merge-коммит, после слияния `main` в ветку | в v1.7.1 **не вошёл** (rc.2 снят 12.11); в `release-1.7` поедет батчем с прогона 17.11 → `[вывод]` v1.7.2 |
| [#4986](https://github.com/cozystack/cozystack/pull/4986) | yankawai | etcd: жёсткий спред членов по нодам | **13.11 в 12:39** lexfrei, merge-коммит, после слияния `main` в ветку | в v1.7.1 **не вошёл** (rc.2 снят 12.11); в `release-1.7` поедет батчем с прогона 17.11 → `[вывод]` v1.7.2 |
| [#4993](https://github.com/cozystack/cozystack/pull/4993) | yankawai | monitoring-agents: алерт `PodReplicasColocated`, `feat` | **13.11 в 14:38** lexfrei, merge-коммит, после слияния `main` в ветку | в v1.7.1 **не вошёл** (rc.2 снят 12.11); в `release-1.7` поедет батчем с прогона 17.11 → `[вывод]` v1.7.2; `feat` — батч не возьмёт без `backport/include` |
| [#5009](https://github.com/cozystack/cozystack/pull/5009) | yankawai | cozy-lib: память меньше 1Mi отклоняется, ломающий | **13.11 в 15:25** lexfrei, merge-коммит, после слияния `main` в ветку | в v1.7.1 **не вошёл** (rc.2 снят 12.11); в `release-1.7` поедет батчем с прогона 17.11 → `[вывод]` v1.7.2; ломающий — батч не возьмёт без `backport/include` |
| [#5011](https://github.com/cozystack/cozystack/pull/5011) | yankawai | cozystack-basics: прямое удаление тенантного неймспейса отклоняется | **13.11 в 15:25** lexfrei, merge-коммит, после слияния `main` в ветку | в v1.7.1 **не вошёл** (rc.2 снят 12.11); в `release-1.7` поедет батчем с прогона 17.11 → `[вывод]` v1.7.2 |
| [#5094](https://github.com/cozystack/cozystack/pull/5094) | yankawai | monitoring: VMServiceScrape vmalert селектит Service оператора | **13.11 в 12:16** lexfrei, merge-коммит, после слияния `main` в ветку | в v1.7.1 **не вошёл** (rc.2 снят 12.11); в `release-1.7` поедет батчем с прогона 17.11 → `[вывод]` v1.7.2 |
| [#5096](https://github.com/cozystack/cozystack/pull/5096) | yankawai | workloadmonitor: Bucket не ready, пока BucketClaim не провизионен | **13.11 в 15:11** lexfrei, merge-коммит, после слияния `main` в ветку | в v1.7.1 **не вошёл** (rc.2 снят 12.11); в `release-1.7` поедет батчем с прогона 17.11 → `[вывод]` v1.7.2; на `release-1.7` конфликтует — батч отложит в Deferred |
| [#5098](https://github.com/cozystack/cozystack/pull/5098) | yankawai | vertical-pod-autoscaler: два admission controller со спредом | **13.11 в 13:32** lexfrei, merge-коммит, после слияния `main` в ветку | в v1.7.1 **не вошёл** (rc.2 снят 12.11); в `release-1.7` поедет батчем с прогона 17.11 → `[вывод]` v1.7.2 |
| [#5100](https://github.com/cozystack/cozystack/pull/5100) | yankawai | etcd: defrag-CronJob по размеру клиента | **13.11 в 12:14** lexfrei, merge-коммит, после слияния `main` в ветку | в v1.7.1 **не вошёл** (rc.2 снят 12.11); в `release-1.7` поедет батчем с прогона 17.11 → `[вывод]` v1.7.2 |
| [#5102](https://github.com/cozystack/cozystack/pull/5102) | yankawai | monitoring-agents: тихий Watchdog — тревога в Alerta | **13.11 в 12:56** lexfrei, merge-коммит, после слияния `main` в ветку | в v1.7.1 **не вошёл** (rc.2 снят 12.11); в `release-1.7` поедет батчем с прогона 17.11 → `[вывод]` v1.7.2 |
| [#5114](https://github.com/cozystack/cozystack/pull/5114) | yankawai | kubevirt: права тенантного админа для `virtctl image-upload` | **13.11 в 15:12** lexfrei, merge-коммит, после слияния `main` в ветку | в v1.7.1 **не вошёл** (rc.2 снят 12.11); в `release-1.7` поедет батчем с прогона 17.11 → `[вывод]` v1.7.2 |
| [#5119](https://github.com/cozystack/cozystack/pull/5119) | yankawai | postgres: пятипольный cron в `backup.schedule` | **13.11 в 14:38** lexfrei, merge-коммит, после слияния `main` в ветку | в v1.7.1 **не вошёл** (rc.2 снят 12.11); в `release-1.7` поедет батчем с прогона 17.11 → `[вывод]` v1.7.2; на `release-1.7` конфликтует — батч отложит в Deferred |
| [#5249](https://github.com/cozystack/cozystack/pull/5249) | yankawai | kubernetes: kubevirt-csi отсоединяет hotplug-том при пропавшем infra PVC | **13.11 в 12:41** lexfrei, merge-коммит, после слияния `main` в ветку | в v1.7.1 **не вошёл** (rc.2 снят 12.11); в `release-1.7` поедет батчем с прогона 17.11 → `[вывод]` v1.7.2 |
| [#5106](https://github.com/cozystack/cozystack/pull/5106) | myasnikovdaniil | release: бэкпорты стали автоматическими — батч `cmd/backport-batch` по писаной политике `docs/release/backports.md`, лейбловый бот и `cmd/backport-audit` удалены, пре-релизный гейт — `backport-batch check`. Нас касается напрямую: наши ручные бэкпорты в 1.6 батч не заменит (правило 1), путь в 1.7 — батч или `backport/include` | 09.11 в 04:42:39, merge-коммит `7097cc5c` | пачки #5246 и #5290 смержены в `release-1.7` 11–12.11 и вышли в v1.7.1-rc.1 и rc.2; #5247 в 1.6 — draft, дополнена прогоном 12.11 |
| [#4985](https://github.com/cozystack/cozystack/pull/4985) | yankawai | etcd: алерт `etcdHighNumberOfFailedGRPCRequests` у тенантного etcd больше не считает ошибкой отменённые Watch-стримы — считались любые коды, кроме `OK`, а `Canceled` на Watch — штатное завершение подписки, поэтому критичный алерт горел на здоровом кластере | 23.10 в 17:40:34, merge-коммит `432d02e2` | в линию 1.6 — наш открытый #4987; **в `release-1.7` с 11.11 — пачка #5246, вышел в v1.7.1-rc.1** |
| [#5005](https://github.com/cozystack/cozystack/pull/5005) | yankawai | tcp-balancer: у `haproxy.cfg` появилась секция `global`, посчитанная от лимитов самой реплики через `cozy-lib.resources.defaultingSanitize` — HAProxy больше не берёт maxconn из nofile ноды и число потоков по её CPU, из-за чего на дефолтном `t1.nano` оба пода были OOMKilled по семь раз за четверть часа; для `t1.nano` это `fd-hard-limit 4096` и `thread-hard-limit 1` | 23.10 в 18:53:40, merge-коммит `55aad6f9` | в линию 1.6 — наш открытый #5006; **в `release-1.7` с 11.11 — пачка #5246, вышел в v1.7.1-rc.1** |
| [#5025](https://github.com/cozystack/cozystack/pull/5025) | yankawai | harbor: helm-unittest сьюта `tests/nginx_resources_test.yaml` пинит пресеты nginx-прокси и портала, которые им задал смерженный 06.09 чужой #4095 — шаблонов не трогает вовсе, закрепляет уже принятое поведение | 23.10 в 17:40:29, merge-коммит `61484797` | в линию 1.6 — наш открытый #5026 (вместе с черри-пиком самого #4095); в 1.7 — только по `backport/include`, батч `test` не берёт: остался в skipped и смерженной пачки #5246 |
| [#4889](https://github.com/cozystack/cozystack/pull/4889) | yankawai | monitoring: vmalert шлёт каждый алерт всем трём репликам Alertmanager, а не одной через общий headless-Service — как и требует README Alertmanager. Кластер дедуплицирует уведомления, только когда каждый алерт получает каждая реплика; иначе алерт на покинутой реплике истекает и уходит как resolved, а на новой заводится как новый. На трёх кластерах v1.6.4 все активные алерты сидели на одной реплике из трёх, на кластере с почтой — около 190 писем за сутки по трём именам алертов. Решает issue #4345, но в теле PR issue не упомянут и он остался открытым | 14.10 в 13:29 | ручной бэкпорт #4890 смержен 14.10 в 13:30, в v1.6.5; в линию 1.7 уехал #4943, смерженный 21.10 в 07:02:51, и 27.10 он вошёл в v1.7.0-rc.3 |
| [#4873](https://github.com/cozystack/cozystack/pull/4873) | yankawai | etcd: часовой defrag CronJob больше не компактит поверх периодической авто-компакции самого кластера (`--compaction=false`) и измеряет свой порог 80% от `quotaBackendBytes` чарта, а не от дефолтных 2 GiB у `etcd-defrag`. До этого каждый Job в логе падал с `mvcc: required revision has been compacted`, а на нагруженном кластере лишняя компакция срезала историю, по которой watcher'ы возобновляют работу | 12.10 | в v1.7.0-rc.1; бэкпорт #4874 смержен 14.10, в v1.6.5 |
| [#4682](https://github.com/cozystack/cozystack/pull/4682) | yankawai | dashboard: у free-form объектов в Form-режиме появился JSON-редактор с сохранением типов и блокировкой submit при невалидном вводе. Это не только NATS `config.merge`: то же было с `addons.*.valuesOverride` у Kubernetes, Kafka `topics[].config`, `talos.registryMirrors`, etcd `affinity`. Закрыл issue #4676 | 08.10 в 23:21 | ботовский бэкпорт #4905 по лейблу `kind/backport-previous` смержен 14.10, в v1.6.5 |
| [#4766](https://github.com/cozystack/cozystack/pull/4766) | yankawai | openbao: opt-in `seal.type: static` — тенантный OpenBAO больше не ждёт ручного `bao operator unseal` после каждого рестарта, HA-поды не зависают на старте (лейбл `allow-to-apiserver`). Ключ в Secret, который создаёт администратор кластера; чарт его не генерирует, молчаливая смена seal заблокирована. Продолжение застрявшего #4168, первый коммит — работа txmazing. Решает issue #2483 и issue #2793, но оба остались открытыми — в теле PR нет закрывающего слова | 08.10 в 23:17 | ботовский бэкпорт #4906 по лейблу `kind/backport-previous` смержен 14.10, в v1.6.5 |
| [#4724](https://github.com/cozystack/cozystack/pull/4724) | yankawai | postgres: `shared_buffers` ниже лимита 512Mi ставится в четверть лимита, но не больше memory request — PostgreSQL больше не стартует со встроенными 128MB, которые на `t1.nano` занимали весь лимит памяти. Явный `shared_buffers` побеждает | 08.10 в 21:10 | бэкпорт #4725 смержен 14.10, в v1.6.5 |
| [#4779](https://github.com/cozystack/cozystack/pull/4779) | yankawai | mongodb: установка и апгрейд с `bootstrap.enabled: true` больше не падают в helm — бакет и префикс для `PerconaServerMongoDBRestore` берутся из `backup.destinationPath` и квотируются. Багу было столько же лет, сколько чарту | 08.10 в 20:54 | бэкпорт #4780 смержен 14.10, в v1.6.5 |
| [#4778](https://github.com/cozystack/cozystack/pull/4778) | yankawai | cilium: на вариантах с kube-ovn cilium берёт свой router IP и Ingress IP не из pod CIDR, откуда kube-ovn выдаёт адреса подам, а из выделенного блока — `ipam.mode: cluster-pool` с /29 на ноду из последнего /21 от `networking.podCIDR`, и тот же диапазон уходит kube-ovn как `--default-exclude-ips`. Блок внутри pod CIDR намеренно: с пулом `100.65.0.0/16` nodeport через Gateway API отвечал 200 только на ноде бэкенда (3/9), с блоком из pod CIDR — 9/9. Живые кластеры переводит post-upgrade хук с проверками ёмкости и вложенности до любых изменений. Закрыл issue #3740. **Ломающее**: `networking.podCIDR` на этих вариантах теперь не уже /20, потолок — 256 нод. 09.10 в 22:34 yrahman82 подтвердил ту же коллизию на свежей v1.6.4 (три ноды, kubeovn-cilium): под получил `CiliumInternalIP` ноды и потерял доступ ко всем портам её хоста, включая apiserver и node-exporter. 17.10 в 14:36 yankawai ответил ему в треде: фикс уехал в `release-1.6` как #4918 и вышел в v1.6.5, в `release-1.7` он тоже есть | 08.10 в 18:41 | ручной бэкпорт #4918 от myasnikovdaniil смержен 14.10, в v1.6.5 — `kind/breaking-change` переносу не помешал |
| [#4789](https://github.com/cozystack/cozystack/pull/4789) | yankawai | dashboard: 403 или 404 на resource map сбрасывает имена ConfigMap независимо от того, было ли удачное чтение, — вкладка ConfigMaps больше не остаётся с прежними именами и ошибкой доступа на каждом из них. Хвост #4612 | 08.10 в 13:50 | отдельный бэкпорт не нужен — коммит приехал внутри #4684; в v1.6.5 |
| [#4812](https://github.com/cozystack/cozystack/pull/4812) | yankawai | monitoring: guard severity алертов больше не пропускает источники правил — поиск ловит закавыченный и закомментированный `kind`, а документ без групп и группа, пустая после вырезания template-действий, валят прогон с именем файла и группы. Хвост #3799, на текущем дереве поведение не меняется | 08.10 в 11:14 | отдельный бэкпорт не нужен — содержимое внутри #4740; в v1.6.5 |
| [#3956](https://github.com/cozystack/cozystack/pull/3956) | myasnikovdaniil | api: пустое обязательное поле в спеке чинится вместо падения всего релиза; nulls без применимых дефолтов чарта сохраняются. Шов на write-пути для create-time дефолтинга из issue #3950. Запрос IvanHunters от 01.09 висел месяц, снят им самим 30.09 | 07.10 | v1.6.5 — бэкпорт #4919, смержен 14.10 |
| [#4675](https://github.com/cozystack/cozystack/pull/4675) | yankawai | nats: JetStream включается для сгенерированного аккаунта `A`, когда он включён на сервере и нет явной настройки — `$JS.API.INFO` больше не отвечает ошибкой 10039 | 05.10 | бэкпорт #4677 смержен 14.10, в v1.6.5 |
| [#4738](https://github.com/cozystack/cozystack/pull/4738) | yankawai | platform: миграция 50 оставляет легаси etcd-оператор в нуле после adoption — тенантные etcd больше не теряют DNS пиров при апгрейде на 1.6.x. Найдено на проде при апгрейде 1.4.3 → 1.6.4 | 05.10 | бэкпорт #4739 смержен 14.10, в v1.6.5 |
| [#3800](https://github.com/cozystack/cozystack/pull/3800) | yankawai | monitoring: почтовый канал алертинга через Alertmanager рядом с Alerta — реализовано наше предложение (`alertnames`, настраиваемые `severities`); пароль SMTP только из константного Secret, адреса и `smarthost` валидируются строже `net/mail`. Пять кругов ревью 18–30.09 | 30.09 | ручной #4614 смержен 14.10, в v1.6.5-rc.2; ботовский #4609 закрыт 10.10 как дубль |
| [#4612](https://github.com/cozystack/cozystack/pull/4612) | yankawai | dashboard + foundationdb: в консоли у приложения появилась вкладка ConfigMaps — строка подключения FoundationDB теперь видна тенанту в дашборде. #4148 дал RBAC на ConfigMap, но у консоли не было представления для него; вкладка generic, но resource map пока только у foundationdb. Закрыл issue #4611 в день открытия | 30.09 | бэкпорт #4684 смержен 14.10, в v1.6.5 |
| [#4610](https://github.com/cozystack/cozystack/pull/4610) | yankawai | cozystack-api: сборщик мусора может финализировать Application — удаление с `--cascade=foreground` или orphan-пропагацией больше не зависает навсегда. Application теперь отдаёт `foregroundDeletion`/`orphan` в `metadata.finalizers` и проносит их изменения до HelmRelease; у пары общий UID, и GC финализировал узел через Application-эндпоинт, который finalizers не показывал. Найдено на v1.6.4 с Bucket и OpenBAO | 30.09 | бэкпорт #4683 смержен 14.10, в v1.6.5 |
| [#3799](https://github.com/cozystack/cozystack/pull/3799) | yankawai | linstor + keda: severity `warning`/`informational` вместо `warn`/`info` — восемь алертов больше не отбрасываются Alerta с ошибкой 500; severity закреплены тестом-контрактом по helm-шаблонам | 30.09 | бэкпорт #4740 смержен 14.10, в v1.6.5 |
| [#4366](https://github.com/cozystack/cozystack/pull/4366) | yankawai | tcp-balancer: бэкенды переименованы (rename взят из #2321, автор указан), образ запинен на `haproxy:3.4.4` LTS вместо `latest`, whitelist-guard, helm-unittest сьюты. Свежая установка снова стартует. Закрыл issue #1966, висевший с 03.02 | 25.09 | бэкпорт #4516 смержен 16.10 в 06:46, в v1.6.5 |
| [terraform-provider-cozystack#34](https://github.com/cozystack/terraform-provider-cozystack/pull/34) | yankawai | terraform-провайдер: дефолтный `resources_preset` у RabbitMQ — `s1.nano` (512Mi) вместо `t1.nano` (128Mi), как в #3936; на `t1.nano` брокер мог получить OOMKill до готовности | 25.09 | ветка `master` провайдера; его последний релиз v1.6.3 вышел 21.09, раньше мержа, — в релизы провайдера пока не попал |
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
| [#3938](https://github.com/cozystack/cozystack/pull/3938) | yankawai | kafka: топики релиза удаляются до снятия topic operator, переустановка больше не падает. Закрыл issue #3793 (23.09, вместе с #4280) | 15.09 | v1.6.4 — бэкпорт #4456, вместе с #4280 |
| [#2682](https://github.com/cozystack/cozystack/pull/2682) | Arsolitt | opensearch: TLS для HTTP API и Dashboards через cert-manager, `setVMMaxMapCount` выключен | 11.09 | пока только main |
| [#4185](https://github.com/cozystack/cozystack/pull/4185) | lexfrei | opensearch-operator: `DNS_BASE` следует домену платформы — securityadmin на `cozy.local` работает | 10.09 | пока только main |
| [#4152](https://github.com/cozystack/cozystack/pull/4152) | lexfrei | opensearch-operator: DaemonSet поднимает `vm.max_map_count` на всех нодах, privileged init больше не нужен | 08.09 | пока только main |
| [#3920](https://github.com/cozystack/cozystack/pull/3920) | lexfrei | CDI v1.66.1: HTTP-импорты в block-тома на 4Kn снова работают, нет шторма `resourceVersion` на prime-PVC | 07.09 | v1.6.4 — бэкпорт #4156 |
| [#4095](https://github.com/cozystack/cozystack/pull/4095) | IvanHunters | harbor: явные ресурсы для nginx-прокси, переживает LimitRange тенанта | 06.09 | в 1.6 везёт наш бэкпорт #5026, открыт 21.10 |
| [#3340](https://github.com/cozystack/cozystack/pull/3340) | lexfrei | nats: релиз с включённым TLS публикует тенанту CA-сертификат сервера — объект `<release>.tenant-ca` с единственным ключом `ca.crt`, без приватного ключа, — чтобы клиенты тенанта могли проверять сервер. Источник объявляется sentinel-объектом `TenantProjection`, который читает контроллер извлечения CA из `internal/controller/cacert`; sentinel рендерится только при фактически включённом TLS | 01.09 | **в 1.6 нет** — merge-коммит `edd767ea` не предок `release-1.6`, бэкпорт-PR нет, в notes v1.6.3–v1.6.5 не упомянут; в `release-1.7` есть. Тип `feat` — `[вывод]` по политике из draft #5106 в патч 1.6 не попадёт, придёт с 1.7. В тред 02.10 сослался issue #4702 (lexfrei): в e2e нет сьюты, которая ставит nats, поэтому путь доверенного CA тенанту не проверен |
