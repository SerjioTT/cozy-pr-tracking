# Отслеживание PR в cozystack

Обновлено: **2026-09-18**. Репозиторий по умолчанию — `cozystack/cozystack`, иначе указано явно.

На GitHub у issue и PR **общая нумерация**, по номеру их не отличить. Поэтому здесь issue всегда помечены словом — `issue #4083`, а голый `#4133` означает PR.

## Главное за 17–18.09

- **Смержены** #4133 (clickhouse), #3934 (kubernetes CSI) и website#697 (документация к #4254 — раньше самого #4254)
- **#4026 закрыт автором**: проблему OpenSearch уже решили в main #4152 (lexfrei, DaemonSet поднимает `vm.max_map_count` на всех нодах) и #2682 (выключил `setVMMaxMapCount` в чарте). Issue #3022 автор считает закрываемым
- **CI наконец запустили** на большинстве PR из форка. Но на шести несвязанных PR E2E падает на одних и тех же сьютах — см. «E2E падает одинаково»
- **lexfrei одобрил** #4292, #4254, #4253, #4184, #4100, #3935 — вопрос второго апрува владельца кода снят. И запросил изменения в #4291 и #4014
- **yankawai запушил правки** в #3799 (трейлер исправлен), #4135 и #4136 (ограничено ожидание удаления), #3937 и #3936 (ребейз, конфликтов больше нет). CI на новых коммитах снова ждёт одобрения
- issue #4073 закрыт, issue #4083 закрылся мержем #4133

## Требует действия

| Что | Где | Кому и что делать |
|---|---|---|
| **Смержить** | #4148 | Мейнтейнер: ревью достаточно, `pre-commit` и `E2E Tests` зелёные, конфликтов нет |
| **Разобраться с E2E** | #4292, #4254, #4253, #4134, #4100, #3800 | Мейнтейнер: одинаковые падения на несвязанных PR, подробности ниже. Все шесть одобрены или ждут только снятия старого запроса |
| **Перезапустить публикацию для E2E** | #3935 | Мейнтейнер: E2E не запускался — упал шаг `Apply pr.patch (reconstruct digest-pinned tree)` в публикации образов |
| **Повторное ревью после правок** | #3799, #4135, #4136 | scooby87: правки по его запросам запушены 17.09 |
| **Повторное ревью после ребейза** | #3937 — lexfrei, #3936 — IvanHunters | Конфликты разрешены 17.09 |
| **Запустить CI на новых коммитах** | #3799, #4135, #4136, #3937, #3936 | Мейнтейнер: после пушей 17.09 workflow снова в `action_required` |
| **Снять устаревший запрос изменений** | #3800 — IvanHunters с 17.08; #3956 — IvanHunters с 01.09 | Повторный проход или dismiss. На #3800 два апрува, на #3956 один |
| **Поправить тестовый файл** | #4291 | yankawai: новый bats-файл ставит три `EXIT`-трапа и не объявляет их, из-за чего `make unit-tests` красный, а workflow «Pull Request» падает до E2E |
| **Закрепить тестами два свойства** | #4014 | yankawai: lexfrei показал, что можно удалить и защиту от ротации пароля, и логику подхвата существующего Secret — сьют остаётся зелёным |
| **Закрыть issue** | issue #3022, issue #3793 | Мейнтейнер: #3022 по словам автора #4026 решён мержем #4152 и #2682. #3793 упомянут в описании смерженного #3938, но без `Fixes`. `[вывод]` Скорее всего #3938 его и закрывает — стоит проверить и закрыть |

**В одобренные PR ничего не пушить** — новый коммит снимет апрув (`dismiss_stale_reviews_on_push: true`).

## Наши PR — открытые

| PR | Автор | Название | Что решает | Состояние |
|---|---|---|---|---|
| [#4292](https://github.com/cozystack/cozystack/pull/4292) | yankawai | fix(linstor): add opt-in graceful satellite shutdown on Talos | Конфигурация Talos в cozystack убирает systemd-страховку DRBD при выключении и ничем её не заменяет. preStop-хук за флагом `talos.gracefulShutdown.enabled` перед остановкой сателлита отпускает **неиспользуемые Secondary**-ресурсы DRBD, чтобы их backing-устройства, в том числе ZFS zvol, освободились при штатном выключении ноды. Без force, без демоута, без новых прав | **APPROVED** — lexfrei и scooby87. `pre-commit` зелёный, **E2E красный** — общая картина, см. ниже. Выключено по умолчанию: preStop срабатывает и на обычном рестарте сателлита, а у Piraeus нет общекластерного барьера, поэтому одновременные рестарты могут приостановить I/O. `size/L`, +444/−0 |
| [#4291](https://github.com/cozystack/cozystack/pull/4291) | yankawai | fix(cluster-api): backport live migration before host eviction | CAPK v0.1.10 при выводе хоста просто дренирует и удаляет гостевую VM вместо живой миграции, ищет в гостевом кластере имя физического хоста, а пересозданный гость может остаться в cordon. Бэкпорт CAPK #374, #389 и #392 | **CHANGES_REQUESTED** — lexfrei 17.09: bats-файл ставит три незаявленных `EXIT`-трапа, `make unit-tests` красный. Workflow «Pull Request» упал, E2E не запускался. Пин образа — стоковый апстрим, бэкпорт заработает только после пересборки образа на релизном пути; репозиторий `cluster-api-provider-kubevirt` должен существовать в build registry до первой продвинутой сборки. Одобрен scooby87. `size/XXL`, +1298/−0 |
| [#4254](https://github.com/cozystack/cozystack/pull/4254) | yankawai | feat(kubevirt): expose migration configuration through platform values | Настройки живой миграции VM на уровне кластера задавались только ручным патчем KubeVirt CR. Добавлен платформенный ключ `kubevirt.migrations`, доезжающий до `spec.configuration.migrations` | **APPROVED** — lexfrei и scooby87. **E2E красный** — общая картина. Документация website#697 уже смержена. Пересекается по файлам с открытым #3989. `size/L`, +252/−3 |
| [#4253](https://github.com/cozystack/cozystack/pull/4253) | yankawai | fix(kubeovn): retry VM migration setup after scheduling | Kube-OVN мог потерять настройку сети миграции VM, если миграция входила в `Scheduling` раньше, чем появлялся целевой под — и целевой под навсегда ждал сеть. Бэкпорт kubeovn/kube-ovn#6834 на v1.15.10 плюс собственная защита от гонки создания целевого пода | **APPROVED** — lexfrei и scooby87. **E2E красный** — общая картина. Бэкпорт снимается при kube-ovn v1.15.14+. Метка `area/uncategorized`: в маппинге лейблера есть `kube-ovn`, но нет `kubeovn`. `size/L`, +152/−0 |
| [#4184](https://github.com/cozystack/cozystack/pull/4184) | yankawai | fix(linstor): reconnect only the stalled peer in the satellite plunger | `drbd_stall_act` отключал один зависший peer, а поднимал через `drbdadm connect res` по всем соединениям сразу. Живые peer'ы валили команду, и отключённый так и оставался отключённым | **APPROVED** — lexfrei и scooby87. `pre-commit` зелёный, **E2E идёт** на момент обновления. `size/XS`, +1/−1 |
| [#4148](https://github.com/cozystack/cozystack/pull/4148) | yankawai | fix(foundationdb): let tenant roles read the connection ConfigMap | Тенант не мог получить строку подключения к своей же FoundationDB: прав на чтение `<release>-config` не было, в dashboard он не выводился | **APPROVED** — scooby87. `pre-commit` и E2E зелёные. **Готов к мержу.** Сетевая доступность самой БД — отдельная тема, Service у FoundationDB нет. `size/L`, +117/−2 |
| [#4136](https://github.com/cozystack/cozystack/pull/4136) | yankawai | fix(opensearch): remove the data PVCs when the application is deleted | После удаления приложения тома данных оставались `Bound` и занимали квоту тенанта. Добавлен post-delete хук по образцу clickhouse | **CHANGES_REQUESTED** — scooby87: нужен лимит ожидания. **Исправлено 17.09** коммитом «keep the PVC cleanup hook from holding the uninstall», ждёт повторного ревью и запуска CI. `Fixes issue #4086` |
| [#4135](https://github.com/cozystack/cozystack/pull/4135) | yankawai | fix(apps): let the cleanup hooks watch the objects they delete | У post-delete хуков harbor, clickhouse, mariadb и qdrant не было `watch`: хук рапортовал о завершении, пока PVC ещё удалялись | **CHANGES_REQUESTED** — scooby87: с правом `watch` ожидание стало реальным, а лимита не было. **Исправлено 17.09** коммитом «bound the wait the cleanup hooks now perform», ждёт повторного ревью и запуска CI. `Fixes issue #4085` |
| [#4134](https://github.com/cozystack/cozystack/pull/4134) | yankawai | fix(nats): merge config.merge.accounts with the generated accounts map | При заданных `users` ключ `accounts` рендерился дважды, установка падала на пост-рендере | **APPROVED** — scooby87. **E2E красный** — общая картина. `Fixes issue #4084` |
| [#4100](https://github.com/cozystack/cozystack/pull/4100) | yankawai | fix(linstor): wait for temporary probe devices | Бэкпорт LINBIT#528. Гонка с udev на ZFS оставляла пул без размера блока, и тома, стартовавшие на такой ноде, навсегда получали `block-size 512` и не набирали реплики на 4K-пулах | **APPROVED** — lexfrei и scooby87. **E2E красный** — общая картина. Про апстрим — раздел «LINSTOR» ниже. На нашем кластере обойдено вручную. `size/XL`, +664/−0 |
| [#4014](https://github.com/cozystack/cozystack/pull/4014) | yankawai | fix(mongodb): fill the dashboard credentials on a first install | Secret `<release>-credentials` для дашборда навсегда оставался с пустыми `password` и `uri`. Чарт теперь сам владеет учётными данными и не ротирует живой пароль | **CHANGES_REQUESTED** — lexfrei 17.09: шаблон верный, но ни одно из двух обещанных свойств не закреплено тестом. `pre-commit` и E2E зелёные. Одобрен scooby87 |
| [#3937](https://github.com/cozystack/cozystack/pull/3937) | yankawai | fix(registry): preserve write options in aggregated storage | Агрегированный API терял опции записи: `kubectl apply --dry-run=server` по `apps.cozystack.io` выполнял **настоящую** запись и создавал релиз, а отказы бэкенда сводились к 500 | **CHANGES_REQUESTED** — lexfrei 11.09. **Ребейз 17.09**, конфликтов нет. Ждёт повторного ревью и запуска CI. `size/XXL` |
| [#3936](https://github.com/cozystack/cozystack/pull/3936) | yankawai | fix(rabbitmq): right-size the default resources preset to s1.nano | На дефолтном пресете `t1.nano` брокер получал OOMKill, набирал 10 рестартов и не становился Ready. Пресет поднят до `s1.nano` | **CHANGES_REQUESTED** — IvanHunters 27.08. **Ребейз 17.09**, конфликтов нет. Ждёт повторного ревью и запуска CI |
| [#3935](https://github.com/cozystack/cozystack/pull/3935) | yankawai | fix(kafka): make topics[].config optional | Схема отвергала топик без `config`, хотя Strimzi считает его необязательным | **APPROVED** — lexfrei и scooby87. E2E не запускался: упала публикация образов на шаге `Apply pr.patch` |
| [#3800](https://github.com/cozystack/cozystack/pull/3800) | yankawai | feat(monitoring): add optional email receiver to alertmanager | Почтовый канал алертинга через Alertmanager рядом с Alerta, пароль SMTP монтируется из Secret. Реализовано наше предложение: список `alertnames` и настраиваемые `severities` | **CHANGES_REQUESTED** — висит запрос IvanHunters от 17.08, хотя lexfrei (14.09) и scooby87 (17.09) одобрили. **E2E красный** — общая картина |
| [#3799](https://github.com/cozystack/cozystack/pull/3799) | yankawai | fix(linstor): use severity warning instead of warn in prometheus rules | Семь алертов LINSTOR/DRBD с `severity: warn` Alerta отбрасывала с ошибкой 500, и они молча терялись | **CHANGES_REQUESTED** — scooby87: единственным блокером был трейлер коммита. **Исправлено 17.09**, теперь везде `Assisted-by: LLM`. Ждёт повторного ревью и запуска CI |

## Чужие PR — открытые

| PR | Автор | Название | Что решает | Состояние |
|---|---|---|---|---|
| [#4171](https://github.com/cozystack/cozystack/pull/4171) | myasnikovdaniil | feat(kubernetes): boot tenant worker disks from a shared golden Talos image (CDI clone) | Воркеры тенантных кластеров грузятся клоном общего golden-образа Talos вместо того, чтобы каждый тянуть ~4 GiB по HTTP. Каталог настраивается платформенным ключом `kubernetesWorkerImage` | **CHANGES_REQUESTED** — lexfrei 09.09. **Конфликтует с main**, E2E красный. Дефолтом clone не становится, выбор per-pool через `osImage.builtin` |
| [#3956](https://github.com/cozystack/cozystack/pull/3956) | myasnikovdaniil | fix(api): repair an empty required field instead of failing the release | Пустое обязательное поле в спеке роняло установку всего релиза | **CHANGES_REQUESTED** — висит запрос IvanHunters от 01.09, lexfrei одобрил 11.09. E2E красный. Важен как шов на write-пути для create-time дефолтинга из issue #3950 |

## Живая миграция VM — как связаны четыре PR

15.09 yankawai открыл пачку из четырёх PR вокруг одной темы — чтобы VM переезжали между хостами живьём, а не убивались.

| PR | Слой | Что даёт | Состояние |
|---|---|---|---|
| #4254 | платформа → KubeVirt | настраивать миграции платформенными values | одобрен, E2E красный |
| website#697 | документация | описание #4254 | **смержен 17.09 — раньше самого #4254** |
| #4253 | сеть, Kube-OVN | миграция не зависает на настройке сети целевого пода | одобрен, E2E красный |
| #4291 | Cluster API, CAPK | при выводе хоста VM сначала мигрирует, потом дренируется | запрошены изменения |

Документация уже в разделе `next` сайта, а сама настройка `kubevirt.migrations` в main ещё не попала.

## E2E падает одинаково на несвязанных PR

17.09 около 21:50 на большинстве PR из форка запустили E2E. Шесть из них упали на **одних и тех же сьютах**, хотя меняют совершенно разные вещи:

| PR | Что меняет | Упавшие сьюты |
|---|---|---|
| #4292 | linstor, shutdown на Talos | backup-roundtrip для redis, rabbitmq, postgres, mongodb; kafka-3-metadata-roundtrip; kafka; opensearch |
| #4254 | kubevirt, платформа | backup-roundtrip для redis, rabbitmq, postgres, mongodb; kafka-3-metadata-roundtrip; kafka |
| #4253 | kube-ovn | clickhouse-2-backup-roundtrip; kafka; opensearch |
| #4134 | nats | backup-roundtrip для redis, rabbitmq, postgres, mongodb; kafka-3-metadata-roundtrip |
| #4100 | linstor, probe-том | backup-roundtrip для redis, rabbitmq, postgres, mongodb; kafka-3-metadata-roundtrip; kafka; opensearch |
| #3800 | monitoring | backup-roundtrip для redis, rabbitmq, postgres, mongodb; kafka-3-metadata-roundtrip; kafka |

При этом E2E у #4148 и #4014 прошёл, а в `cozystack/cozystack` открыты issue #4262 («postgres round-trip fails on a plugin/barmanObjectStore conflict») и #4258 («CNPG BackupJob fails…») — оба от 15.09.

`[вывод]` Одинаковый набор падающих сьютов на PR, которые не трогают ни бэкапы, ни kafka, ни opensearch, указывает на общую причину, а не на регрессии в самих PR.

`[предположение]` Причина связана с открытыми issue по backup round-trip. Проверить можно так: посмотреть, падают ли те же сьюты на свежем прогоне из ветки основного репозитория, не из форка.

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
| **ничего — можно мержить** | #4148 |
| общее падение E2E | #4292, #4254, #4253, #4134, #4100 |
| E2E ещё идёт | #4184 |
| публикация образов для E2E | #3935 |
| повторное ревью и запуск CI после правок | #3799, #4135, #4136, #3937, #3936 |
| тесты на два свойства | #4014 |
| тестовый файл, сборка образа | #4291 |
| неснятый запрос изменений и E2E | #3800, #3956 |
| конфликт, правки и E2E | #4171 |

## Связь PR и issue

`Fixes #NNNN` в теле PR — стандартный синтаксис GitHub: при мерже указанный issue закрывается автоматически. Четыре issue #4083–#4086 завёл yankawai 05.09, а 07.09 выкатил на них PR.

| Issue | Название | PR | Состояние |
|---|---|---|---|
| [#4083](https://github.com/cozystack/cozystack/issues/4083) | clickhouse: Keeper is never created when the application name is longer than 15 characters | #4133 | **закрыт** мержем 17.09 |
| [#4084](https://github.com/cozystack/cozystack/issues/4084) | nats: config.merge.accounts duplicates the generated accounts key and breaks the install | #4134 | открыт |
| [#4085](https://github.com/cozystack/cozystack/issues/4085) | harbor/clickhouse: cleanup hook Role lacks watch on persistentvolumeclaims | #4135 | открыт |
| [#4086](https://github.com/cozystack/cozystack/issues/4086) | opensearch: data PVCs are left behind after the application is removed | #4136 | открыт |

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
| [LINBIT/linstor-server#528](https://github.com/LINBIT/linstor-server/pull/528) | yankawai | satellite: wait for probe devices before reading block information | **MERGED** 08.09 в `master`, коммит `6e5557839`, вошёл в релиз v1.35.1 от 09.09. Закрывает issue LINBIT#527 |

Пакет `packages/system/linstor` собирает `piraeus-server` **из исходников**: берёт тег из `LINSTOR_VERSION ?= 1.33.3` и накладывает патчи из `images/piraeus-server/patches/`.

| | Версия |
|---|---|
| Прибито в cozystack | **1.33.3** |
| Актуальный релиз LINSTOR | **1.35.1** от 09.09 |
| Разрыв | **448 коммитов**, два минорных релиза |

У LINSTOR **нет релизных веток** — только `master` и рабочие ветки. Бэкпортов в 1.33.x не бывает, поэтому на сегодняшнем пине патч #4100 — единственный способ получить фикс.

- у патча **определённое условие снятия**: при подъёме `LINSTOR_VERSION` до 1.35.1+ его надо удалить, иначе сборка образа упадёт. Механизма, который это заметит, в репозитории нет
- **запроса на бамп версии не существует**. Похожий issue #3857 — про `linstor-csi`, это другой компонент
- при бампе из шести патчей **осталось бы три**:

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
| [#4133](https://github.com/cozystack/cozystack/pull/4133) | yankawai | clickhouse: Keeper создаётся и при имени приложения длиннее 15 символов. Закрыл issue #4083 | 17.09 |
| [#3934](https://github.com/cozystack/cozystack/pull/3934) | yankawai | kubernetes: сайдкары KubeVirt CSI запинены на версии SIG Storage — контроллер CSI в тенанте больше не падает на CPU без x86-64-v3 | 17.09 |
| [website#697](https://github.com/cozystack/website/pull/697) | yankawai | Документация `kubevirt.migrations` в разделе `next` сайта | 17.09 |
| [#3938](https://github.com/cozystack/cozystack/pull/3938) | yankawai | kafka: топики релиза удаляются до снятия topic operator, переустановка больше не падает | 15.09 |
| [#2682](https://github.com/cozystack/cozystack/pull/2682) | Arsolitt | opensearch: TLS для HTTP API и Dashboards через cert-manager, `setVMMaxMapCount` выключен | 11.09 |
| [#4185](https://github.com/cozystack/cozystack/pull/4185) | lexfrei | opensearch-operator: `DNS_BASE` следует домену платформы — securityadmin на `cozy.local` работает | 10.09 |
| [#4152](https://github.com/cozystack/cozystack/pull/4152) | lexfrei | opensearch-operator: DaemonSet поднимает `vm.max_map_count` на всех нодах, privileged init больше не нужен | 08.09 |
| [#3920](https://github.com/cozystack/cozystack/pull/3920) | lexfrei | CDI v1.66.1: HTTP-импорты в block-тома на 4Kn снова работают, нет шторма `resourceVersion` на prime-PVC | 07.09 |
| [#4095](https://github.com/cozystack/cozystack/pull/4095) | IvanHunters | harbor: явные ресурсы для nginx-прокси, переживает LimitRange тенанта | 06.09 |
