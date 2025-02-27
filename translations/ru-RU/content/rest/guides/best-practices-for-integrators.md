---
title: Рекомендации для интеграторов
intro: 'Создайте приложение, которое надежно взаимодействует с API-интерфейсами {% ifversion fpt or ghec %}{% data variables.product.prodname_dotcom %}{% else %}{% data variables.product.product_name %}{% endif %} и обеспечивает максимально эффективное взаимодействие с пользователями.'
redirect_from:
  - /guides/best-practices-for-integrators
  - /v3/guides/best-practices-for-integrators
versions:
  fpt: '*'
  ghes: '*'
  ghae: '*'
  ghec: '*'
topics:
  - API
shortTitle: Integrator best practices
ms.openlocfilehash: bdfc2449946e40b017dc028869deb7991d5a344a
ms.sourcegitcommit: 6185352bc563024d22dee0b257e2775cadd5b797
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/09/2022
ms.locfileid: '148193523'
---
Хотите осуществить интеграцию с платформой GitHub? [Тогда вы обратились по адресу](https://github.com/integrations). Это руководство поможет вам создать приложение, максимально удобное для пользователей *и* обеспечивающее надежное взаимодействие с API. 

## Безопасная доставка полезных данных из GitHub

Очень важно защитить [полезные данные, передаваемые из GitHub][event-types]. Хотя личная информация (например, пароли) никогда не передается в полезных данных, нельзя допускать утечки *никаких* сведений. К конфиденциальным сведениям могут относиться адрес электронной почты участника или имена частных репозиториев.

Существует ряд мер, которыми можно защитить полезные данные, получаемые из GitHub.

1. Убедитесь в том, что принимающий сервер подключен по HTTPS. По умолчанию GitHub проверяет SSL-сертификаты при доставке полезных данных.{% ifversion fpt or ghec %}
1. Вы можете добавить [IP-адрес, используемый при доставке перехватчиков](/github/authenticating-to-github/about-githubs-ip-addresses), в список разрешений сервера. Чтобы всегда проверять правильный IP-адрес, можно [использовать конечную точку `/meta`](/rest/reference/meta#meta) для определения используемого нами адреса.{% endif %}
1. Предоставьте [секретный токен](/webhooks/securing/), чтобы обеспечить уверенность в том, что полезные данные поступают из GitHub. Применяя секретный маркер, вы гарантируете, что все данные, получаемые сервером, точно поступают из GitHub. В идеале необходимо предоставить отдельный секретный токен *для каждого пользователя* службы. Если один токен будет скомпрометирован, другие пользователи затронуты не будут.

## Предпочтительное использование асинхронных задач вместо синхронных

GitHub ожидает, что интеграция ответит в течение {% ifversion fpt or ghec %}10{% else %}30{% endif %} секунд после получения полезных данных веб-перехватчика. Если службе требуется больше времени на ответ, GitHub завершает подключение и полезные данные теряются.

Так как невозможно предсказать, насколько быстро ответит служба, вся "реальная работа" должна выполняться в фоновом задании. Примерами библиотек, которые могут выполнять постановку в очередь и обработку фоновых заданий, являются [Resque](https://github.com/resque/resque/) (для Ruby), [RQ](http://python-rq.org/) (для Python) или [RabbitMQ](http://www.rabbitmq.com/) (для Java).

Обратите внимание, что даже при выполнении фонового задания GitHub по-прежнему ожидает, что сервер ответит в течение {% ifversion fpt or ghec %}10{% else %}30{% endif %} секунд. Ваш сервер должен подтвердить получение полезных данных, отправив какой-либо ответ. Очень важно, чтобы служба проводила все проверки полезных данных как можно скорее, чтобы вы могли точно сообщить, будет ли сервер продолжать выполнение запроса.

## Использование соответствующих кодов состояния HTTP при ответе на запросы GitHub

У каждого веб-перехватчика есть собственный раздел "Последние доставки", в котором указывается, было ли развертывание успешным.

![Представление "Последние доставки"](/assets/images/webhooks_recent_deliveries.png)

Для информирования пользователей следует использовать правильные коды состояния HTTP. С помощью такого кода, как `201` или `202`, можно подтвердить получение полезных данных, которые не будут обрабатываться (например, доставленных ветвью не по умолчанию). Зарезервируйте код ошибки `500` для неустранимых сбоев.

## Предоставление пользователю максимально подробных сведений

Пользователи могут изучать ответы сервера, отправляемые обратно в GitHub. Сообщения должны быть понятны и информативны.

![Просмотр ответа касательно полезных данных](/assets/images/payload_response_tab.png)

## Следование всем перенаправлениям, отправляемым API

GitHub явным образом сообщает о перемещении ресурса, предоставляя код состояния перенаправления. Вам необходимо следовать этим перенаправлениям. В каждом ответе о перенаправлении задается заголовок `Location` с новым универсальным кодом ресурса (URI) для перехода. Если вы получили перенаправление, лучше всего обновить код для использования нового кода URI, чтобы не запрашивать устаревший путь, который, возможно, удален.

Мы предоставили [список кодов состояния HTTP](/rest#http-redirects) для отслеживания перенаправлений при разработке приложения.

## Нежелательность анализа URL-адресов вручную

Ответы API часто содержат данные в виде URL-адресов. Например, при запросе репозитория мы отправляем ключ `clone_url` с URL-адресом, который можно использовать для клонирования репозитория.

Для стабильной работы приложения не следует пытаться анализировать эти данные или пытаться угадать формат будущих URL-адресов. Если мы решим изменить URL-адрес, работа приложения может нарушиться.

Например, при работе с результатами с разбивкой на страницы кажется логичным, что следует добавлять `?page=<number>` в конец URL-адреса. Избегайте такого подхода. Дополнительные сведения о надежном выполнении результатов с разбивкой на страницы см. [в разделе Использование разбиения на страницы в REST API](/rest/guides/using-pagination-in-the-rest-api).

## Проверка типа события и действия перед обработкой события

Существует множество [типов событий веб-перехватчиков][event-types], и с каждым событием может быть связано несколько действий. По мере расширения набора функций GitHub мы иногда будем добавлять новые типы событий или новые действия к существующим типам событий. Перед обработкой веб-перехватчика приложение должно явным образом проверить тип и действие события. С помощью заголовка запроса `X-GitHub-Event` можно узнать, какое событие было получено, чтобы обработать его соответствующим образом. Аналогичным образом, у полезных данных есть ключ верхнего уровня `action`, с помощью которого можно узнать, какое действие было выполнено с соответствующим объектом.

Например, если для веб-перехватчика GitHub настроено действие "Отправить мне **все**", приложение начнет получать новые типы событий и действия по мере их добавления. Поэтому **не рекомендуется использовать предложения типа catch-all else**. Рассмотрим следующий пример кода:

```ruby
# Not recommended: a catch-all else clause
def receive
  event_type = request.headers["X-GitHub-Event"]
  payload    = request.body

  case event_type
  when "repository"
    process_repository(payload)
  when "issues"
    process_issues(payload)
  else
    process_pull_requests
  end
end
```

В этом примере методы `process_repository` и `process_issues` будут вызываться правильно при получении события `repository` или `issues`. Однако любой другой тип события приведет к вызову `process_pull_requests`. По мере добавления новых типов событий это приведет к неправильному поведению, и новые типы событий будут обрабатываться так же, как событие `pull_request`.

Вместо этого мы рекомендуем явно проверять типы событий и действовать соответствующим образом. В следующем примере кода мы явным образом проверяем событие `pull_request`, а предложение `else` просто сообщает, что получен новый тип события:

```ruby
# Recommended: explicitly check each event type
def receive
  event_type = request.headers["X-GitHub-Event"]
  payload    = JSON.parse(request.body)

  case event_type
  when "repository"
    process_repository(payload)
  when "issues"
    process_issue(payload)
  when "pull_request"
    process_pull_requests(payload)
  else
    puts "Oooh, something new from GitHub: #{event_type}"
  end
end
```

Так как у каждого события также может быть несколько действий, рекомендуется проверять действия аналогичным образом. Например, у [`IssuesEvent`](/webhooks/event-payloads/#issues) имеется несколько возможных действий. К ним относятся `opened` при создании проблемы, `closed` при закрытии проблемы и `assigned` при назначении проблемы пользователю.

Помимо добавления типов событий, мы также можем добавлять новые действия для существующий событий. Поэтому при проверке действия события также **не рекомендуется использовать предложения типа catch-all else**. Вместо этого мы рекомендуем явно проверять действия события, как и в случае с типом события. Пример выглядит очень похоже на то, что было предложено для типов событий выше:

```ruby
# Recommended: explicitly check each action
def process_issue(payload)
  case payload["action"]
  when "opened"
    process_issue_opened(payload)
  when "assigned"
    process_issue_assigned(payload)
  when "closed"
    process_issue_closed(payload)
  else
    puts "Oooh, something new from GitHub: #{payload["action"]}"
  end
end
```

В этом примере действие `closed` проверяется перед вызовом метода `process_closed`. Все неопознанные действия регистрируются для проверки в будущем.

{% ifversion fpt or ghec or ghae %}

## Ограничения скорости

[Ограничение скорости](/rest/overview/resources-in-the-rest-api#rate-limiting) API GitHub обеспечивает быструю работу и доступность API для всех пользователей.

Если ограничение скорости достигнуто, следует сделать паузу и продолжить выполнять запросы, когда это будет разрешено. В противном случае приложение может быть заблокировано.

Вы можете в любой момент [проверить состояние ограничения скорости](/rest/reference/rate-limit). Запрос проверки ограничения скорости не учитывается в общем количестве запросов.

## Дополнительные ограничения скорости

[Дополнительные ограничения скорости](/rest/overview/resources-in-the-rest-api#secondary-rate-limits) — еще один способ обеспечения доступности API.
Чтобы избежать достижения этих ограничений, следуйте приведенным ниже рекомендациям в приложении.

* Выполняйте запросы с проверкой подлинности или используйте идентификатор клиента и секрет приложения. В отношении запросов без проверки подлинности действуют более строгие дополнительные ограничения скорости.
* Выполняйте запросы для одного пользователя или идентификатора клиента последовательно. Не выполняйте запросы для одного пользователя или идентификатора клиента параллельно.
* Если вы выполняете много запросов `POST`, `PATCH`, `PUT` или `DELETE` для одного пользователя или идентификатора клиента, между ними следует делать паузу длительностью по крайней мере в одну секунду.
* Если ограничение достигнуто, используйте заголовок ответа `Retry-After`, чтобы замедлить выполнение запросов. Заголовок `Retry-After` всегда имеет целочисленное значение, представляющее количество секунд, которое следует подождать, прежде чем выполнять запросы снова. Например, `Retry-After: 30` означает, что перед отправкой дополнительных запросов следует подождать 30 секунд.
* Запросы, которые создают содержимое, активирующее уведомления, например проблемы, комментарии и запросы на вытягивание, могут быть ограничены дополнительно, и заголовок `Retry-After` в ответ не включается. Создавайте такое содержимое с разумной частотой, чтобы избежать достижения ограничения.

Мы оставляем за собой право по мере необходимости изменять эти правила, чтобы обеспечить доступность.

{% endif %}

## Обработка ошибок API

Даже если в коде нет неполадок, при попытке доступа к API могут возникнуть повторяющиеся ошибки.

Вместо того чтобы игнорировать повторяющиеся коды состояния `4xx` и `5xx`, следует убедиться в том, что вы правильно взаимодействуете с API. Например, если конечная точка запрашивает строку, а вы передаете числовое значение, то произойдет ошибка проверки `5xx` и вызов не будет выполнен. Аналогичным образом, попытка доступа к неавторизованной или несуществующей конечной точке приведет к ошибке `4xx`.

Намеренное игнорирование повторяющихся ошибок проверки может привести к временному блокированию приложения из-за нарушения.

[event-types]: /webhooks/event-payloads
