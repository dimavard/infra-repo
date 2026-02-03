                                          Как ставил Flux (без секретов)?

Установил Flux посредством chocolatey.
Сгенерировал GIT-токен

Flux BOOTSTRAP 
---------------------------------
$env:GITHUB_TOKEN="###"
flux bootstrap github
  --owner=dimavard
  --repository=infra-repo
  --branch=main
  --path=./clusters/my-cluster
  --personal
---------------------------------

                                            Как устроены директории?
Максимально примитивно.
Не трогал автоматически созданные Flux'ом.
Остальные файлы создавал в корне.

                                            Как проверить podinfo (команда port-forward и URL)?

kubectl port-forward -n podinfo svc/podinfo 8080:80
127.0.0.1:8080

                                            Что ломал в шаге 7 и как чинил?

Менял тег образа podinfo на несуществующий.
Диагностировал через Freelens, kubectl и flux:

Freelens выдал warning в котором было указано на попытку использования несуществующего образа.

kubectl get pods -n podinfo
№ STATUS: ImagePullBackOff

kubectl describe pod -n podinfo -l app=podinfo
# Events: Failed to pull image "stefanprodan/podinfo:6.6.6": not found

flux get kustomizations
# READY: True (reconciliation succeeded — конфиг доставлен)
# Но поды не запускаются

Flux успешно применил манифесты, но Kubernetes не смог запустить контейнеры
из-за отсутствующего образа. Rolling update создал новый ReplicaSet с битым
образом, старые поды продолжали работать
