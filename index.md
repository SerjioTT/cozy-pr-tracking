# Отслеживание PR в cozystack

Обновлено: **2026-10-07**. Репозиторий по умолчанию — `cozystack/cozystack`, иначе указано явно.

На GitHub у issue и PR **общая нумерация**, по номеру их не отличить. Поэтому здесь issue всегда помечены словом — `issue #4083`, а голый `#4133` означает PR.

## Главное за 06–07.10

- **#3956 смержен 07.10** lexfrei (11:35, E2E зелёный) — после ещё одного ребейза в ночь на 07.10. Шов на write-пути для create-time дефолтинга из issue #3950 теперь в main
- **#4724 и #4682 одобрены** lexfrei 06.10 в 13:14 — правки от 05.10 приняты: cap `shared_buffers` по memory request и скриншоты. Бэкпорт #4725 синхронизирован с #4724 вечером 06.10
- **#4766 переодобрен** lexfrei 06.10 в 19:06 после доработки документации
- **Все три одобренных PR красные только из-за поломок main, уже починенных там**: у #4724 и #4682 — контракт-тест `internal/fluxcontract` (в main починен 05.10 в 15:10), у #4766 — сборка тестов `internal/backupcontroller`, сломанная в main 06.10 между мержами #4669 и #4671 и починенная #4768 07.10 в 04:36. Статус `E2E Tests` лишь пересылает падение workflow «Pull Request». Нужен ребейз и свежий прогон. `[вывод]` Пуш в одобренный PR снимет апрув — удобнее, если ребейз сделает мейнтейнер, как lexfrei сделал с #3956
- **yankawai 06.10 описал в issue #3740 четыре случая на наших кластерах** (v1.6.4, три кластера, дефолты kubeovn-cilium): свежий postgres не поднялся, хук очистки mariadb упал и оставил PVC, поды virt-operator и keycloak-operator живут на адресах cilium чужих нод. Обход — исключить адреса cilium в `excludeIps` у `ovn-default`; вариант с `cluster-pool` из `100.65.0.0/16` ломает Gateway API между нодами. yankawai готовит PR с пулом из конца pod CIDR, исключённым из kube-ovn
- **Без изменений**: бэкпорты #4614/#4609, `kind/backport` на #3799, #4738, #4675 так и не повешен, community#25, issue #3950, issue #3022. Нового стабильного релиза после v1.6.4 нет

## Требует действия

Всё в таблице — действия мейнтейнеров.

| Что | Где | Кому и что делать |
|---|---|---|
| **Ребейз, прогон и мерж** | #4724, #4682, #4766 | Мейнтейнер: все три одобрены lexfrei; красный CI — от поломок main, уже починенных там (`internal/fluxcontract` 05.10, `internal/backupcontroller` в #4768 07.10). `[вывод]` Пуш автора снимет апрув, поэтому удобнее ребейз мейнтейнером с переодобрением |
| **Ревью и запуск CI** | #4725 | Мейнтейнер: бэкпорт #4724 в release-1.6, синхронизирован с одобренным оригиналом 06.10, ревью и CI нет |
| **Бэкпорты в release-1.6** | #4614, #4609, #3799, #4738, #4675 | Мейнтейнер: смержить ручной #4614 и закрыть конфликтный ботовский #4609; повесить `kind/backport` на #3799, #4738 и #4675 — ручных бэкпортов у них нет, дубля не будет |
| **Ревью proposal** | community#25 | Любой мейнтейнер: второй драфт с 21.08 без единого ревью, наш отчёт о прогоне миграции с 03.09 без ответа |
| **Закрыть issue** | issue #3022 | Мейнтейнер: по словам автора #4026 решён мержем #4152 и #2682 |

**В одобренные PR ничего не пушить** — новый коммит снимет апрув (`dismiss_stale_reviews_on_push: true`).

## Наши PR — открытые

| PR | Автор | Название | Что решает | Состояние |
|---|---|---|---|---|
| [#4766](https://github.com/cozystack/cozystack/pull/4766) | yankawai | feat(openbao): optional static-key auto-unseal and apiserver egress label | Тенантный OpenBAO поднимается запечатанным и после каждого рестарта ждёт ручного `bao operator unseal`, а HA-поды зависают на старте без egress к apiserver. PR добавляет opt-in `seal.type: static` (ключ в Secret, который создаёт администратор кластера; чарт ключ не генерирует), защиту от молчаливой смены seal, лейбл `allow-to-apiserver` и ограничения на значения: `keyId` уходит в `tpl` внутри helm-controller с правами cluster-admin. Продолжение застрявшего #4168: первый коммит — его работа, автор txmazing | **APPROVED** — lexfrei 06.10 в 15:07 и повторно в 19:06 после доработки документации. Красный CI — сборка тестов `internal/backupcontroller`, которого PR не трогает: main был сломан 06.10 между мержами #4669 и #4671 и починен #4768 07.10 в 04:36. Нужен ребейз и прогон. Закроет issue #2483 и issue #2793 |
| [#4724](https://github.com/cozystack/cozystack/pull/4724) | yankawai | fix(postgres): cap the default shared_buffers at a quarter of the memory limit | Чарт не задаёт `shared_buffers`, и CNPG не задаёт — PostgreSQL стартует со встроенными 128MB: на `t1.nano` это весь лимит памяти, на дефолтном `t1.micro` — половина. Дамп, большой скан или догоняющая реплика заполняют пул — инстанс получает OOMKill. Ниже лимита 512Mi чарт ставит четверть лимита, но не больше memory request; явный `shared_buffers` побеждает | **APPROVED** — lexfrei 06.10 в 13:14, после того как правка от 05.10 ограничила `shared_buffers` memory request: иначе вебхук CloudNativePG отвергал бы Cluster при `memory-allocation-ratio` выше 4. Красный CI — контракт-тест `internal/fluxcontract`, сломанный в main и починенный там 05.10 в 15:10: прогон шёл на старой базе. Нужен ребейз и прогон. Бэкпорт #4725 синхронизирован 06.10 вечером, ревью и CI у него нет |
| [#4682](https://github.com/cozystack/cozystack/pull/4682) | yankawai | fix(dashboard): allow editing free-form objects in application forms | В Form-режиме консоли у free-form полей нет редактора — они рендерятся пустым fieldset. По словам lexfrei, это не только NATS `config.merge`: то же с десятью `addons.*.valuesOverride` у Kubernetes, Kafka `topics[].config`, `talos.registryMirrors`, etcd `affinity`. PR добавляет JSON-редактор с сохранением типов и блокировкой submit при невалидном вводе. `Fixes issue #4676` | **APPROVED** — lexfrei 06.10 в 13:14, после добавленных скриншотов; влитый main merge-коммит его не остановил. Красный CI — тот же контракт-тест `internal/fluxcontract`, уже починенный в main. Нужен ребейз и прогон. CodeRabbit оставил minor про валидацию перед переключением в YAML |

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

Красные статусы на одобренных PR сейчас — только от поломок main, уже починенных там: контракт-тест `internal/fluxcontract` (05.10) и сборка тестов `internal/backupcontroller` (#4768, 07.10). Из двух issue про backup round-trip issue #4262 закрыт 23.09 lexfrei, issue #4258 открыт.

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
| ребейз и прогон — красное только от поломок main, уже починенных; все три одобрены | #4724, #4682, #4766 |
| ревью и запуск CI | #4725 |

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
| [#3950](https://github.com/cozystack/cozystack/issues/3950) | IvanHunters | Platform-wide defaults for tenant Talos worker settings | OPEN, **23.09 lexfrei принял в работу**: `triage/accepted` и `priority/important-longterm` вместо `triage/needs-triage`. Смерженный 24.09 #4171 закрыл каталожную сторону — платформенный ключ `kubernetesWorkerImage` позволяет направить импорт golden-образа в зеркало, — но потребляющая сторона дефолтов (`imageFactoryURL`, `installerRepository`, `registryMirrors` на `KubernetesNodes`) осталась. В треде готовый разбор: наш комментарий 07.09 о create-time дефолтинге, ответ myasnikovdaniil 08.09 (двум полям платформенный дефолт безопасен, `imageFactoryURL` — прокат флота), предупреждение lexfrei 01.09 о невозможности plain default chain в Helm. Шов на write-пути — #3956, **смержен 07.10** |
| [#3022](https://github.com/cozystack/cozystack/issues/3022) | lexfrei | OpenSearch fails to start in tenant namespaces: privileged init-sysctl violates baseline PodSecurity | OPEN, но **по сути решён**: #4152 поднимает `vm.max_map_count` DaemonSet'ом, #2682 выключил `setVMMaxMapCount`. Автор #4026 предложил закрыть |
| [#4073](https://github.com/cozystack/cozystack/issues/4073) | lexfrei | opensearch-operator: dnsBase stays cluster.local | **закрыт** 17.09 — фикс в #4185 |
| [#3793](https://github.com/cozystack/cozystack/issues/3793) | IvanHunters | Deleting a tenant with a Kafka app hangs the namespace in Terminating (KafkaTopic strimzi.io/topic-operator finalizer) | **закрыт** 23.09 lexfrei — как мы и просили: fixed by #3938, плюс #4280 для кредов хука; бэкпорт в release-1.6 — #4421 |
| [#4676](https://github.com/cozystack/cozystack/issues/4676) | yankawai | bug(dashboard): NATS free-form configuration fields have no editor in Form mode | OPEN с 01.10. Фикс — #4682: код признан готовым 05.10, ждёт повторного ревью после добавленных скриншотов. По ревью lexfrei, баг шире NATS — касается всех free-form объектов в формах |
| [#4611](https://github.com/cozystack/cozystack/issues/4611) | yankawai | FoundationDB connection string is not visible to tenants in the dashboard | **закрыт 30.09** мержем #4612 — в день открытия. Хвост #4148: RBAC появился, но консоль не умела показывать ConfigMap |
| [#1966](https://github.com/cozystack/cozystack/issues/1966) | lllamnyp | tcp-balancer: HAProxy 3.3 breaks due to frontend/backend name collision | **закрыт 25.09** мержем #4366 — спустя почти восемь месяцев. Провисел с 03.02 с `priority/important-soon` и успел получить `lifecycle/stale`; первый фикс #2321 застрял на авторе, довёл задачу #4366 |
| [#3740](https://github.com/cozystack/cozystack/issues/3740) | lexfrei | networking: cilium takes its per-node infrastructure IPs from the pod CIDR kube-ovn allocates pods from | OPEN с 10.08, `triage/accepted`, `priority/important-soon`. Kube-OVN выдаёт адреса подам из всего `networking.podCIDR`, а cilium берёт по два адреса на ноду (router IP и Ingress IP) из `Node.spec.podCIDR` — по дефолтам это один диапазон, и под рано или поздно получает уже занятый адрес. **06.10 yankawai задокументировал четыре случая на наших кластерах** (v1.6.4, дефолты kubeovn-cilium, 186–212 занятых адресов на кластер): свежий postgres не поднялся — пробы kubelet попадали в `cilium_host`; хук очистки mariadb упал на `putEndpointIdInvalid` и оставил PVC; virt-operator и keycloak-operator держат адреса cilium чужих нод. Обход — адреса cilium в `excludeIps` у `ovn-default`, но его надо повторять на каждую новую ноду. Вариант `cluster-pool` из `100.65.0.0/16` ломает Gateway API между нодами: источник вне `ovn40subnets` не маскарадится. Пул из конца pod CIDR, исключённый из kube-ovn, работает — yankawai готовит PR |
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
| [#3956](https://github.com/cozystack/cozystack/pull/3956) | myasnikovdaniil | api: пустое обязательное поле в спеке чинится вместо падения всего релиза; nulls без применимых дефолтов чарта сохраняются. Шов на write-пути для create-time дефолтинга из issue #3950. Запрос IvanHunters от 01.09 висел месяц, снят им самим 30.09 | 07.10 | пока только main |
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
