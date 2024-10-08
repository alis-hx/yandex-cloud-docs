# Как отключить защиту от DDoS


## Описание задачи {#case-description}

Необходимо отключить защиту от DDoS на существующем IP.

## Решение {#case-resolution}

Адреса с защитой от DDoS выдаются из отдельного пула IP-адресов. Отключить или включить защиту от DDoS на внешнем IP-адресе, уже подключенном к ВМ, невозможно. 

Вы можете отвязать от ВМ существующий IP и подключить новый, без включенной защиты от DDoS. 

Для этого следует проделать следующие действия в [Консоли управления]({{ link-console-main }}):

1. Перейти на страницу со свойствами виртуальной машины в [Консоли управления]({{ link-console-main }}).
1. На вкладке **{{ ui-key.yacloud.common.overview }}** выбрать ![***](../../../_assets/options.svg) в правой верхней части раздела **Сеть**.
1. Выбрать пункт **{{ ui-key.yacloud.compute.instance.overview.button_remove-public-ip }}**.
1. После удаления публичного IP повторить действие и выбрать **{{ ui-key.yacloud.compute.instance.overview.button_add-public-ip }}**.
1. Во всплывающем окне не выбирать опцию **{{ ui-key.yacloud.common.field_ddos-protection-provider }}**.

{% note alert %}

Это действие приведет к смене публичного IP адреса ВМ и ее недоступности по этому адресу.

{% endnote %}