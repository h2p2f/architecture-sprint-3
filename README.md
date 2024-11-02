# Sprint 3
## Анализ и планирование

### Функциональность исходного монолита

Пользователи API могут:
 - удаленно включать/выключать отопление 
 - устанавливать желаемую температуру 
 - просматривать текущую температуру
 - автоматически управлять отоплением в зависимости от текущей температуры и желаемой температуры (coming soon, see developer's comment: // TODO: Implement automatic temperature maintenance logic in the service layer )

### Архитектура исходного монолита

 - Язык программирования: Java
 - База данных: PostgreSQL 
 - Архитектура: Monolith 
 - Взаимодействие с пользователем: HTTP, запросы обрабатываются синхронно
 - Взаимодействие с VendorAPI: отсутствует документация
 - Масштабируемость: вертикальная 
 - Развертывание: виртуальная машина, остановка для обслуживания

### Домены и границы контекстов

 - Домен: Управление устройствами
   - контекст: Управление отоплением - включение и выключение отопления
   - контекст: Управление температурой - установка желаемой температуры
 - Домен: Мониторинг температуры
   - контекст: Обработка телеметрии с датчика температуры - получение текущей температуры

### Анализ проблем монолита
 - Ограниченный функционал (нет возможности добавить новые устройства через API, нет автоматического поддержания температуры)
 - Низкие масштабируемость, отказоустойчивость и скорость разработки
 - Зависимость от VendorAPI так как отсутствует собственное производство устройств

### Диаграмма контекста

![Диаграмма контекста](./images/monolith-context.png)

файл: [monolith-context.puml](./schemas/monolith-context.puml)

## Проектирование микросервисной архитектуры

### Разделение на микросервисы

 - Управление устройствами
 - Управление пользователями и домами
 - Управление вызовами внешних API
 - Мониторинг устройств

### Диаграмма контейнеров
Файл в формате documentation-as-code: [microservices-containers.puml](./schemas/microservices-containers.puml)
(при рендеринге выглядит отвратительно, но в PlantUML работает)

Более щадащая глаза схема (подготовлена в draw.io):

![Диаграмма контейнеров](./images/micro-containers.png)

### Диаграмма компонентов

Компонент управления устройствами

![Диаграмма компонентов](./images/micro-components-devices.png)

Компонент телеметрии

![Диаграмма компонентов](./images/micro-components-telemetry.png)

### Диаграмма кода

![Диаграмма кода](./images/micro-code.png)

ER-диаграмма данных

![ER-диаграмма](./images/er.png)



# Базовая настройка

## Запуск minikube

[Инструкция по установке](https://minikube.sigs.k8s.io/docs/start/)

```bash
minikube start
```

## Добавление токена авторизации GitHub

[Получение токена](https://github.com/settings/tokens/new)

```bash
kubectl create secret docker-registry ghcr --docker-server=https://ghcr.io --docker-username=<github_username> --docker-password=<github_token> -n default
```

## Установка API GW kusk

[Install Kusk CLI](https://docs.kusk.io/getting-started/install-kusk-cli)

```bash
kusk cluster install
```

## Смена адреса образа в helm chart

После того как вы сделали форк репозитория и у вас в репозитории отработал GitHub Action. Вам нужно получить адрес образа <https://github.com/><github_username>/architecture-sprint-3/pkgs/container/architecture-sprint-3

Он выглядит таким образом
```ghcr.io/<github_username>/architecture-sprint-3:latest```

Замените адрес образа в файле `helm/smart-home-monolith/values.yaml` на полученный файл:

```yaml
image:
  repository: ghcr.io/<github_username>/architecture-sprint-3
  tag: latest
```

## Настройка terraform

[Установите Terraform](https://yandex.cloud/ru/docs/tutorials/infrastructure-management/terraform-quickstart#install-terraform)

Создайте файл ~/.terraformrc

```hcl
provider_installation {
  network_mirror {
    url = "https://terraform-mirror.yandexcloud.net/"
    include = ["registry.terraform.io/*/*"]
  }
  direct {
    exclude = ["registry.terraform.io/*/*"]
  }
}
```

## Применяем terraform конфигурацию

```bash
cd terraform
terraform init
terraform apply
```

## Настройка API GW

```bash
kusk deploy -i api.yaml
```

## Проверяем работоспособность

```bash
kubectl port-forward svc/kusk-gateway-envoy-fleet -n kusk-system 8080:80
curl localhost:8080/hello
```

## Delete minikube

```bash
minikube delete
```
