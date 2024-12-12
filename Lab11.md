# Lab 11. Docker swarm.md
## Практические задания
### 1. Выполняем установку кластера с одной управляющей и двумя рабочими нодами.
На одном из узлов выполняем инициализацию Swarm
```sh
docker swarm init --advertise-addr eth0
docker swarm join
```
### 2. Удаление и повторное добавление рабочей ноды
Выбираем узел, который хотим удалить и выполняем
```sh
docker swarm leave
```
### 3. Создание Docker-образа с метаданными узла
Создаем простое приложение на языке JS(к примеру), которое выводит метаданные узла, включая IP.
```sh
server.js
-------------------------
const express = require('express');
const os = require('os');

const app = express();
const port = 3000;

app.get('/', (req, res) => {
  res.send(`Node IP: ${os.networkInterfaces().eth0[0].address}\n`);
});

app.listen(port, () => {
  console.log(`Server running on http://localhost:${port}`);
});
```
```sh
Dockerfile:
-------------------------
ROM node:14

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .

EXPOSE 3000

CMD ["node", "server.js"]
```
После чего необходимо собрать образ
```sh
docker build -t metadata-app .
```
### 4. Развёртывание сервиса с 3 репликами
```sh
docker service create --replicas 3 -p 3000:3000 --name metadata-service metadata-app
```
### 5. Пересборка образа с дополнительной информацией о разработчике
Обновляем код в 'server.js' добавив информацию о разработчике и пересобираем образ
```sh
server.js
-------------------------
const express = require('express');
const os = require('os');

const app = express();
const port = 3000;

app.get('/', (req, res) => {
  res.send(`Node IP: ${os.networkInterfaces().eth0[0].address}\nDeveloper: Your Name\n`);
});

app.listen(port, () => {
  console.log(`Server running on http://localhost:${port}`);
});
```
Пересобираем образ
```sh
docker build -t metadata-app .
```
Обновляем сервис
```sh
docker service update --image metadata-app metadata-service
```
