---
title: "Прощавай, sidecar: Ambient режим Istio досяг бета-версії в v1.22"
description: Функції Layer 4 та Layer 7 тепер готові до промислового використання.
publishdate: 2024-05-13
attribution: "Lin Sun (Solo.io), для Комітетів з керівництва та технічного нагляду Istio"
keywords: [ambient,sidecars]
---

Сьогодні революційний новий режим Ambient {{< gloss "панель даних" >}}панелі даних{{< /gloss >}} в Istio досяг бета-версії. Ambient режим розроблений для спрощення операцій, розширення сумісності з застосунками та зменшення витрат на інфраструктуру. Він забезпечує панель даних без sidecar, інтегровану у вашу інфраструктуру, залишаючи основні функції Istio, такі як безпека нульового рівня довіри, телеметрія та управління трафіком.

Ambient режим [був оголошений у вересні 2022 року](/blog/2022/introducing-ambient-mesh/). З того часу наша спільнота впродовж 20 місяців важкої роботи та співпраці, з участю від Solo.io, Google, Microsoft, Intel, Aviatrix, Huawei, IBM, Red Hat та багатьох інших, зробила значний внесок. Статус бета-версії в 1.22 вказує на те, що функції ambient режиму тепер готові до промислового використання з відповідними запобіжними заходами. Це величезна подія для Istio, що забезпечує як функції рівня 4, так і рівня 7 для готовності до використання без sidecar контейнерів.

## Чому режим Ambient? {#why-ambient-mode}

Дослухаючись до відгуків користувачів Istio, ми спостерігали зростаючий попит на можливості мережі для застосунків — але почули, що багато з вас вважали витрати ресурсів та операційну складність sidecar важкими для подолання. Проблеми, якими користувачі sidecar ділилися з нами, включають те, як Istio може зламати застосунки після додавання sidecar контейнерів, велике споживання CPU та памʼяті sidecar контейнерами та незручність вимоги перезапуску застосунків з кожним новим випуском проксі.

Як спільнота, ми спроєктували режим ambient для усунення цих проблем, прибираючи попередні барʼєри складності, з якими стикалися користувачі, які намагалися впровадити сервісну мережу. Новий набір функцій був названий "ambient режим" (режим оточення) оскільки він був призначений бути прозорим для вашого застосунку, забезпечуючи відсутність необхідності додаткової конфігурації для його прийняття, і не вимагав перезапуску застосунків користувачами.

У режимі ambient додати чи видалити застосунки з мережі дуже просто. Тепер ви можете просто [позначити простір імен](/docs/ambient/usage/add-workloads/), і всі застосунки в цьому просторі імен будуть додані до мережі. Це негайно забезпечує весь трафік з mTLS, все без sidecar контейнерів чи потреби перезапуску застосунків.

Перегляньте [блог про представлення Ambient Mesh](/blog/2022/introducing-ambient-mesh/) для отримання додаткової інформації про те, чому ми створили режим ambient.

## Як режим Ambient полегшує адаптацію? {#how-does-ambient-mode-make-adoption-easier}

Режим ambient Istio вводить легкі, спільні проксі рівня 4 (L4) та необовʼязкові проксі рівня 7 (L7), усуваючи необхідність традиційних sidecar проксі з панелі даних. Основна інновація за режимом ambient полягає в тому, що він розділяє обробку L4 і L7 на два окремі шари. Такий шаровий підхід дозволяє вам поступово впроваджувати Istio, забезпечуючи плавний перехід від відсутності мережі до безпечного накладення (L4), до необовʼязкової повної обробки L7, в основі простору імен, за необхідності, по всьому вашому флоту.

Режим ambient працює без будь-яких модифікацій ваших поточних розгортань Kubernetes. Ви можете позначити простір імен, щоб додати всі його навантаження до мережі або вибрати певні розгортання за потреби. Використовуючи режим ambient, користувачі оминають деякі з попередніх обмежувальних елементів моделі sidecar контейнерів. Протоколи "server-send-first" тепер працюють, більшість зарезервованих портів тепер доступні, і можливість для контейнерів оминати sidecar контейнер, або зловмисно, або ні, — усунена.

Легкий спільний L4 проксі називається *[ztunnel](/docs/ambient/overview/#ztunnel)* (тунель нульової довіри). Ztunnel радикально зменшує витрати на запуск мережі, усуваючи потребу в потенційному перевиділенню памʼяті та CPU в кластері для обробки очікуваних навантажень. У деяких випадках економія може перевищувати 90% і більше, при цьому забезпечуючи безпеку нульового рівня довіри за допомогою мTLS з криптографічною ідентичністю, простими політиками авторизації L4 та телеметрією.

Проксі L7 називаються *[waypoints](/docs/ambient/overview/#waypoint-proxies)*. Waypoints обробляють функції L7 такі як маршрутизація трафіку, докладний контроль політики авторизації та стійкість на промисловому рівні. Waypoints працюють поза вашими розгортаннями застосунків і можуть масштабуватися незалежно від ваших потреб, хай то для всього простору імен або для кількох сервісів у просторі імен. В порівнянні з sidecar, вам не потрібен waypoint на кожен pod застосунку, і ви можете ефективно масштабувати свій waypoint залежно від його обсягу, таким чином зберігаючи значні кількості CPU і памʼяті у більшості випадків.

Розділення між рівнем безпеки L4 і рівнем обробки L7 дозволяє поступову адаптацію панелі даних до режиму ambient, на відміну від ранньої бінарної "все-разом" інʼєкції sidecar контейнерів. Користувачі можуть почати з рівня безпеки L4, який пропонує більшість функцій, для яких люди впроваджують Istio (mTLS, політика авторизації та телеметрія). Складна обробка L7, така як повторні спроби, розподіл трафіку, балансування навантаження та збір спостережень, можна додати поступово у разі потреби в кожному конкретному випадку.

## Що в рамках бета-версії? {#what-is-in-the-scope-of-the-beta}

Ми рекомендуємо вам дослідити наступні функції бета-версії режиму ambient у промисловому застосуванні з відповідними запобіжними заходами, після перевірки їх у тестових середовищах:

- [Інсталяція Istio з підтримкою режиму ambient](/docs/ambient/install/).
- [Додавання ваших навантажень до мережі](/docs/ambient/usage/add-workloads/) для отримання мTLS з криптографічною ідентичністю, [політик авторизації L4](/docs/ambient/usage/l4-policy/) та телеметрії.
- [Конфігурація waypoints](/docs/ambient/usage/waypoint/) для [використання функцій L7](/docs/ambient/usage/l7-features/) таких як перемикання трафіку, маршрутизація запитів та контроль докладної політики авторизації.
- Підключення шлюзу вхідного трафіку Istio до навантажень у режимі ambient, підтримуючи всі наявні API Istio.
- Використання `istioctl` для управління waypoint'ами та усунення несправностей ztunnel і waypoint'ів.

### Альфа-функції {#alpha-features}

Багато інших функцій, які ми хочемо включити в режим ambient, були реалізовані, але залишаються в статусі альфа у цьому випуску. Будь ласка, допоможіть перевірити їх, щоб вони могли бути підвищені до бета-версії в 1.23 або пізніше:

- Мільтикластерні встановлення
- Проксіювання DNS
- Взаємодія з sidecar
- Підтримка IPv6/Двох стеків
- Підтримка SOCKS5 (для виводу)
- Класичні API Istio (`VirtualService` та `DestinationRule`)

### Плани {#roadmap}

У нас є кілька функцій, які ще не реалізовані в режимі ambient, але заплановані для майбутніх випусків:

- Контрольований вихідний трафік
- Підтримка мультимереж
- Покращення повідомлень `status` на ресурсах для допомоги в усуненні несправностей та розумінні мережі
- Підтримка віртуальних машин

## Що з sidecar? {#what-about-sidecars}

Sidecar контейнери не зникають і залишаються в Istio. Ви можете продовжувати використовувати sidecar контейнери, і вони залишаться повністю підтримуваними. Для будь-якої функції поза межами альфа- або бета-версії режиму ambient, слід розглянути використання sidecar режиму до того, як функція буде додана до режиму ambient. Деякі випадки використання, такі як перемикання трафіку на основі міток джерел, все ще краще реалізовані за допомогою режиму sidecar. Хоча ми вважаємо, що більшість випадків використання будуть найкраще обслужені мережею в режимі ambient, проєкт Istio залишається відданим постійній підтримці sidecar режиму.

## Спробуйте режим ambient сьогодні {#try-ambient-mode-today}

З випуском 1.22 Istio та бета-версією режиму ambient, тепер простіше ніж будь-коли спробувати Istio на ваших власних навантаженнях. Слідуйте [інструкції з початку роботи](/docs/ambient/getting-started/), щоб дослідити режим ambient, або прочитайте наші нові [посібники користувача](/docs/ambient/usage/), щоб дізнатися, як поступово впроваджувати ambient для mTLS та політики авторизації L4, управління трафіком, розширеної політики авторизації L7 та іншого. Ви можете спілкуватися з розробниками в каналі #ambient на [Istio Slack](https://slack.istio.io), або скористатися форумом discussion на [GitHub](https://github.com/istio/istio/discussions) для будь-яких запитань, які можуть у вас виникнути.