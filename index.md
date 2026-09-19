# Отслеживание PR в cozystack

Обновлено: **2026-09-19**. Репозиторий по умолчанию — `cozystack/cozystack`, иначе указано явно.

На GitHub у issue и PR **общая нумерация**, по номеру их не отличить. Поэтому здесь issue всегда помечены словом — `issue #4083`, а голый `#4133` означает PR.

## Главное за вечер 18.09

- **lexfrei смержил ещё семь наших PR**: #4253 и #4136 (13:29), #4014 (15:36), #3935 (17:29), #4100 (17:50), #3937 (18:16), #4291 (18:23). Вместе с утренними — **13 наших PR в main за один день**. Issue #4086 закрыт мержем #4136 — вся четвёрка issue #4083–#4086 закрыта
- Перед мержем lexfrei **переодобрил** #4014 (после тестов), #3937 (после ребейза) и #4291 (после объявления `EXIT`-трапов). #4291 смержен **с красным E2E** — PR с новым образом из форка до конца и не проверить
- **Перезапуски E2E позеленели и на opensearch**: #4100 и #3935 смержены с зелёным E2E. `[вывод]` Вчерашнее устойчивое падение opensearch тоже было нестабильностью стенда, а не регрессией
- **Вся четвёрка живой миграции VM в main**: #4254, website#697, #4253, #4291
- **Из наших открытых осталось три PR**: #3936, #3800, #3799. Вечером yankawai запушил в #3800 и #3936 правки по оставшимся замечаниям IvanHunters — CI на новых коммитах снова в `action_required`
- **#3799 прошёл CI после ребейза**: `pre-commit` и `E2E Tests` зелёные. Единственный блокер — неснятый запрос scooby87, его замечание про трейлер исправлено ещё 17.09

## Требует действия

| Что | Где | Кому и что делать |
|---|---|---|
| **Снять запрос или повторное ревью** | #3799 | scooby87: CI зелёный, замечание про трейлер исправлено 17.09 — PR готов, кроме этого запроса |
| **Запустить CI на новых коммитах** | #3800, #3936 | Мейнтейнер: workflow стоят в `action_required` после пушей вечера 18.09 |
| **Повторное ревью после правок** | #3800, #3936 | IvanHunters: правки по его ревью от 18.09 запушены в тот же вечер |
| **Снять устаревший запрос изменений** | #3956 — IvanHunters с 01.09 | Повторный проход или dismiss. lexfrei одобрил 11.09 |
| **Закрыть issue** | issue #3022, issue #3793 | Мейнтейнер: #3022 по словам автора #4026 решён мержем #4152 и #2682. #3793 упомянут в описании смерженного #3938, но без `Fixes`. `[вывод]` Скорее всего #3938 его и закрывает — стоит проверить и закрыть |
| **Разрешить конфликт** | #4171 | myasnikovdaniil: конфликт с main и запрос изменений lexfrei от 09.09, без движения с 10.09 |

**В одобренные PR ничего не пушить** — новый коммит снимет апрув (`dismiss_stale_reviews_on_push: true`).

## Наши PR — открытые

| PR | Автор | Название | Что решает | Состояние |
|---|---|---|---|---|
| [#3936](https://github.com/cozystack/cozystack/pull/3936) | yankawai | fix(rabbitmq): right-size the default resources preset to s1.nano | На дефолтном пресете `t1.nano` брокер получал OOMKill, набирал 10 рестартов и не становился Ready. Пресет поднят до `s1.nano` | **CHANGES_REQUESTED** — IvanHunters 18.09: 2 MAJOR — release note обещает новый дефолт и тем инстансам, у которых `resourcesPreset` уже сохранён в спеке, а README документирует дефолт, отсутствующий в его же таблице пресетов — и 1 MINOR про частично заданные `resources`. **Вечером 18.09 запушены правки**: описание пресета теперь говорит, что он дозаполняет незаданное в `resources`, а раздел пресетов в README указывает на общую матрицу размеров. `[вывод]` По заголовкам коммитов закрыты MAJOR про README и MINOR; закрыт ли MAJOR про release note — по заголовкам не видно. Ждёт повторного ревью и запуска CI |
| [#3800](https://github.com/cozystack/cozystack/pull/3800) | yankawai | feat(monitoring): add optional email receiver to alertmanager | Почтовый канал алертинга через Alertmanager рядом с Alerta, пароль SMTP монтируется из Secret. Реализовано наше предложение: список `alertnames` и настраиваемые `severities` | **CHANGES_REQUESTED** — IvanHunters 18.09: 2 MAJOR — `smarthost` не валидируется; связка `spec.secrets` и свободного `smarthost` превращает Monitoring CR в примитив чтения Secret, недоступных автору CR по RBAC — плюс 3 MINOR. Днём 18.09 запушены правки по обоим MAJOR (пароль SMTP из фиксированного Secret, валидация `smarthost`, наследование корневого `group_by`), **вечером — второй пакет**: валидация адресов to/from, тесты на каждый терм guard'а частичной конфигурации, в docs закреплено, что SMTP-кред принадлежит одному тенанту. `[вывод]` По заголовкам коммитов закрыты оба MAJOR и все три MINOR. Ждёт повторного ревью IvanHunters и запуска CI |
| [#3799](https://github.com/cozystack/cozystack/pull/3799) | yankawai | fix(linstor): use severity warning instead of warn in prometheus rules | Семь алертов LINSTOR/DRBD с `severity: warn` Alerta отбрасывала с ошибкой 500, и они молча терялись | **CHANGES_REQUESTED** — висит запрос scooby87 от 17.09: единственным блокером был трейлер коммита, исправлено 17.09 (`Assisted-by: LLM`). Ребейз 18.09, **CI прошёл: `pre-commit` и `E2E Tests` зелёные**. Заодно keda-алерт переведён с `severity: info` на `informational`, severity закреплены тестом-контрактом по helm-шаблонам, включая template rules. Остался только неснятый запрос |

## Чужие PR — открытые

| PR | Автор | Название | Что решает | Состояние |
|---|---|---|---|---|
| [#4171](https://github.com/cozystack/cozystack/pull/4171) | myasnikovdaniil | feat(kubernetes): boot tenant worker disks from a shared golden Talos image (CDI clone) | Воркеры тенантных кластеров грузятся клоном общего golden-образа Talos вместо того, чтобы каждый тянуть ~4 GiB по HTTP. Каталог настраивается платформенным ключом `kubernetesWorkerImage` | **CHANGES_REQUESTED** — lexfrei 09.09. **Конфликтует с main**, E2E красный. Без движения с 10.09. Дефолтом clone не становится, выбор per-pool через `osImage.builtin` |
| [#3956](https://github.com/cozystack/cozystack/pull/3956) | myasnikovdaniil | fix(api): repair an empty required field instead of failing the release | Пустое обязательное поле в спеке роняло установку всего релиза | **CHANGES_REQUESTED** — висит запрос IvanHunters от 01.09, lexfrei одобрил 11.09. E2E красный (прогон от 10.09). Важен как шов на write-пути для create-time дефолтинга из issue #3950 |

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

Красный E2E остался только у #4171 (конфликтует с main) и #3956 (прогон от 10.09). Issue #4262 и #4258 про backup round-trip открыты — `[предположение]` флаки стенда связаны с ними.

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
| только неснятый запрос scooby87 — CI зелёный | #3799 |
| повторное ревью IvanHunters и запуск CI после правок | #3800, #3936 |
| неснятый запрос изменений и E2E | #3956 |
| конфликт, правки и E2E | #4171 |

## Связь PR и issue

`Fixes #NNNN` в теле PR — стандартный синтаксис GitHub: при мерже указанный issue закрывается автоматически. Четыре issue #4083–#4086 завёл yankawai 05.09, 07.09 выкатил на них PR — **18.09 закрыт последний из четырёх**.

| Issue | Название | PR | Состояние |
|---|---|---|---|
| [#4083](https://github.com/cozystack/cozystack/issues/4083) | clickhouse: Keeper is never created when the application name is longer than 15 characters | #4133 | **закрыт** мержем 17.09 |
| [#4084](https://github.com/cozystack/cozystack/issues/4084) | nats: config.merge.accounts duplicates the generated accounts key and breaks the install | #4134 | **закрыт** мержем 18.09 |
| [#4085](https://github.com/cozystack/cozystack/issues/4085) | harbor/clickhouse: cleanup hook Role lacks watch on persistentvolumeclaims | #4135 | **закрыт** мержем 18.09 |
| [#4086](https://github.com/cozystack/cozystack/issues/4086) | opensearch: data PVCs are left behind after the application is removed | #4136 | **закрыт** мержем 18.09 |

## Issues

| Issue | Автор | Название | Состояние |
|---|---|---|---|
| [#3950](https://github.com/cozystack/cozystack/issues/3950) | IvanHunters | Platform-wide defaults for tenant Talos worker settings | OPEN, `triage/needs-triage`. Наш комментарий 07.09 о create-time дефолтинге, развёрнутый ответ myasnikovdaniil 08.09. Дальше без движения |
| [#3022](https://github.com/cozystack/cozystack/issues/3022) | lexfrei | OpenSearch fails to start in tenant namespaces: privileged init-sysctl violates baseline PodSecurity | OPEN, но **по сути решён**: #4152 поднимает `vm.max_map_count` DaemonSet'ом, #2682 выключил `setVMMaxMapCount`. Автор #4026 предложил закрыть |
| [#4073](https://github.com/cozystack/cozystack/issues/4073) | lexfrei | opensearch-operator: dnsBase stays cluster.local | **закрыт** 17.09 — фикс в #4185 |
| [#3793](https://github.com/cozystack/cozystack/issues/3793) | IvanHunters | Deleting a tenant with a Kafka app hangs the namespace in Terminating (KafkaTopic strimzi.io/topic-operator finalizer) | OPEN. Смерженный 15.09 #3938 на него ссылается в описании, но без `Fixes`, поэтому issue не закрылся автоматически |

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
| [#4026](https://github.com/cozystack/cozystack/pull/4026) | yankawai | Ручка `setVMMaxMapCount` для OpenSearch | Закрыт автором 17.09: main обогнал — #4152 поднимает sysctl DaemonSet'ом, #2682 выключил `setVMMaxMapCount`. Ребейз вернул бы дефолт `true`, обратный смерженному |
| [#3357](https://github.com/cozystack/cozystack/pull/3357) | fuad00 | DNS_BASE для opensearch-operator | Закрыт 10.09: коммит с сохранением авторства довели через #4185, маршрут изменён на `opensearch-operator.manager.dnsBase` |
| [#3294](https://github.com/cozystack/cozystack/pull/3294) | myasnikovdaniil | Golden Talos image, первая версия | Смержен 07.09 в ветку `test/drop-ghcr-mirror`, до main не доехал. Работа — в #4171 |
| [#4007](https://github.com/cozystack/cozystack/pull/4007) | myasnikovdaniil | Удаление ghcr.io pull-through mirror из e2e | Закрыт 09.09 как пустой: #4020 уже всё удалил |
| [#3949](https://github.com/cozystack/cozystack/pull/3949) | IvanHunters | Поля `talos.*` необязательными в схеме | Закрыт 24.08 |
| [#2751](https://github.com/cozystack/cozystack/pull/2751) | SerjioTT | SMTP для Grafana | Закрыт 13.08 stale-ботом, осознанно: #3800 решает задачу на правильном слое |

## Смержено в main

PR из отслеживаемого скоупа, которые уже в `main` своего репозитория. Часть из них подробнее описана выше.

| PR | Автор | Что решил | Смержен |
|---|---|---|---|
| [#4291](https://github.com/cozystack/cozystack/pull/4291) | yankawai | cluster-api: бэкпорт живой миграции перед выводом хоста — CAPK мигрирует VM, а не удаляет (CAPK #374, #389, #392) | 18.09 |
| [#3937](https://github.com/cozystack/cozystack/pull/3937) | yankawai | registry: `kubectl apply --dry-run=server` больше не делает настоящую запись, коды ошибок бэкенда сохраняются | 18.09 |
| [#4100](https://github.com/cozystack/cozystack/pull/4100) | yankawai | linstor: ожидание временных probe-устройств — тома на ZFS 4K-пулах снова набирают реплики (бэкпорт LINBIT#528) | 18.09 |
| [#3935](https://github.com/cozystack/cozystack/pull/3935) | yankawai | kafka: `topics[].config` необязателен, как и у Strimzi | 18.09 |
| [#4014](https://github.com/cozystack/cozystack/pull/4014) | yankawai | mongodb: креды дашборда заполняются при первой установке, живой пароль не ротируется | 18.09 |
| [#4136](https://github.com/cozystack/cozystack/pull/4136) | yankawai | opensearch: post-delete хук удаляет PVC данных — квота тенанта освобождается. Закрыл issue #4086 | 18.09 |
| [#4253](https://github.com/cozystack/cozystack/pull/4253) | yankawai | kubeovn: ретрай настройки сети миграции — живая миграция VM не зависает | 18.09 |
| [#4135](https://github.com/cozystack/cozystack/pull/4135) | yankawai | apps: хуки очистки harbor, clickhouse, mariadb и qdrant получили `watch` и лимит ожидания — рапортуют о завершении после реального удаления PVC. Закрыл issue #4085 | 18.09 |
| [#4134](https://github.com/cozystack/cozystack/pull/4134) | yankawai | nats: `config.merge.accounts` сливается со сгенерированной картой аккаунтов, установка с `users` не падает. Закрыл issue #4084 | 18.09 |
| [#4292](https://github.com/cozystack/cozystack/pull/4292) | yankawai | linstor: опциональный graceful shutdown сателлита на Talos — Secondary-ресурсы DRBD отпускаются при штатном выключении ноды | 18.09 |
| [#4254](https://github.com/cozystack/cozystack/pull/4254) | yankawai | kubevirt: платформенный ключ `kubevirt.migrations` доезжает до `spec.configuration.migrations` | 18.09 |
| [#4184](https://github.com/cozystack/cozystack/pull/4184) | yankawai | linstor: plunger переподключает только зависший peer — отключённая реплика возвращается | 18.09 |
| [#4148](https://github.com/cozystack/cozystack/pull/4148) | yankawai | foundationdb: тенант читает ConfigMap со строкой подключения своей базы | 18.09 |
| [#4133](https://github.com/cozystack/cozystack/pull/4133) | yankawai | clickhouse: Keeper создаётся и при имени приложения длиннее 15 символов. Закрыл issue #4083 | 17.09 |
| [#3934](https://github.com/cozystack/cozystack/pull/3934) | yankawai | kubernetes: сайдкары KubeVirt CSI запинены на версии SIG Storage — контроллер CSI в тенанте больше не падает на CPU без x86-64-v3 | 17.09 |
| [website#697](https://github.com/cozystack/website/pull/697) | yankawai | Документация `kubevirt.migrations` в разделе `next` сайта | 17.09 |
| [#3938](https://github.com/cozystack/cozystack/pull/3938) | yankawai | kafka: топики релиза удаляются до снятия topic operator, переустановка больше не падает | 15.09 |
| [#2682](https://github.com/cozystack/cozystack/pull/2682) | Arsolitt | opensearch: TLS для HTTP API и Dashboards через cert-manager, `setVMMaxMapCount` выключен | 11.09 |
| [#4185](https://github.com/cozystack/cozystack/pull/4185) | lexfrei | opensearch-operator: `DNS_BASE` следует домену платформы — securityadmin на `cozy.local` работает | 10.09 |
| [#4152](https://github.com/cozystack/cozystack/pull/4152) | lexfrei | opensearch-operator: DaemonSet поднимает `vm.max_map_count` на всех нодах, privileged init больше не нужен | 08.09 |
| [#3920](https://github.com/cozystack/cozystack/pull/3920) | lexfrei | CDI v1.66.1: HTTP-импорты в block-тома на 4Kn снова работают, нет шторма `resourceVersion` на prime-PVC | 07.09 |
| [#4095](https://github.com/cozystack/cozystack/pull/4095) | IvanHunters | harbor: явные ресурсы для nginx-прокси, переживает LimitRange тенанта | 06.09 |
