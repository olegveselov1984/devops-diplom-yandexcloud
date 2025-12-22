# Дипломный практикум в Yandex.Cloud
---
## Цели:

1. Подготовить облачную инфраструктуру на базе облачного провайдера Яндекс.Облако.
2. Запустить и сконфигурировать Kubernetes кластер.
3. Установить и настроить систему мониторинга.
4. Настроить и автоматизировать сборку тестового приложения с использованием Docker-контейнеров.
5. Настроить CI для автоматической сборки и тестирования.
6. Настроить CD для автоматического развёртывания приложения.

---
## Этапы выполнения:


### Создание облачной инфраструктуры

Для начала необходимо подготовить облачную инфраструктуру в ЯО при помощи [Terraform](https://www.terraform.io/).

<img width="1487" height="1030" alt="image" src="https://github.com/user-attachments/assets/82189a64-18fa-45e0-843c-e169ba51e95d" />

Особенности выполнения:

Предварительная подготовка к установке и запуску Kubernetes кластера.

1. Создайте сервисный аккаунт, который будет в дальнейшем использоваться Terraform для работы с инфраструктурой с необходимыми и достаточными правами. Не стоит использовать права суперпользователя (Создал terraformeditor)

<img width="1002" height="363" alt="image" src="https://github.com/user-attachments/assets/216a54da-53d9-476b-8357-21ee587a2881" />

2. Подготовьте [backend](https://developer.hashicorp.com/terraform/language/backend) для Terraform:  
   а. Рекомендуемый вариант: S3 bucket в созданном ЯО аккаунте(создание бакета через TF)  

3. Создайте конфигурацию Terrafrom, используя созданный бакет ранее как бекенд для хранения стейт файла. Конфигурации Terraform для создания сервисного аккаунта и бакета и основной инфраструктуры следует сохранить в разных папках.

<img width="1146" height="466" alt="image" src="https://github.com/user-attachments/assets/239e0854-391c-4680-8f5d-f4c41a3a5502" />

4. Создайте VPC с подсетями в разных зонах доступности.

<img width="2095" height="827" alt="image" src="https://github.com/user-attachments/assets/40c488ed-68a8-4fba-a4a6-2f201b971db4" />

5. Убедитесь, что теперь вы можете выполнить команды `terraform destroy` и `terraform apply` без дополнительных ручных действий.
```
/devops-diplom-yandexcloud/terraform$ terraform init
Initializing the backend...
Initializing modules...
Initializing provider plugins...
- Reusing previous version of yandex-cloud/yandex from the dependency lock file
- Reusing previous version of hashicorp/template from the dependency lock file
- Using previously-installed yandex-cloud/yandex v0.177.0
- Using previously-installed hashicorp/template v2.2.0

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```

<img width="1143" height="580" alt="image" src="https://github.com/user-attachments/assets/5d5a2fda-4e14-47ca-a49f-b5d095b20677" />


Ожидаемые результаты:

1. Terraform сконфигурирован и создание инфраструктуры посредством Terraform возможно без дополнительных ручных действий, стейт основной конфигурации сохраняется в бакете или Terraform Cloud
2. Полученная конфигурация инфраструктуры является предварительной, поэтому в ходе дальнейшего выполнения задания возможны изменения.

---
### Создание Kubernetes кластера

На этом этапе необходимо создать [Kubernetes](https://kubernetes.io/ru/docs/concepts/overview/what-is-kubernetes/) кластер на базе предварительно созданной инфраструктуры.   Требуется обеспечить доступ к ресурсам из Интернета.

Это можно сделать двумя способами (Выбрал второй вариант):

1. Рекомендуемый вариант: самостоятельная установка Kubernetes кластера.  
   а. При помощи Terraform подготовить как минимум 3 виртуальных машины Compute Cloud для создания Kubernetes-кластера. Тип виртуальной машины следует выбрать самостоятельно с учётом требовании к производительности и стоимости. Если в дальнейшем поймете, что необходимо сменить тип инстанса, используйте Terraform для внесения изменений.  
   б. Подготовить [ansible](https://www.ansible.com/) конфигурации, можно воспользоваться, например [Kubespray](https://kubernetes.io/docs/setup/production-environment/tools/kubespray/)  
   в. Задеплоить Kubernetes на подготовленные ранее инстансы, в случае нехватки каких-либо ресурсов вы всегда можете создать их при помощи Terraform.
2. Альтернативный вариант: воспользуйтесь сервисом [Yandex Managed Service for Kubernetes](https://cloud.yandex.ru/services/managed-kubernetes)  
  а. С помощью terraform resource для [kubernetes](https://registry.terraform.io/providers/yandex-cloud/yandex/latest/docs/resources/kubernetes_cluster) создать **региональный** мастер kubernetes с размещением нод в разных 3 подсетях      
  б. С помощью terraform resource для [kubernetes node group](https://registry.terraform.io/providers/yandex-cloud/yandex/latest/docs/resources/kubernetes_node_group)
  
Ожидаемый результат:

1. Работоспособный Kubernetes кластер.
2. В файле `~/.kube/config` находятся данные для доступа к кластеру.
3. Команда `kubectl get pods --all-namespaces` отрабатывает без ошибок.

<img width="970" height="552" alt="image" src="https://github.com/user-attachments/assets/50719172-7cdb-4b29-832d-bc6807f6d42d" />


---
### Создание тестового приложения

Для перехода к следующему этапу необходимо подготовить тестовое приложение, эмулирующее основное приложение разрабатываемое вашей компанией.

Способ подготовки:

1. Рекомендуемый вариант:  
   а. Создайте отдельный git репозиторий с простым nginx конфигом, который будет отдавать статические данные.
   https://github.com/olegveselov1984/devops-diplom-yandexcloud-nginx-.git

<img width="825" height="409" alt="image" src="https://github.com/user-attachments/assets/95e6cbd4-3ad6-43f6-9ed7-765caf9bbf51" />


```
events {
  worker_connections  1024;
}

http {

 server {
  listen   80;
   server_name  localhost;

    location / {
      root   /var/www/html;
      index  index.html index.htm;
    }

    charset koi8-r;

    error_page   500 502 503 504  /50x.html;
      location = /50x.html {
      root   /var/www/html;
    }
  }
}
```

   б. Подготовьте Dockerfile для создания образа приложения.  
   
```
FROM nginx:1.21.6-alpine
# Configuration
ADD nginx.conf /etc/nginx.conf
# Content
COPY index.html /usr/share/nginx/html
```

Проверяем локально:

```
/devops-diplom-yandexcloud/devops-diplom-yandexcloud-nginx-$ docker build -t olegveselov1984/diplom:0.0.1 .
[+] Building 1.7s (9/9) FINISHED                                                                                                                docker:default
 => [internal] load build definition from Dockerfile                                                                                                      0.0s
 => => transferring dockerfile: 159B                                                                                                                      0.0s
 => [internal] load metadata for docker.io/library/nginx:1.21.6-alpine                                                                                    1.5s
 => [auth] library/nginx:pull token for registry-1.docker.io                                                                                              0.0s
 => [internal] load .dockerignore                                                                                                                         0.0s
 => => transferring context: 2B                                                                                                                           0.0s
 => [1/3] FROM docker.io/library/nginx:1.21.6-alpine@sha256:a74534e76ee1121d418fa7394ca930eb67440deda413848bc67c68138535b989                              0.0s
 => [internal] load build context                                                                                                                         0.0s
 => => transferring context: 61B                                                                                                                          0.0s
 => CACHED [2/3] ADD nginx.conf /etc/nginx.conf                                                                                                           0.0s
 => CACHED [3/3] COPY index.html /usr/share/nginx/html                                                                                                    0.0s
 => exporting to image                                                                                                                                    0.0s
 => => exporting layers                                                                                                                                   0.0s
 => => writing image sha256:9200a171013d19b0c55d7a6cb6575a366489981916ecebd3f342c1dac41a99c0                                                              0.0s
 => => naming to docker.io/olegveselov1984/diplom:0.0.1                                                                                                   0.0s
ubuntu@ubuntu:~/src/devops-diplom-yandexcloud
/devops-diplom-yandexcloud/devops-diplom-yandexcloud-nginx-$ docker run --network host olegveselov1984/diplom:0.0.1
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2025/12/22 11:30:57 [notice] 1#1: using the "epoll" event method
2025/12/22 11:30:57 [notice] 1#1: nginx/1.21.6
2025/12/22 11:30:57 [notice] 1#1: built by gcc 10.3.1 20211027 (Alpine 10.3.1_git20211027) 
2025/12/22 11:30:57 [notice] 1#1: OS: Linux 6.8.0-90-generic
2025/12/22 11:30:57 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2025/12/22 11:30:57 [notice] 1#1: start worker processes
2025/12/22 11:30:57 [notice] 1#1: start worker process 32
2025/12/22 11:30:57 [notice] 1#1: start worker process 33
2025/12/22 11:30:57 [notice] 1#1: start worker process 34
2025/12/22 11:30:57 [notice] 1#1: start worker process 35
127.0.0.1 - - [22/Dec/2025:11:31:05 +0000] "GET / HTTP/1.1" 200 45 "-" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:146.0) Gecko/20100101 Firefox/146.0" "-"
```


```
/devops-diplom-yandexcloud$ docker image ls
REPOSITORY                              TAG          IMAGE ID       CREATED          SIZE
olegveselov1984/diplom                  0.0.1        9200a171013d   13 minutes ago   23.4MB
```


<img width="480" height="162" alt="image" src="https://github.com/user-attachments/assets/75efa123-c6f5-4513-9de4-b9d9303ef3bb" />


Отправляем docker image в DockerHub:
Логинемся
```
/devops-diplom-yandexcloud$ docker login

USING WEB-BASED LOGIN

i Info → To sign in with credentials on the command line, use 'docker login -u <username>'
         

Your one-time device confirmation code is: TVKS-QRVR
Press ENTER to open your browser or submit your device code here: https://login.docker.com/activate

Waiting for authentication in the browser…


WARNING! Your credentials are stored unencrypted in '/home/ubuntu/.docker/config.json'.
Configure a credential helper to remove this warning. See
https://docs.docker.com/go/credential-store/

Login Succeeded
```

Отправляем:
```
/devops-diplom-yandexcloud$ docker push olegveselov1984/diplom:0.0.1
The push refers to repository [docker.io/olegveselov1984/diplom]
970b60ca3d01: Pushed 
0d65d2f3bf38: Pushed 
c0e7c94aefd8: Pushed 
d6dd885da0bb: Pushed 
a43749efe4ec: Pushed 
45b275e8a06d: Pushed 
4721bfafc708: Pushed 
4fc242d58285: Pushed 
0.0.1: digest: sha256:9a0b8513f9e12062962bc689855fb3416999d66566e1a985ac3e5889bd73ae92 size: 1982
```


Ожидаемый результат:

1. Git репозиторий с тестовым приложением и Dockerfile.

<img width="711" height="406" alt="image" src="https://github.com/user-attachments/assets/bbf8128d-1775-4ed7-8de2-7ad4ccc1468a" />


2. Регистри с собранным docker image. В качестве регистри может быть DockerHub или [Yandex Container Registry](https://cloud.yandex.ru/services/container-registry), созданный также с помощью terraform.

---
### Подготовка cистемы мониторинга и деплой приложения

Уже должны быть готовы конфигурации для автоматического создания облачной инфраструктуры и поднятия Kubernetes кластера.  
Теперь необходимо подготовить конфигурационные файлы для настройки нашего Kubernetes кластера.

Цель:
1. Задеплоить в кластер [prometheus](https://prometheus.io/), [grafana](https://grafana.com/), [alertmanager](https://github.com/prometheus/alertmanager), [экспортер](https://github.com/prometheus/node_exporter) основных метрик Kubernetes.
2. Задеплоить тестовое приложение, например, [nginx](https://www.nginx.com/) сервер отдающий статическую страницу.

Способ выполнения:
1. Воспользоваться пакетом [kube-prometheus](https://github.com/prometheus-operator/kube-prometheus), который уже включает в себя [Kubernetes оператор](https://operatorhub.io/) для [grafana](https://grafana.com/), [prometheus](https://prometheus.io/), [alertmanager](https://github.com/prometheus/alertmanager) и [node_exporter](https://github.com/prometheus/node_exporter). Альтернативный вариант - использовать набор helm чартов от [bitnami](https://github.com/bitnami/charts/tree/main/bitnami).

Буду использовать kube-prometheus

Потребуется установить пакет поддержки языка Golang:
```
sudo snap install go --classic
```
```
/devops-diplom-yandexcloud/prometheus$ go version
go version go1.25.5 linux/amd64
```

Для удобства путь к исполняемым файлам Golang следует указать в PATH:
```
export PATH=$PATH:$HOME/go/bin
```
Далее следует установить инструмент jsonnet-bundler:
```
go install -a github.com/jsonnet-bundler/jsonnet-bundler/cmd/jb@latest
```

Далее переходим в папку prometheus, инициализуем в ней "jsonnet-bundler" и устанавливаем необходимые для "kube-prometheus" зависимости:
```
jb init
jb install github.com/prometheus-operator/kube-prometheus/jsonnet/kube-prometheus@main
```
Далее скачиваем файл example.jsonnet используемый в качестве образца файла конфигурирования:
```
 wget https://raw.githubusercontent.com/prometheus-operator/kube-prometheus/main/example.jsonnet -O example.jsonnet
```
А также скачиваем скрипт, используемый для сборки:
```
wget https://raw.githubusercontent.com/prometheus-operator/kube-prometheus/main/build.sh -O build.sh
```
Полученный файл build.sh помечаем как исполняемый:
```
chmod u+x build.sh
```
Далее обновляем зависимости для "kube-prometheus":
```
jb update   
```
Перед компиляцией следует дополнительно установить инструменты "gojsontoyaml" и "jsonnet":
```
go install github.com/brancz/gojsontoyaml@latest
go install github.com/google/go-jsonnet/cmd/jsonnet@latest

```

Создаём копию файла example.jsonnet под именем monitoring.jsonnet и указываем в нем настройки инструментов мониторинга, установка которых предполагается в кластер Kubenetes. После этого запускаем скрипт генерации манифестов:
```
./build.sh monitoring.jsonnet
```
В результате в папке manifests получаем набор манифестов для разворачивания системы мониторинга. Для их применения используется утилита kubectl. Её вызов производится в два этапа.



### Деплой инфраструктуры в terraform pipeline

1. Если на первом этапе вы не воспользовались [Terraform Cloud](https://app.terraform.io/), то задеплойте и настройте в кластере [atlantis](https://www.runatlantis.io/) для отслеживания изменений инфраструктуры. Альтернативный вариант 3 задания: вместо Terraform Cloud или atlantis настройте на автоматический запуск и применение конфигурации terraform из вашего git-репозитория в выбранной вами CI-CD системе при любом комите в main ветку. Предоставьте скриншоты работы пайплайна из CI/CD системы.

Ожидаемый результат:
1. Git репозиторий с конфигурационными файлами для настройки Kubernetes.
2. Http доступ на 80 порту к web интерфейсу grafana.
3. Дашборды в grafana отображающие состояние Kubernetes кластера.
4. Http доступ на 80 порту к тестовому приложению.
5. Atlantis или terraform cloud или ci/cd-terraform
---
### Установка и настройка CI/CD

Осталось настроить ci/cd систему для автоматической сборки docker image и деплоя приложения при изменении кода.

Цель:

1. Автоматическая сборка docker образа при коммите в репозиторий с тестовым приложением.
2. Автоматический деплой нового docker образа.

Можно использовать [teamcity](https://www.jetbrains.com/ru-ru/teamcity/), [jenkins](https://www.jenkins.io/), [GitLab CI](https://about.gitlab.com/stages-devops-lifecycle/continuous-integration/) или GitHub Actions.

Ожидаемый результат:

1. Интерфейс ci/cd сервиса доступен по http.
2. При любом коммите в репозиторие с тестовым приложением происходит сборка и отправка в регистр Docker образа.
3. При создании тега (например, v1.0.0) происходит сборка и отправка с соответствующим label в регистри, а также деплой соответствующего Docker образа в кластер Kubernetes.

---
## Что необходимо для сдачи задания?

1. Репозиторий с конфигурационными файлами Terraform и готовность продемонстрировать создание всех ресурсов с нуля.
2. Пример pull request с комментариями созданными atlantis'ом или снимки экрана из Terraform Cloud или вашего CI-CD-terraform pipeline.
3. Репозиторий с конфигурацией ansible, если был выбран способ создания Kubernetes кластера при помощи ansible.
4. Репозиторий с Dockerfile тестового приложения и ссылка на собранный docker image.
5. Репозиторий с конфигурацией Kubernetes кластера.
6. Ссылка на тестовое приложение и веб интерфейс Grafana с данными доступа.
7. Все репозитории рекомендуется хранить на одном ресурсе (github, gitlab)

