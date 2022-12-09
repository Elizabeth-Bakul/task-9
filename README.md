#### Задание 9. ПИС.

Контроллеры:
```js
app.get("/", (_, res: Response) => {
  res.end(JSON.stringify(counter));
});

app.get("/stat", (_, res: Response) => {
  res.end(JSON.stringify(counter++));
});

app.get("/about", (_, res: Response) => {
  const name = "Liza Bakulina";
  const html = `<h3>Hello, ${name}!</h3>`;

  res.writeHead(200, {'Content-Type': 'text/html'});
  res.end(html);
});
```

```dockerfile
FROM node:latest
WORKDIR /app
COPY ["index.ts",  "package.json", "tsconfig.json", ".env", "./"]
RUN npm install
RUN npm run build
CMD npm run start
EXPOSE 8080
```

Использование образа:
```text
docker build -t elizabethbbakul/task-7:latest .
docker push elizabethbbakul/task-7:latest
docker pull elizabethbbakul/task-7:latest
docker run elizabethbbakul/task-7:latest
```


C помощью docker-compose запускаем два контейнера (первый с базой, второй – с приложением-счетчиком) с помощью команды:
```text
docker-compose up
```
Внутрь прокидываются переменные окружения из файла .env, счетчик цепляется к базе и производит запросы через pg-promise. База данных – PostgreSQL.

#### CI / CD
GitHub Actions скрипты для автоматической сборки и развертывания приложения присутствуют в соответствующей папке.
Разворачивание происходит на удаленный арендованный сервер.

#### Кластеризация

Предлагается использовать OpenShift-манифесты - для deployment'ов и service'ов.
Имеем развернутый OpenShift / Kubernetes, либо локально запущенный (minikube).
Команды для обработки шаблона deployment, применения сервисов для каждого приложения:

```text
oc process -f openshift/itmo-counter/deployment-template.yml -p IMAGE_TAG=latest | oc apply -f -
oc apply -f openshift/itmo-counter/service.yml

oc process -f openshift/itmo-postgres/deployment-template.yml -p IMAGE_TAG=latest | oc apply -f -
oc apply -f openshift/itmo-postgres/service.yml
```

Таким образом, для деплоймента счетчика указываются 4 реплики - будут созданы 4 подов, смотрящие на базу (которая создается другим деплойментом). 
Общение между ними происходит через сервисы. При желании, для доступа извне можно организовать route.