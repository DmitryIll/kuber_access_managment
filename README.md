# Домашнее задание к занятию «Управление доступом» - Илларионов Дмитрий

### Цель задания

В тестовой среде Kubernetes нужно предоставить ограниченный доступ пользователю.

------

### Чеклист готовности к домашнему заданию

1. Установлено k8s-решение, например MicroK8S.
2. Установленный локальный kubectl.
3. Редактор YAML-файлов с подключённым github-репозиторием.

------

### Инструменты / дополнительные материалы, которые пригодятся для выполнения задания

1. [Описание](https://kubernetes.io/docs/reference/access-authn-authz/rbac/) RBAC.
2. [Пользователи и авторизация RBAC в Kubernetes](https://habr.com/ru/company/flant/blog/470503/).
3. [RBAC with Kubernetes in Minikube](https://medium.com/@HoussemDellai/rbac-with-kubernetes-in-minikube-4deed658ea7b).

------

### Задание 1. Создайте конфигурацию для подключения пользователя

1. Создайте и подпишите SSL-сертификат для подключения к кластеру.

На рабочей ВМ выполнил (от куда пользователь будет коннектиться): 

```
useradd -m -d /home/kubereader kubereader
cd /home/kubereader
openssl genrsa -out kubereader.key 2048
openssl req -new -key kubereader.key -out kubereader.csr -subj "/CN=kubereader"
```

Появились файлы с закрытым ключем и запроса на сертификат:
![alt text](image.png)

![alt text](image-1.png)

Конвертирую:

```
cat kubereader.csr | base64 | tr -d '\n'
```
![alt text](image-2.png)

Копирую код в запрос (создаю файл):

```
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: kubereader-csr
spec:
  # groups:
  # - system: authenticated
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1dqQ0NBVUlDQVFBd0ZURVRNQkVHQTFVRUF3d0thM1ZpWlhKbFlXUmxjakNDQVNJd0RRWUpLb1pJaHZjTgpBUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTmhyaWMzQXBGckhnNEF1bmErOVJRYktUVEszVXBRNytQYkRNbCs3Ck9JOEVtSTZuUkVyTVVRR1ZsdEhqdVlxcm9MZGhEWU9pOVhsdEpSN2pIbVZoUHBONVZyTFpOWnV6b3QyeCtqY0UKQ0FtZGNRZFo0M3pvdDZ5SGU0cWtEdHg5N01yMVZscEhUeElWeGZPWFd0L0F3RnBhVDFLeElIMEZ1cE5qMzF6RQpoOGh5ZHJRaUdHSnJEaForeFhjYU1JcC83Z01WbnI1RXpPeVNNTVRyajhRQjg3Q3BlV2NBWFhhRzJrL2RtUzZsCjZuemJWci85U3JoNlc2enZ2VlloR1VMeXZjdGM0V2ZtRGt1enc0NzBZVlpmeE9NVjV1K2EzbkhqVytVTUU5MmUKcXFTNnNxMmdhMXhiM3JIbktJMXBVR1hwZGJNUlc4eWQzNkVZYTRRZStsd25sVlVDQXdFQUFhQUFNQTBHQ1NxRwpTSWIzRFFFQkN3VUFBNElCQVFCelZmMUQyZG9jZnpINnQ5YnUxZU0yNm9RRHk4ckVwRWRYTjZCQ0g2ODc2MGhkCjE5RmRmUDFOSnNYeEFiN1M5ckhCZzNsVUpaY2FrQitUTWtQTDgzR0R2U0cybDdFRFRjQ1pMS0d3OUdMTEF4TGsKc21tTXRqQ0FyQjBqZUZvdW5Vb2MzTkhzL3R3ZGZJN3JtRERIKytOS3ltSnhxdTIvMnNHcG5UTUVOblYrRWRHcwpCbHVOa1I1V2RnMUhuYTdJWG9DMHptd3pQVHJDblNxM3JUNHdDekV4Q1pGdWpTM2FMbWsybFk1SmVWNlkwM1ExClNyTWlYRHdHR3IvaTg5MkRQNXoxcGI0aGxKSVJGN1g2VzBYZUoxNFN0alpzbnlVTTZxNUVPN2tDbCtBSTFqRlkKV1J2b2RCMzIzVTFpNmNtOVBkam5IQ1RKd1ZxN25OdTRFTTBTYUxVTQotLS0tLUVORCBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0K
  signerName: kubernetes.io/kube-apiserver-client-kubelet
  usages:
  - digital signature
  - key encipherment
  - server auth
  - client auth
```

Применяю:

```
kubectl -f ssl-csr.yaml apply
```
![alt text](image-3.png)


![alt text](image-4.png)

```
kubectl describe csr kubereader-csr
```
![alt text](image-5.png)

```
kubectl certificate approve kubereader-csr
```
![alt text](image-6.png)

Видимо что-то не сработало:

![alt text](image-7.png)

```
kubectl get csr/${SERVICE}.${NAMESPACE} \
--output=jsonpath="{.status}" \
| jq .
```

![alt text](image-8.png)

Предполагаю проблемв  в 

```
"reason": "SignerValidationFailure"
```



- Failed

Почему?
Что не так?

2. Настройте конфигурационный файл kubectl для подключения.
3. Создайте роли и все необходимые настройки для пользователя.
4. Предусмотрите права пользователя. Пользователь может просматривать логи подов и их конфигурацию (`kubectl logs pod <pod_id>`, `kubectl describe pod <pod_id>`).
5. Предоставьте манифесты и скриншоты и/или вывод необходимых команд.

------

### Правила приёма работы

1. Домашняя работа оформляется в своём Git-репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl`, скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.

------

