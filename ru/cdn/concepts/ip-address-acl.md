# Политика доступа по IP-адресам

С помощью _политики доступа по IP-адресам_ можно ограничить доступ к контенту, распространяемому через CDN, задавая для [CDN-ресурсов](./resource.md) _разрешающую_ или _блокирующую_ политику.

Вы можете настроить политику доступа при [создании](../operations/resources/create-resource.md) или [изменении](../operations/resources/configure-basics.md) CDN-ресурса с помощью [консоли управления]({{ link-console-main }}), [CLI](../../cli/quickstart.md), [{{ TF }}]({{ tf-provider-resources-link }}/cdn_resource) и [API](../../api-design-guide/concepts/general.md).

Политика доступа по IP-адресам определяется [типом политики](#policy-type) и [списком IP-адресов](#ip-list), которые будут исключены из политики.

Подробнее о политике доступа по IP-адресам см. в [документации CDN-провайдера](https://support.edgecenter.ru/knowledge_base/item/257918?sid=57227) EdgeЦентр.

## Тип политики {#policy-type}

Для ресурса можно задать разрешающую или блокирующую политику.

Разрешающая политика открывает доступ к контенту CDN-ресурса для любых IP-адресов, кроме заданных пользователем в списке исключений. Политика задается значением `allow`.

Блокирующая политика запрещает доступ к контенту CDN-ресурса для любых IP-адресов, кроме заданных пользователем в списке исключений. Политика задается значением `deny`.

Для CDN-ресурса можно активировать либо одну разрешающую, либо одну блокирующую политику доступа. Настроить для одного CDN-ресурса одновременно более одной политики доступа по IP-адресам нельзя.

## Список исключенных IP-адресов {#ip-list}

Исключаемые из политики доступа IP-адреса должны быть указаны с префиксом подсети в [нотации CIDR](https://ru.wikipedia.org/wiki/Бесклассовая_адресация), например: `192.168.3.2/32` или `2a03:d000:2980:7::8/128`. Для каждого заданного пользователем адреса сервис вычисляет диапазон IP-адресов сети, к которой относится этот адрес, и добавляет в список исключений все адреса из этого диапазона. 

Например, если пользователь укажет адрес `1.2.3.4/24`, то сервис добавит в список исключений весь диапазон IP-адресов от `1.2.3.1` до `1.2.3.254`, потому что все они принадлежат сети `1.2.3.0/24`, в которую входит заданный пользователем адрес.

В список исключенных IP-адресов можно добавить [несколько](./limits.md#cdn-limits) IP-адресов (префиксов подсети).

При изменении списка исключенных IP-адресов существующий список исключений будет полностью перезаписан. Чтобы добавить новые IP-адреса в список исключений или удалить из него ненужные адреса, заново задайте полный, содержащий все необходимые изменения, актуальный список исключенных IP-адресов.

#### См. также {#see-also}

* [{#T}](../operations/resources/create-resource.md)
* [{#T}](../operations/resources/configure-basics.md)