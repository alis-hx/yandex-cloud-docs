# Виртуальная машина выключается операцией `Instance guest stop processing`


## Описание проблемы {#issue-description}

В списке операций виртуальной машины появилась операция от имени пользователя `yc.compute.tf-compute-node-sa` с описанием `Instance guest stop processing`.

## Решение {#issue-resolution}

Эта операция сообщает о том, что виртуальная машина была выключена из гостевой операционной системы. Необходимо проверить системный журнал на наличие событий о завершении работы.