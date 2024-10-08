# Ошибочно определяется географическая принадлежность IP-адресов ресурсов {{ yandex-cloud }}


## Описание проблемы {#issue-description}

* Государственные сайты блокируют подключения с виртуальной машины, считая IP-адрес источника не российским;
* Исходя из информации со сторонних интернет-ресурсов, внешний IP-адрес принадлежит другой стране.

## Решение {#issue-resolution}

Все диапазоны внешних IP-адресов {{ yandex-cloud }} относятся к Российской Федерации. Однако информация о диапазонах IP адресов передается по интернету, и некоторые компании могут обновлять информацию о принадлежности адресов несвоевременно.

Если необходимо, вы можете получить другой публичный IP для своей виртуальной машины. Для этого достаточно остановить ее в [Консоли управления]({{ link-console-main }}) и запустить снова. Если в параметрах виртуальной машины указано использование динамического публичного IP-адреса, при выключении и последующем запуске этой виртуальной машины ей будет назначен новый IP-адрес. При необходимости возможно сделать этот адрес статическим, тогда он не будет изменяться при перезапуске виртуальной машины.

{% note info %}

Диапазоны публичных IP-адресов {{ yandex-cloud }} перечислены на этой [странице](../../../vpc/concepts/ips.md). Проверить принадлежность IP-адреса к определенной стране можно с помощью команды `whois` или на публичном ресурсе [RIPE](https://apps.db.ripe.net/db-web-ui/query).

{% endnote %}

Также вы можете попробовать перенести вашу ВМ в другую зону доступности. Для этого вы можете воспользоваться этой [инструкцией](../../../compute/operations/vm-control/vm-change-zone.md).

Чтобы сделать публичный адрес виртуальной машины статическим, нужно:

1. При запущенной ВМ в консоли облака перейти в раздел **{{ ui-key.yacloud.component.navigation-menu.label_vpc }}**.
1. Перейти в раздел **{{ ui-key.yacloud.vpc.addresses.label_title }}**.
1. Найти в списке нужный адрес.
1. Нажать в строке адреса на кнопку ![***](../../../_assets/options.svg).
1. В выпадающем меню выбрать пункт **{{ ui-key.yacloud.vpc.addresses.button_action-static }}**. 

Подробнее об этом мы пишем в [документации](../../../vpc/operations/set-static-ip.md).