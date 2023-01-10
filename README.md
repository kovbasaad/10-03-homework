# Домашнее задание к занятию "10.3 Pacemaker" - Ковбаса Анна


### Задание 1

*Опишите основные функции и назначение Pacemaker.*

- обнаружение и восстановление сбоев на уровне узлов и сервисов;
- возможность гарантировать целостность данных путем ограждения неисправных узлов;
- поддержка одного или нескольких узлов на кластер;
- поддержка нескольких стандартов интерфейса ресурсов (все, что может быть написано сценарием, может быть кластеризовано);
- независимость от подсистемы хранения — общий диск не требуется;
- поддержка STONITH (Shoot-The-Other-Node-In-The-Head);
- поддержка и кворумных и ресурсозависимых кластеров;
- автоматически реплицируемая конфигурация, которую можно обновлять с любого узла;
- возможность задания порядка запуска ресурсов, а также их совместимости на одном узле;
- поддерживает расширенные типы ресурсов: клоны (когда ресурс запущен на множестве узлов) и дополнительные состояния (master/slave и подобное);
- единые инструменты управления кластером с поддержкой сценариев.

---

### Задание 2

*Опишите основные функции и назначение Corosync.*

- отслеживание статуса приложений;
- оповещение приложений о смене активной ноды кластера;
- отправка одинаковых сообщений процессам на всех узлах кластера;
- предоставление доступа к базе данных с конфигурацией и статистикой, а также отправка уведомлений о ее изменениях.


---

### Задание 3

*Соберите модель, состоящую из двух виртуальных машин. Установите pacemaker, corosync, pcs. Настройте HA кластер.*


```
totem {
    version: 2
    cluster_name: hw-cluster
    transport: knet
    crypto_cipher: aes256
    crypto_hash: sha256
}

nodelist {
    node {
        ring0_addr: deb10
        name: deb10
        nodeid: 1
    }

    node {
        ring0_addr: deb10-1
        name: deb10-1
        nodeid: 2
    }
}

quorum {
    provider: corosync_votequorum
    two_node: 1
}

logging {
    to_logfile: yes
    logfile: /var/log/corosync/corosync.log
    to_syslog: yes
    timestamp: on
}
```


![3-1](https://github.com/kovbasaad/10-03-homework/blob/main/img/10.jpeg)
![3-2](https://github.com/kovbasaad/10-03-homework/blob/main/img/10-1.JPG)

### Задание 4*

*Установите и настройте DRBD сервис для настроенного кластера.*

```
resource www {
    protocol C;

    disk {
        fencing resource-only;
    }

    handlers {
        fence-peer "/usr/lib/drbd/crm-fence-peer.sh";
        after-resync-target "/usr/lib/drbd/crm-unfence-peer.sh";
    }
syncer {
        rate 110M;
    }
    on deb10
    {
        device /dev/drbd2;
        disk /dev/vg0/www;
        address 192.168.1.1:7794;
        meta-disk internal;
    }
    on deb10-1
    {
        device /dev/drbd2;
        disk /dev/vg0/www;
        address 192.168.1.2:7794;
        meta-disk internal;
    }
}
```

```
resource mysql {
    protocol C;

    disk {
        fencing resource-only;
    }

    handlers {
        fence-peer "/usr/lib/drbd/crm-fence-peer.sh";
        after-resync-target "/usr/lib/drbd/crm-unfence-peer.sh";
    }
syncer {
        rate 110M;
    }
    on deb10
    {
        device /dev/drbd3;
        disk /dev/vg0/mysql;
        address 192.168.1.1:7795;
        meta-disk internal;
    }
    on deb10-1
    {
        device /dev/drbd3;
        disk /dev/vg0/mysql;
        address 192.168.1.2:7795;
        meta-disk internal;
    }
}

```
