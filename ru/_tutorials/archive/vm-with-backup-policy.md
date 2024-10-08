# Автоматическая привязка политики резервного копирования {{ backup-full-name }} к ВМ


В {{ yandex-cloud }} можно создать виртуальную машину на базе [поддерживаемого {{ backup-name }} образа](../../backup/concepts/vm-connection.md#os), к которой будет автоматически привязана [политика резервного копирования](../../backup/concepts/policy.md). Для этого нужно передать в [метаданных](../../compute/concepts/vm-metadata.md) скрипт для установки агента резервного копирования и идентификаторы нужных политик. Указанные политики автоматически привяжутся к ВМ после того, как ВМ и агент создадутся, инициализируются и запустятся.

Чтобы создать виртуальную машину с автоматической привязкой к политике резервного копирования {{ backup-name }}:

1. [Подготовьте облако к работе](#before-begin).
1. [Активируйте сервис](#service-activate).
1. [Создайте сервисный аккаунт](#create-sa).
1. [Создайте облачную сеть и подсети](#create-network).
1. [Создайте и настройте группу безопасности](#create-sg).
1. [Создайте политику резервного копирования](#create-policy).
1. [Создайте ВМ](#create-vm).

Если созданные ресурсы вам больше не нужны, [удалите их](#clear-out).

## Подготовьте облако к работе {#before-begin}

{% include [before-you-begin](../_tutorials_includes/before-you-begin.md) %}

### Необходимые платные ресурсы {#paid-resources}

В стоимость поддержки инфраструктуры входит:

* плата за вычислительные ресурсы ВМ (см. [тарифы {{ compute-full-name }}](../../compute/pricing.md#prices-instance-resources));
* плата за диски ВМ (см. [тарифы {{ compute-full-name }}](../../compute/pricing.md#prices-storage));
* плата за использование динамического внешнего IP-адреса (см. [тарифы {{ vpc-full-name }}](../../vpc/pricing.md#prices-public-ip));
* плата за подключенные к сервису {{ backup-name }} ВМ и объем резервных копий (см. [тарифы {{ backup-full-name }}](../../backup/pricing.md#rules)).

## Активируйте сервис {#service-activate}

{% note info %}

Минимальная роль в [каталоге](../../resource-manager/concepts/resources-hierarchy.md#folder), необходимая для [активации сервиса](../../backup/concepts/index.md#providers), — `backup.editor` (см. [описание роли](../../backup/security/index.md#backup-editor)).

{% endnote %}

Чтобы активировать сервис:

{% list tabs group=instructions %}

- Консоль управления {#console}

  1. В [консоли управления]({{ link-console-main }}) выберите каталог, в котором вы хотите создать ВМ с подключением к {{ backup-name }}.
  1. В списке сервисов выберите **{{ ui-key.yacloud.iam.folder.dashboard.label_backup }}**.
  1. Если сервис {{ backup-name }} еще не активирован, нажмите **{{ ui-key.yacloud.backup.button_action-activate }}**.

      Если кнопки **{{ ui-key.yacloud.backup.button_action-activate }}** нет, и вам доступно создание ВМ с подключением к {{ backup-name }}, значит, сервис уже активирован. Переходите к следующему шагу.

{% endlist %}

## Создайте сервисный аккаунт {#create-sa}

{% list tabs group=instructions %}

- Консоль управления {#console}

  1. В [консоли управления]({{ link-console-main }}) выберите каталог, в котором активирован сервис.
  1. В верхней части экрана перейдите на вкладку **{{ ui-key.yacloud.iam.folder.switch_service-accounts }}**.
  1. Нажмите кнопку **{{ ui-key.yacloud.iam.folder.service-accounts.button_add }}**.
  1. Введите имя [сервисного аккаунта](../../iam/concepts/users/service-accounts.md) — `backup-sa`.
  1. Нажмите кнопку ![plus-sign](../../_assets/console-icons/plus.svg) **{{ ui-key.yacloud.iam.folder.service-account.label_add-role }}** и выберите [роль](../../backup/security/index.md#backup-editor) `backup.editor`.
  1. Нажмите кнопку **{{ ui-key.yacloud.iam.folder.service-account.popup-robot_button_add }}**.

- {{ yandex-cloud }} CLI {#cli}

  {% include [cli-install](../../_includes/cli-install.md) %}

  {% include [default-catalogue](../../_includes/default-catalogue.md) %}

  1. Создайте сервисный аккаунт с именем `backup-sa`:

     ```bash
     yc iam service-account create --name backup-sa
     ```

     Результат:

     ```text
     id: ajehb3tcdfa1********
     folder_id: b1g86q4m5vej********
     created_at: "2024-07-22T16:05:14.237381531Z"
     name: backup-sa
     ```

     Подробнее о команде `yc iam service-account create` см. в [справочнике CLI](../../cli/cli-ref/managed-services/iam/service-account/create.md).
  
  1. Назначьте сервисному аккаунту роль `backup.editor` на каталог:

     ```bash
     yc resource-manager folder add-access-binding <идентификатор_каталога> \
       --role backup.editor \
       --subject serviceAccount:<идентификатор_сервисного_аккаунта>
     ```

     Результат:

     ```text
     done (3s)
     effective_deltas:
       - action: ADD
         access_binding:
           role_id: backup.editor
           subject:
             id: ajehb3tcdfa1********
             type: serviceAccount
     ```

     Подробнее о команде `yc resource-manager folder add-access-binding` см. в [справочнике CLI](../../cli/cli-ref/managed-services/resource-manager/folder/add-access-binding.md).

- API {#api}

  Чтобы создать сервисный аккаунт, воспользуйтесь методом REST API [create](../../iam/api-ref/ServiceAccount/create.md) для ресурса [ServiceAccount](../../iam/api-ref/ServiceAccount/index.md) или вызовом gRPC API [ServiceAccountService/Create](../../iam/api-ref/grpc/service_account_service.md#Create).

  Чтобы назначить сервисному аккаунту роль `backup.editor` на каталог, воспользуйтесь методом [setAccessBindings](../../iam/api-ref/ServiceAccount/setAccessBindings.md) для ресурса [ServiceAccount](../../iam/api-ref/ServiceAccount/index.md) или вызовом gRPC API [ServiceAccountService/SetAccessBindings](../../iam/api-ref/grpc/service_account_service.md#SetAccessBindings).

{% endlist %}

## Создайте облачную сеть и подсети {#create-network}

Создайте [облачную сеть](../../vpc/concepts/network.md#network) с [подсетью](../../vpc/concepts/network.md#subnet) в той [зоне доступности](../../overview/concepts/geo-scope.md), где будет находиться виртуальная машина.

{% list tabs group=instructions %}

- Консоль управления {#console}

  1. В [консоли управления]({{ link-console-main }}) выберите каталог, в котором хотите создать облачную сеть.
  1. В списке сервисов выберите **{{ ui-key.yacloud.iam.folder.dashboard.label_vpc }}**.
  1. Справа сверху нажмите кнопку **{{ ui-key.yacloud.vpc.networks.button_create }}**.
  1. В поле **{{ ui-key.yacloud.vpc.networks.create.field_name }}** укажите `cloud-network`.
  1. В поле **{{ ui-key.yacloud.vpc.networks.create.field_advanced }}** выберите опцию **{{ ui-key.yacloud.vpc.networks.create.field_is-default }}**.
  1. Нажмите кнопку **{{ ui-key.yacloud.vpc.networks.button_create }}**.

- {{ yandex-cloud }} CLI {#cli}

  1. Создайте облачную сеть с именем `cloud-network`:
  
     ```bash
     yc vpc network create cloud-network
     ```

     Результат:

     ```text
     id: enptrcle5q3d********
     folder_id: b1g9hv2loamq********
     created_at: "2024-06-08T09:25:03Z"
     name: cloud-network
     default_security_group_id: enpbsnnop4ak********
     ```

     Подробнее о команде `yc vpc network create` см. в [справочнике CLI](../../cli/cli-ref/managed-services/vpc/network/create.md).

  1. Создайте подсеть `cloud-network-{{ region-id }}-d` в зоне доступности `{{ region-id }}-d`:
  
     ```bash
     yc vpc subnet create cloud-network-{{ region-id }}-d \
       --zone {{ region-id }}-d \
       --network-name cloud-network \
       --range 10.1.0.0/16
     ```

     Результат:

     ```text
     id: e9bnnssj8sc8********
     folder_id: b1g9hv2loamq********
     created_at: "2024-06-08T09:27:00Z"
     name: cloud-network-{{ region-id }}-d
     network_id: enptrcle5q3d********
     zone_id: {{ region-id }}-d
     v4_cidr_blocks:
     - 10.1.0.0/16
     ```

     Подробнее о команде `yc vpc subnet create` см. в [справочнике CLI](../../cli/cli-ref/managed-services/vpc/subnet/create.md).

- API {#api}

  1. Создайте сеть `cloud-network` с помощью метода REST API [create](../../vpc/api-ref/Network/create.md) для ресурса [Network](../../vpc/api-ref/Network/index.md) или вызова gRPC API [NetworkService/Create](../../vpc/api-ref/grpc/network_service.md#Create).
  1. Создайте подсеть `cloud-network-{{ region-id }}-d` с помощью метода REST API [create](../../vpc/api-ref/Subnet/create.md) для ресурса [Subnet](../../vpc/api-ref/Subnet/index.md) или вызова gRPC API [SubnetService/Create](../../vpc/api-ref/grpc/subnet_service.md#Create).

{% endlist %}

## Создайте и настройте группу безопасности {#create-sg}

Чтобы агент {{ backup-name }} мог обмениваться данными с серверами [провайдера резервного копирования](../../backup/concepts/index.md#providers), группа безопасности должна содержать правила, разрешающие сетевой доступ к IP-адресам ресурсов сервиса {{ backup-name }}.

{% list tabs group=instructions %}

- Консоль управления {#console}

  1. В [консоли управления]({{ link-console-main }}) перейдите в каталог, в котором вы хотите создать ВМ с подключением к {{ backup-name }}.
  1. В списке сервисов выберите **{{ ui-key.yacloud.iam.folder.dashboard.label_vpc }}**.
  1. На панели слева выберите ![image](../../_assets/console-icons/shield.svg) **{{ ui-key.yacloud.vpc.switch_security-groups }}**.
  1. Нажмите **{{ ui-key.yacloud.vpc.network.security-groups.button_create }}**.
  1. В поле **{{ ui-key.yacloud.vpc.network.security-groups.forms.field_sg-name }}** укажите `backup-sg`.
  1. В поле **{{ ui-key.yacloud.vpc.network.security-groups.forms.field_sg-network }}** выберите `cloud-network`.
  1. В блоке **{{ ui-key.yacloud.vpc.network.security-groups.forms.label_section-rules }}** перейдите на вкладку **{{ ui-key.yacloud.vpc.network.security-groups.label_egress }}** и нажмите **{{ ui-key.yacloud.vpc.network.security-groups.button_add-rule }}**.
  1. Последовательно добавьте следующие правила для исходящего трафика:

      {% include [outgoing-traffic](../../_includes/backup/outgoing-rules.md) %}

  1. Нажмите **{{ ui-key.yacloud.common.create }}**.

- {{ yandex-cloud }} CLI {#cli}

  Выполните следующую команду:

  ```bash
  yc vpc security-group create backup-sg \
    --network-name network-1 \
    --rule direction=egress,port=80,protocol=tcp,v4-cidrs=[213.180.193.0/24] \
    --rule direction=egress,port=80,protocol=tcp,v4-cidrs=[213.180.204.0/24] \
    --rule direction=egress,port=443,protocol=tcp,v4-cidrs=[84.47.172.0/24] \
    --rule direction=egress,port=443,protocol=tcp,v4-cidrs=[84.201.181.0/24] \
    --rule direction=egress,port=443,protocol=tcp,v4-cidrs=[178.176.128.0/24] \
    --rule direction=egress,port=443,protocol=tcp,v4-cidrs=[213.180.193.0/24] \
    --rule direction=egress,port=443,protocol=tcp,v4-cidrs=[213.180.204.0/24] \
    --rule direction=egress,from-port=7770,to-port=7800,protocol=tcp,v4-cidrs=[84.47.172.0/24] \
    --rule direction=egress,port=8443,protocol=tcp,v4-cidrs=[84.47.172.0/24] \
    --rule direction=egress,port=44445,protocol=tcp,v4-cidrs=[51.250.1.0/24] \
    --rule direction=ingress,port=22,protocol=tcp,v4-cidrs=[0.0.0.0/0]
  ```

  Результат:

  ```text
  id: enp0v73fe8fs********
  folder_id: b1g86q4m5vej********
  created_at: "2024-07-22T20:17:43Z"
  name: backup-sgg
  network_id: enp3srbi9u49********
  status: ACTIVE
  rules:
  - id: enpporsovuhj********
      direction: EGRESS
      ports:
        from_port: "80"
        to_port: "80"
      protocol_name: TCP
      protocol_number: "6"
      cidr_blocks:
        v4_cidr_blocks:
          - 213.180.193.0/24
  - id: enp7p6asol5i********
      direction: EGRESS
      ports:
        from_port: "80"
        to_port: "80"
      protocol_name: TCP
      protocol_number: "6"
      cidr_blocks:
        v4_cidr_blocks:
          - 213.180.204.0/24
  ...
  - id: enp36mip5nhe********
      direction: INGRESS
      ports:
        from_port: "22"
        to_port: "22"
      protocol_name: TCP
      protocol_number: "6"
      cidr_blocks:
        v4_cidr_blocks:
          - 0.0.0.0/0
  ```

  Подробнее о команде `yc vpc security-group create` см. в [справочнике CLI](../../cli/cli-ref/managed-services/vpc/security-group/create.md).

- API {#api}

  Чтобы создать группу безопасности, воспользуйтесь методом REST API [create](../../vpc/api-ref/SecurityGroup/create.md) для ресурса [SecurityGroup](../../vpc/api-ref/SecurityGroup/index.md) или вызовом gRPC API [SecurityGroupService/Create](../../vpc/api-ref/grpc/security_group_service.md#Create).

{% endlist %}

## Создайте политику резервного копирования {#create-policy}

{% list tabs group=instructions %}

- Консоль управления {#console}

  1. В [консоли управления]({{ link-console-main }}) выберите [каталог](../../resource-manager/concepts/resources-hierarchy.md#folder), в котором нужно создать политику резервного копирования.
  1. В списке сервисов выберите **{{ ui-key.yacloud.iam.folder.dashboard.label_backup }}**.
  1. Перейдите на вкладку ![policies](../../_assets/console-icons/calendar.svg) **{{ ui-key.yacloud.backup.label_policies }}**.
  1. Нажмите кнопку **{{ ui-key.yacloud.backup.button_create-policy }}**.
  1. Укажите параметры политики:

     * **{{ ui-key.yacloud.common.name }}** — `daily-backup`.
     * **{{ ui-key.yacloud.backup.field_repeat-period-type }}** — `{{ ui-key.yacloud.backup.value_period-time-weekly }}`.
     * **{{ ui-key.yacloud.backup.field_days-of-week }}** — `{{ ui-key.yacloud.backup.value_day-fri }}`.
     * **{{ ui-key.yacloud.backup.field_time }}** — `03:00`.
     * **{{ ui-key.yacloud.backup.field_backup-scheme }}** — `{{ ui-key.yacloud.backup.value_type-incremental }}`.
     * **{{ ui-key.yacloud.backup.field_auto-delete }}** — `{{ ui-key.yacloud.backup.value_retention-save-all }}`.

  1. Нажмите кнопку **{{ ui-key.yacloud.common.save }}**.

- CLI {#cli}

  1. Опишите конфигурацию создаваемой политики резервного копирования в файле `backup-policy-schema.json`.

     ```json
     {
       "compression": "NORMAL",
       "format": "AUTO",
       "multiVolumeSnapshottingEnabled": true,
       "preserveFileSecuritySettings": true,
       "reattempts": {
         "enabled": true,
         "interval": {
           "type": "SECONDS",
           "count": "30"
         },
         "maxAttempts": "30"
       },
       "silentModeEnabled": true,
       "splitting": {
         "size": "1099511627776"
       },
       "vmSnapshotReattempts": {
         "enabled": true,
         "interval": {
           "type": "MINUTES",
           "count": "5"
         },
         "maxAttempts": "3"
       },
       "vss": {
         "enabled": true,
         "provider": "TARGET_SYSTEM_DEFINED"
       },
       "archive": {
         "name": "'[Machine Name]-[Plan ID]-[Unique ID]A'"
       },
       "performanceWindow": {
         "enabled": true
       },
       "scheduling": {
         "backupSets": [
           {
             "time": {
               "weekdays": [
                 "FRIDAY"
               ],
               "repeatAt": [
                 {
                   "hour": "3"
                 }
               ],
               "type": "WEEKLY"
             }
           }
         ],
         "enabled": true,
         "maxParallelBackups": "2",
         "randMaxDelay": {
           "type": "MINUTES",
           "count": "30"
         },
         "scheme": "ALWAYS_INCREMENTAL",
         "weeklyBackupDay": "MONDAY"
       },
       "cbt": "ENABLE_AND_USE",
       "fastBackupEnabled": true,
       "quiesceSnapshottingEnabled": true
     }
     ```

  1. Создайте политику резервного копирования:

     ```bash
     yc backup policy create \
       --name daily-backup \
       --settings-from-file ./backup-policy-scheme.json
     ```

     Результат:

     ```text
     id: cdgo5vytuw57********
     name: daily-backup
     created_at: "2024-07-23T20:34:37Z"
     updated_at: "2024-07-23T20:34:37Z"
     enabled: true
     settings:
       compression: NORMAL
       format: AUTO
       multi_volume_snapshotting_enabled: true
       preserve_file_security_settings: true
       reattempts:
         enabled: true
         interval:
           type: SECONDS
           count: "30"
         max_attempts: "30"
       silent_mode_enabled: true
       splitting:
         size: "1099511627776"
       vm_snapshot_reattempts:
         enabled: true
         interval:
           type: MINUTES
           count: "5"
         max_attempts: "3"
       vss:
         enabled: true
         provider: TARGET_SYSTEM_DEFINED
       archive:
         name: '''[Machine Name]-[Plan ID]-[Unique ID]A'''
       performance_window:
         enabled: true
       retention: {}
       scheduling:
         backup_sets:
           - time:
               weekdays:
                 - FRIDAY
               repeat_at:
                 - hour: "3"
               type: WEEKLY
             type: TYPE_AUTO
         enabled: true
         max_parallel_backups: "2"
         rand_max_delay:
           type: MINUTES
           count: "30"
         scheme: ALWAYS_INCREMENTAL
         weekly_backup_day: MONDAY
       cbt: ENABLE_AND_USE
       fast_backup_enabled: true
       quiesce_snapshotting_enabled: true
     folder_id: b1g86q4m5vej********
     ```

     Подробнее о команде `yc backup policy create` см. в [справочнике CLI](../../cli/cli-ref/managed-services/backup/policy/create.md).

- API {#api}

  Чтобы создать [политику резервного копирования](../../backup/concepts/policy.md), воспользуйтесь методом REST API [create](../../backup/backup/api-ref/Policy/create.md) для ресурса [Policy](../../backup/backup/api-ref/Policy/index.md) или вызовом gRPC API [PolicyService/Create](../../backup/backup/api-ref/grpc/policy_service.md#Create).

{% endlist %}

## Создайте ВМ {#create-vm}

{% list tabs group=instructions %}

- Консоль управления {#console}

  1. В [консоли управления]({{ link-console-main }}) выберите каталог, в котором нужно создать ВМ.
  1. В списке сервисов выберите **{{ ui-key.yacloud.iam.folder.dashboard.label_compute }}**.
  1. На панели слева выберите ![image](../../_assets/console-icons/server.svg) **{{ ui-key.yacloud.compute.switch_instances }}** и нажмите кнопку **{{ ui-key.yacloud.compute.instances.button_create }}**.
  1. Введите имя ВМ: `backup-instance`.
  1. Выберите [зону доступности](../../overview/concepts/geo-scope.md) `{{ region-id }}-d`.
  1. В блоке **{{ ui-key.yacloud.compute.instances.create.section_image }}** на вкладке **{{ ui-key.yacloud.compute.instances.create.image_value_os-products }}** выберите операционную систему `Ubuntu 22.04`.
  1. В блоке **{{ ui-key.yacloud.compute.instances.create.section_network }}**:

     1. Выберите подсеть `cloud-network-{{ region-id }}-d`.
     1. В поле **{{ ui-key.yacloud.component.compute.network-select.field_external }}** выберите `{{ ui-key.yacloud.component.compute.network-select.switch_auto }}`.
     1. Выберите группу безопасности `backup-sg`.

  1. В блоке **{{ ui-key.yacloud.compute.instances.create.section_access }}**:

     1. Выберите сервисный аккаунт `backup-sa`.
     1. В поле **{{ ui-key.yacloud.compute.instances.create.field_user }}** введите имя пользователя: `vm-user`.
     1. В поле **{{ ui-key.yacloud.compute.instances.create.field_key }}** вставьте содержимое файла [открытого ключа](../../compute/operations/vm-connect/ssh.md#creating-ssh-keys). Пару ключей для подключения по [SSH](../../glossary/ssh-keygen.md) необходимо [создать](../../compute/operations/vm-connect/ssh.md#creating-ssh-keys) самостоятельно.

  1. В блоке **{{ ui-key.yacloud.compute.instances.create.section_additional }}** включите опцию **{{ ui-key.yacloud.compute.instances.create.section_backup }}**.
  1. В блоке **{{ ui-key.yacloud.common.metadata }}** добавьте поле с ключом `cloudbackup` и значением `{"initialPolicies": ["<идентификатор_политики_daily-backup>"]}`.
  1. Нажмите кнопку **{{ ui-key.yacloud.compute.instances.create.button_create }}**.

- {{ yandex-cloud }} CLI {#cli}
  
  1. Опишите конфигурацию пользовательских метаданных в файле `user-data.yaml`:

     ```yaml
     #cloud-config
     datasource:
      Ec2:
       strict_id: false
     ssh_pwauth: no
     users:
     - name: vm-user
       sudo: ALL=(ALL) NOPASSWD:ALL
       shell: /bin/bash
       ssh_authorized_keys:
       - <публичный_SSH-ключ>
     packages:
       - curl
       - perl
       - jq
     runcmd:
       - curl https://storage.yandexcloud.net/backup-distributions/agent_installer.sh | sudo bash
     ```

  1. Сохраните идентификатор политики `daily-backup` в файле `cloudbackup.json`:

     ```json
     {"initialPolicies": ["<идентификатор_политики_daily-backup>"]}
     ```

  1. Выполните команду:

     ```bash
     yc compute instance create \
       --name backup-instance \
       --zone {{ region-id }}-d \
       --network-interface subnet-name=cloud-network-{{ region-id }}-d,security-group-ids=<идентификатор_группы_безопасности_backup-sg>,ipv4-address=auto,nat-ip-version=ipv4 \
       --create-boot-disk image-folder-id=standard-images,image-family=ubuntu-2204-lts,size=15 \
       --metadata-from-file user-data=./user-data.yaml,cloudbackup=./cloudbackup.json \
       --service-account-name backup-sa
     ```

     Подробнее о команде `yc compute instance create` см. в [справочнике CLI](../../cli/cli-ref/managed-services/compute/instance/create.md).

- API {#api}

  Чтобы создать ВМ, воспользуйтесь методом REST API [create](../../compute/api-ref/Instance/create.md) для ресурса [Instance](../../compute/api-ref/Instance/index.md) или вызовом gRPC API [InstanceService/Create](../../compute/api-ref/grpc/instance_service.md#Create).
  
  В теле запроса передайте:

  * в поле `metadata` объект `user-data`, содержащий конфигурацию пользовательских метаданных со скриптом для установки агента резервного копирования;
  * в поле `cloudbackup` идентификатор политики резервного копирования.
  
  Используйте `\n` в качестве разделителя строк:

  {% cut "Пример тела запроса" %}
  
  ```json
  {
    "folderId": "идентификатор_каталога",
    "name": "backup-instance",
    "zoneId": "ru-central1-d",
    "platformId": "standard-v3",
    "resourcesSpec": {
      "memory": "2147483648",
      "cores": "2"
    },
    "metadata": {
      "user-data": "#cloud-config\ndatasource:\nEc2:\n  strict_id: false\nssh_pwauth: no\nusers:\n- name: vm-user\n  shell: /bin/bash\n  sudo: ALL=(ALL) NOPASSWD:ALL\n  ssh-authorized-keys:\n  - <публичный_SSH-ключ>\npackages:\n  - curl\n  - perl\n  - jq\nruncmd:\n  - curl https://storage.yandexcloud.net/backup-distributions/agent_installer.sh | sudo bash",
      "cloudbackup": "{\"initialPolicies\": [\"идентификатор_политики\"]}"
    },
    "bootDiskSpec": {
      "diskSpec": {
        "size": "16106127360",
        "imageId": "fd8ljvsrm3l1q2tgqji9"
      }
    },
    "networkInterfaceSpecs": [
      {
        "subnetId": "идентификатор_подсети",
        "primaryV4AddressSpec": {
          "oneToOneNatSpec": {
            "ipVersion": "IPV4"
          }
        },
        "securityGroupIds": [
          "идентификатор_группы_безопасности"
        ]
      }
    ],
    "serviceAccountId": "идентификатор_сервисного_аккаунта"
  }
  ```

  {% endcut %}
  
{% endlist %}

{% note info %}

{% include [agent-installation-timespan](../../_includes/backup/agent-installation-timespan.md) %}

Привязка политики выполняется асинхронно после создания и инициализации ВМ, а также установки и настройки агента резервного копирования. Это может занимать до 10–15 минут. В итоге виртуальная машина появится в списке ВМ сервиса {{ backup-name }} и в списке ВМ, привязанных к политике `daily-backup`.

{% endnote %}

За процессом установки можно следить с помощью [последовательного порта](../../compute/operations/vm-info/get-serial-port-output.md) ВМ в консоли управления.

## Как удалить созданные ресурсы {#clear-out}

Чтобы перестать платить за созданные ресурсы:
1. [Удалите](../../backup/operations/delete-vm.md) ВМ из {{ backup-name }}.
1. [Удалите](../../compute/operations/vm-control/vm-delete.md) ВМ из {{ compute-name }}.
1. Если для ВМ были созданы резервные копии, [удалите](../../backup/operations/backup-vm/delete.md) их. 
