# Отслеживание PR в cozystack

Обновлено: **2026-10-06**. Репозиторий по умолчанию — `cozystack/cozystack`, иначе указано явно.

На GitHub у issue и PR **общая нумерация**, по номеру их не отличить. Поэтому здесь issue всегда помечены словом — `issue #4083`, а голый `#4133` означает PR.

## Главное за 05–06.10

- **05.10 lexfrei прошёл по всем нашим открытым PR за одно утро** (11:41–11:44): два одобрил и в тот же день смержил, на два других запросил правки — и yankawai закрыл обе в тот же день
- **#4738 смержен 05.10** (апрув 11:44, мерж 14:53, E2E зелёный): легаси etcd-оператор больше не возвращается после adoption в миграции 50. Backport-лейбла нет — в release-1.6 сам не поедет
- **#4675 смержен 05.10** (апрув 11:41, мерж 15:00, E2E зелёный): JetStream включается для сгенерированного аккаунта NATS. Backport-лейбла тоже нет
- **#4724 — NOT LGTM, но по делу и уже исправлено**: lexfrei нашёл, что вебхук CloudNativePG сравнивает `shared_buffers` с memory **request**, а не limit — на платформах с `memory-allocation-ratio` выше 4 новый дефолт сделал бы Cluster неприемлемым на admission, и апгрейд каждой маленькой базы упал бы. yankawai в 12:56 запушил «cap shared_buffers at the memory request», бэкпорт #4725 обновлён в 12:57
- **#4682 — код признан готовым**, блокер был один: шаблон PR требует скриншоты для UI-изменений. yankawai добавил их в 13:39. lexfrei отдельно отметил, что баг шире NATS: пустым fieldset рендерится любой free-form объект — десять `addons.*.valuesOverride` у Kubernetes, Kafka `topics[].config`, `talos.registryMirrors`, etcd `affinity`
- **#3956: E2E «красный» не из-за PR**: lexfrei 05.10 ребейзнул ветку и переодобрил, но на новой голове упал unit-тест `internal/fluxcontract` про chainsaw-сьют monitoring — файлов PR он не касается; в main этот контракт починили в тот же день в 15:10. `[вывод]` Ещё один ребейз и прогон — и PR зелёный
- **06.10 yankawai открыл #4766 — и lexfrei одобрил его в тот же день**: static-key auto-unseal для OpenBAO, продолжение застрявшего с 10.09 #4168 от txmazing с сохранением авторства. Код был признан верным ещё на #4168, правки были только текстовые. CI красный на сборке тестов пакета, который PR не трогает. Вокруг — эпик issue #2787 и платформенный draft #4177
- **Взят в трекинг issue #3740** (lexfrei, `priority/important-soon`): cilium и kube-ovn берут адреса из одного pod CIDR, и под может получить уже занятый адрес. По дефолтам установки касается живых кластеров, фикса пока нет
- **Без изменений**: бэкпорты #4614/#4609, отсутствие `kind/backport` на #3799, community#25, issue #3950, issue #3022. Нового стабильного релиза после v1.6.4 нет

## Требует действия

Всё в таблице — действия мейнтейнеров; на нашей стороне только мелочь по #4682 (см. строку PR).

| Что | Где | Кому и что делать |
|---|---|---|
| **Повторное ревью после правок и запуск CI** | #4724, #4725, #4682 | lexfrei: правки по его ревью от 05.10 запушены в тот же день — cap по memory request в #4724 и бэкпорте, скриншоты в #4682. На всех трёх головах CI в `action_required` |
| **Ребейз и прогон** | #3956, #4766 | Мейнтейнер: оба одобрены; красное в CI — тесты пакетов, которые PR не трогают: у #3956 контракт-тест `internal/fluxcontract`, уже починенный в main, у #4766 сборка тестов `internal/backupcontroller` |
| **Закрыть заменённый** | #4168 | Мейнтейнер: после мержа #4766, который несёт работу txmazing с сохранённым авторством |
| **Бэкпорты в release-1.6** | #4614, #4609, #3799, #4738, #4675 | Мейнтейнер: смержить ручной #4614 и закрыть конфликтный ботовский #4609; повесить `kind/backport` на #3799, а также на свежесмерженные #4738 и #4675 — ручных бэкпортов у них нет, дубля не будет |
| **Ревью proposal** | community#25 | Любой мейнтейнер: второй драфт с 21.08 без единого ревью, наш отчёт о прогоне миграции с 03.09 без ответа |
| **Закрыть issue** | issue #3022 | Мейнтейнер: по словам автора #4026 решён мержем #4152 и #2682 |

**В одобренные PR ничего не пушить** — новый коммит снимет апрув (`dismiss_stale_reviews_on_push: true`).

## Наши PR — открытые

| PR | Автор | Название | Что решает | Состояние |
|---|---|---|---|---|
| [#4766](https://github.com/cozystack/cozystack/pull/4766) | yankawai | feat(openbao): optional static-key auto-unseal and apiserver egress label | Тенантный OpenBAO поднимается запечатанным и после каждого рестарта ждёт ручного `bao operator unseal`, а HA-поды зависают на старте без egress к apiserver. PR добавляет opt-in `seal.type: static` (ключ в Secret, который создаёт администратор кластера; чарт ключ не генерирует), защиту от молчаливой смены seal, лейбл `allow-to-apiserver` и ограничения на значения: `keyId` уходит в `tpl` внутри helm-controller с правами cluster-admin. Продолжение застрявшего #4168: первый коммит — его работа, автор txmazing | **APPROVED** — lexfrei 06.10 15:07, после своего же NOT LGTM в 13:21, закрытого за полтора часа. CI прогнался: `pre-commit` зелёный, workflow «Pull Request» красный на сборке тестов `internal/backupcontroller` — пакета, который PR не трогает. `[вывод]` Рассинхрон в main вокруг бэкап-PR, нужен ребейз и прогон. Закроет issue #2483 и issue #2793 |
| [#4724](https://github.com/cozystack/cozystack/pull/4724) | yankawai | fix(postgres): cap the default shared_buffers at a quarter of the memory limit | Чарт не задаёт `shared_buffers`, и CNPG не задаёт — PostgreSQL стартует со встроенными 128MB: на `t1.nano` это весь лимит памяти, на дефолтном `t1.micro` — половина. Дамп, большой скан или догоняющая реплика заполняют пул — инстанс получает OOMKill. Ниже лимита 512Mi чарт ставит четверть лимита, но не больше memory request; явный `shared_buffers` побеждает | **CHANGES_REQUESTED** — lexfrei 05.10: вебхук CloudNativePG (`validateResources` в v1.30.0) отвергает Cluster, если `shared_buffers` больше memory **request**, а request — это limit, делённый на `memory-allocation-ratio`; при ratio выше 4 на маленьких пресетах четверть лимита больше request, и апгрейд каждой маленькой базы упал бы на admission. **Исправлено в тот же день** коммитом «cap shared_buffers at the memory request»; бэкпорт #4725 обновлён синхронно. Ждёт повторного ревью и запуска CI (`action_required` на обоих) |
| [#4682](https://github.com/cozystack/cozystack/pull/4682) | yankawai | fix(dashboard): allow editing free-form objects in application forms | В Form-режиме консоли у free-form полей нет редактора — они рендерятся пустым fieldset. По словам lexfrei, это не только NATS `config.merge`: то же с десятью `addons.*.valuesOverride` у Kubernetes, Kafka `topics[].config`, `talos.registryMirrors`, etcd `affinity`. PR добавляет JSON-редактор с сохранением типов и блокировкой submit при невалидном вводе. `Fixes issue #4676` | **CHANGES_REQUESTED** — lexfrei 05.10: «код готов к мержу», единственный блокер — скриншоты, обязательные по шаблону для UI-изменений; он прогнал 463 теста консоли и мутационно проверил новые. Вышел из draft 04.10; **скриншоты добавлены 05.10**. После этого в ветку влит main merge-коммитом — `[вывод]` на #3800 lexfrei блокировал именно merge-коммит в истории, может попросить ребейз; CodeRabbit оставил minor про валидацию перед переключением в YAML. Ждёт повторного ревью и запуска CI (`action_required`, включая UI Test) |

## Чужие PR — открытые

| PR | Автор | Название | Что решает | Состояние |
|---|---|---|---|---|
| [#3956](https://github.com/cozystack/cozystack/pull/3956) | myasnikovdaniil | fix(api): repair an empty required field instead of failing the release | Пустое обязательное поле в спеке роняло установку всего релиза | **APPROVED** — IvanHunters 30.09, lexfrei переодобрил 05.10 после ребейза («to pick up the bucket suite fix that made the last E2E run red»). На новой голове упал unit-тест `TestChainsawSuitesReadHistoryThroughTheSharedName` в `internal/fluxcontract`: он проверяет `hack/e2e-chainsaw/monitoring/chainsaw-test.yaml`, которого PR не касается, а в main этот контракт починен 05.10 в 15:10 — `[вывод]` нужен ещё один ребейз и прогон. Важен как шов на write-пути для create-time дефолтинга из issue #3950 |
| [#4168](https://github.com/cozystack/cozystack/pull/4168) | txmazing | feat(openbao): optional static-key auto-unseal and apiserver egress label | Первая версия static-key auto-unseal и egress-лейбла для OpenBAO | **Заменён #4766.** lexfrei 30.09: «NOT LGTM, but only for text», код признан верным. Автор не отвечал с 10.09, yankawai 06.10 перенёс работу в #4766 с сохранением авторства и sign-off и оставил в #4168 комментарий. Кандидат на закрытие после мержа #4766 |

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

Красный статус остался только у #3956 — и тот от контракт-теста, сломанного и в тот же день починенного в main 05.10. Из двух issue про backup round-trip issue #4262 закрыт 23.09 lexfrei, issue #4258 открыт.

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
| повторное ревью lexfrei и запуск CI после правок от 05.10 | #4724, #4725, #4682 |
| ещё один ребейз и прогон — красный от уже починенного в main контракт-теста | #3956 |
| ребейз и прогон — красная сборка тестов `internal/backupcontroller`, не связанная с PR | #4766 |
| ничего — заменён #4766, закрыть после его мержа | #4168 |

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

## Issues

| Issue | Автор | Название | Состояние |
|---|---|---|---|
| [#3950](https://github.com/cozystack/cozystack/issues/3950) | IvanHunters | Platform-wide defaults for tenant Talos worker settings | OPEN, **23.09 lexfrei принял в работу**: `triage/accepted` и `priority/important-longterm` вместо `triage/needs-triage`. Смерженный 24.09 #4171 закрыл каталожную сторону — платформенный ключ `kubernetesWorkerImage` позволяет направить импорт golden-образа в зеркало, — но потребляющая сторона дефолтов (`imageFactoryURL`, `installerRepository`, `registryMirrors` на `KubernetesNodes`) осталась. В треде готовый разбор: наш комментарий 07.09 о create-time дефолтинге, ответ myasnikovdaniil 08.09 (двум полям платформенный дефолт безопасен, `imageFactoryURL` — прокат флота), предупреждение lexfrei 01.09 о невозможности plain default chain в Helm. Шов на write-пути — #3956 |
| [#3022](https://github.com/cozystack/cozystack/issues/3022) | lexfrei | OpenSearch fails to start in tenant namespaces: privileged init-sysctl violates baseline PodSecurity | OPEN, но **по сути решён**: #4152 поднимает `vm.max_map_count` DaemonSet'ом, #2682 выключил `setVMMaxMapCount`. Автор #4026 предложил закрыть |
| [#4073](https://github.com/cozystack/cozystack/issues/4073) | lexfrei | opensearch-operator: dnsBase stays cluster.local | **закрыт** 17.09 — фикс в #4185 |
| [#3793](https://github.com/cozystack/cozystack/issues/3793) | IvanHunters | Deleting a tenant with a Kafka app hangs the namespace in Terminating (KafkaTopic strimzi.io/topic-operator finalizer) | **закрыт** 23.09 lexfrei — как мы и просили: fixed by #3938, плюс #4280 для кредов хука; бэкпорт в release-1.6 — #4421 |
| [#4676](https://github.com/cozystack/cozystack/issues/4676) | yankawai | bug(dashboard): NATS free-form configuration fields have no editor in Form mode | OPEN с 01.10. Фикс — #4682: код признан готовым 05.10, ждёт повторного ревью после добавленных скриншотов. По ревью lexfrei, баг шире NATS — касается всех free-form объектов в формах |
| [#4611](https://github.com/cozystack/cozystack/issues/4611) | yankawai | FoundationDB connection string is not visible to tenants in the dashboard | **закрыт 30.09** мержем #4612 — в день открытия. Хвост #4148: RBAC появился, но консоль не умела показывать ConfigMap |
| [#1966](https://github.com/cozystack/cozystack/issues/1966) | lllamnyp | tcp-balancer: HAProxy 3.3 breaks due to frontend/backend name collision | **закрыт 25.09** мержем #4366 — спустя почти восемь месяцев. Провисел с 03.02 с `priority/important-soon` и успел получить `lifecycle/stale`; первый фикс #2321 застрял на авторе, довёл задачу #4366 |
| [#2787](https://github.com/cozystack/cozystack/issues/2787) | myasnikovdaniil | Make managed OpenBAO production-mature: auto-unseal, TLS, init/bootstrap, guardrails | OPEN с 02.06, эпик: `security`, `triage/accepted`, `priority/important-longterm`. Тенантные инстансы стартуют запечатанными и неинициализированными, TLS выключен везде — включая репликацию Raft, системный OpenBAO — заглушка. Две линии работы: простой static-key auto-unseal — #4766 (одобрен, продолжение #4168), и платформенный путь — draft #4177 от myasnikovdaniil: центральный OpenBAO с transit-ключом на тенанта, фазы 1–2 эпика, тенантной половины пока нет, без ревью с 09.09. Единственный комментарий в эпике — «This need design doc first» (09.06) |
| [#3740](https://github.com/cozystack/cozystack/issues/3740) | lexfrei | networking: cilium takes its per-node infrastructure IPs from the pod CIDR kube-ovn allocates pods from | OPEN с 10.08, 23.09 — `triage/accepted`, `priority/important-soon`. Kube-OVN выдаёт адреса подам из всего `networking.podCIDR`, а cilium берёт два адреса на ноду (router IP и Ingress IP) из `Node.spec.podCIDR` — по дефолтам это один диапазон. Рано или поздно под получает адрес, который уже держит cilium: тихо ломается трафик или под висит `ContainerCreating` с `putEndpointIdInvalid`. Оба документированных пути установки дают пересечение: в talm `podSubnets` совпадает с дефолтом платформы `10.244.0.0/16`, в k3s-роли оба — `10.42.0.0/16`. 26.09 воспроизведено на свежей установке. В CI обойдено #3750 только для песочницы, фикса дефолтов нет. Касается живых кластеров |
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

| PR | Автор | Что решил | Смержен | Релиз |
|---|---|---|---|---|
| [#4675](https://github.com/cozystack/cozystack/pull/4675) | yankawai | nats: JetStream включается для сгенерированного аккаунта `A`, когда он включён на сервере и нет явной настройки — `$JS.API.INFO` больше не отвечает ошибкой 10039 | 05.10 | пока только main — backport-лейбла нет |
| [#4738](https://github.com/cozystack/cozystack/pull/4738) | yankawai | platform: миграция 50 оставляет легаси etcd-оператор в нуле после adoption — тенантные etcd больше не теряют DNS пиров при апгрейде на 1.6.x. Найдено на проде при апгрейде 1.4.3 → 1.6.4 | 05.10 | пока только main — backport-лейбла нет |
| [#3800](https://github.com/cozystack/cozystack/pull/3800) | yankawai | monitoring: почтовый канал алертинга через Alertmanager рядом с Alerta — реализовано наше предложение (`alertnames`, настраиваемые `severities`); пароль SMTP только из константного Secret, адреса и `smarthost` валидируются строже `net/mail`. Пять кругов ревью 18–30.09 | 30.09 | бэкпорт в release-1.6 в пути: ботовский #4609 конфликтнул (draft с маркерами), чистый ручной #4614 ждёт ревью |
| [#4612](https://github.com/cozystack/cozystack/pull/4612) | yankawai | dashboard + foundationdb: в консоли у приложения появилась вкладка ConfigMaps — строка подключения FoundationDB теперь видна тенанту в дашборде. #4148 дал RBAC на ConfigMap, но у консоли не было представления для него; вкладка generic, но resource map пока только у foundationdb. Закрыл issue #4611 в день открытия | 30.09 | пока только main |
| [#4610](https://github.com/cozystack/cozystack/pull/4610) | yankawai | cozystack-api: сборщик мусора может финализировать Application — удаление с `--cascade=foreground` или orphan-пропагацией больше не зависает навсегда. Application теперь отдаёт `foregroundDeletion`/`orphan` в `metadata.finalizers` и проносит их изменения до HelmRelease; у пары общий UID, и GC финализировал узел через Application-эндпоинт, который finalizers не показывал. Найдено на v1.6.4 с Bucket и OpenBAO | 30.09 | пока только main |
| [#3799](https://github.com/cozystack/cozystack/pull/3799) | yankawai | linstor + keda: severity `warning`/`informational` вместо `warn`/`info` — восемь алертов больше не отбрасываются Alerta с ошибкой 500; severity закреплены тестом-контрактом по helm-шаблонам | 30.09 | пока только main — backport-лейбла нет, автобэкпорт не создавался |
| [#4366](https://github.com/cozystack/cozystack/pull/4366) | yankawai | tcp-balancer: бэкенды переименованы (rename взят из #2321, автор указан), образ запинен на `haproxy:3.4.4` LTS вместо `latest`, whitelist-guard, helm-unittest сьюты. Свежая установка снова стартует. Закрыл issue #1966, висевший с 03.02 | 25.09 | пока только main |
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
