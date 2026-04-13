# homework-lesson-03-Yevhen-Marholin
# Dockerfile для simple-app

## Збірка образу

Було створено файл `Dockerfile` для застосунку `apps/simple-app`, який описує процес збірки та запуску контейнера.
## Dockerfile

```dockerfile
FROM golang:1.23-alpine AS deps
WORKDIR /app

COPY go.mod go.sum ./
RUN go mod download

FROM golang:1.23-alpine AS builder
WORKDIR /app

COPY --from=deps /go/pkg /go/pkg
COPY . .

RUN CGO_ENABLED=0 go build -trimpath -ldflags="-s -w" -o /out/simple-app .

FROM alpine:3.23.3 AS runtime
WORKDIR /app

RUN addgroup -S app && adduser -S app -G app

COPY --from=builder /out/simple-app /app/simple-app

USER app

EXPOSE 8080

CMD ["./simple-app"]
```


```bash
docker build -t simple-app .
```

## Запуск контейнера

```bash
docker run --rm -p 8080:8080 --name simple-app simple-app
```

Відкрити в браузері:
- http://localhost:8080
- http://localhost:8080/health
додав https://github.com/YevhenMarholin/homework-lesson-03-Yevhen-Marholin/blob/46c2399d225bfd65867228b412fb745435c6095d/image.png
---

## Порт

Застосунок працює на порту `8080`, який проброшено назовні.

---

## Користувач

Контейнер запускається не від root, а від окремого користувача:

```
app
```

---

## Multi-stage build

У Dockerfile використано multi-stage build:

- етап `deps` — завантаження залежностей
- етап `builder` — збірка застосунку
- етап `runtime` — фінальний образ для запуску

Це дозволяє значно зменшити розмір образу, оскільки у фінальний image не потрапляють:

- Go компілятор
- кеш збірки
- вихідний код
- залежності, потрібні лише для збірки

У фінальному образі залишається лише:
- скомпільований бінарник
- мінімальне середовище виконання

Завантажити образ у власний реєстр (Docker Hub/GHCR) і запустити його з реєстру (опціонально).

## Завантаження образу у Docker Hub

Образ було завантажено у Docker Hub.

### Тегування образу
```bash
docker tag simple-app yevhen_marholin/simple-app:latest
```

### Авторизація
```bash
docker login
```

### Завантаження образу
```bash
docker push yevhen_marholin/simple-app:latest
```

### Запуск з реєстру
```bash
docker run --rm -p 8080:8080 yevhen_marholin/simple-app:latest
```

```
@YevhenMarholin ➜ /workspaces/homework-lesson-03-Yevhen-Marholin/apps/simple-app (main) $ docker build -t simple-app .
[+] Building 33.9s (19/19) FINISHED                                                                    docker:default
 => [internal] load build definition from Dockerfile                                                             0.0s
 => => transferring dockerfile: 482B                                                                             0.0s
 => [internal] load metadata for docker.io/library/golang:1.23-alpine                                            0.5s
 => [internal] load metadata for docker.io/library/alpine:3.23.3                                                 0.5s
 => [auth] library/golang:pull token for registry-1.docker.io                                                    0.0s
 => [auth] library/alpine:pull token for registry-1.docker.io                                                    0.0s
 => [internal] load .dockerignore                                                                                0.0s
 => => transferring context: 2B                                                                                  0.0s
 => [deps 1/4] FROM docker.io/library/golang:1.23-alpine@sha256:383395b794dffa5b53012a212365d40c8e37109a626ca30  8.5s
 => => resolve docker.io/library/golang:1.23-alpine@sha256:383395b794dffa5b53012a212365d40c8e37109a626ca30d6151  0.0s
 => => sha256:9824c27679d3b27c5e1cb00a73adb6f4f8d556994111c12db3c5d61a0c843df8 3.80MB / 3.80MB                   0.2s
 => => sha256:8371a51cbc44c1323f96695eaeaeeb1c44b2d30d138a279b348af5c18b2c5e24 282.45kB / 282.45kB               0.2s
 => => sha256:383395b794dffa5b53012a212365d40c8e37109a626ca30d6151c8348d380b5f 10.30kB / 10.30kB                 0.0s
 => => sha256:a7ecaac5efda22510d8c903bdc6b19026543f1eac3317d47363680df22161bd8 1.92kB / 1.92kB                   0.0s
 => => sha256:72fd06a5dd05d7cb31d0fa2a125e89eecc16fbff26acf95bbd2762cdcfb5af38 2.08kB / 2.08kB                   0.0s
 => => extracting sha256:9824c27679d3b27c5e1cb00a73adb6f4f8d556994111c12db3c5d61a0c843df8                        0.3s
 => => sha256:d5791340ef181f05ef1e99760e4be81cb2008d99b2bc1e56b2faa9b6295d0ac3 74.07MB / 74.07MB                 1.6s
 => => sha256:d3178a7b27090e22bc2464346d1cdc8f59d0fd7a241452e8148342edf3d0b1e9 126B / 126B                       0.5s
 => => sha256:4f4fb700ef54461cfa02571ae0db9a0dc1e0cdb5577484a6d75e68dc38e8acc1 0B / 32B                         33.3s
 => => extracting sha256:8371a51cbc44c1323f96695eaeaeeb1c44b2d30d138a279b348af5c18b2c5e24                        0.1s
 => => extracting sha256:d5791340ef181f05ef1e99760e4be81cb2008d99b2bc1e56b2faa9b6295d0ac3                        4.2s
 => => extracting sha256:d3178a7b27090e22bc2464346d1cdc8f59d0fd7a241452e8148342edf3d0b1e9                        0.0s
 => => extracting sha256:4f4fb700ef54461cfa02571ae0db9a0dc1e0cdb5577484a6d75e68dc38e8acc1                        0.0s
 => [runtime 1/4] FROM docker.io/library/alpine:3.23.3@sha256:25109184c71bdad752c8312a8623239686a9a2071e8825f20  0.6s
 => => resolve docker.io/library/alpine:3.23.3@sha256:25109184c71bdad752c8312a8623239686a9a2071e8825f20acb8f219  0.0s
 => => sha256:a40c03cbb81c59bfb0e0887ab0b1859727075da7b9cc576a1cec2c771f38c5fb 611B / 611B                       0.0s
 => => sha256:589002ba0eaed121a1dbf42f6648f29e5be55d5c8a6ee0f8eaa0285cc21ac153 3.86MB / 3.86MB                   0.2s
 => => sha256:25109184c71bdad752c8312a8623239686a9a2071e8825f20acb8f2198c3f659 9.22kB / 9.22kB                   0.0s
 => => sha256:59855d3dceb3ae53991193bd03301e082b2a7faa56a514b03527ae0ec2ce3a95 1.02kB / 1.02kB                   0.0s
 => => extracting sha256:589002ba0eaed121a1dbf42f6648f29e5be55d5c8a6ee0f8eaa0285cc21ac153                        0.3s
 => [internal] load build context                                                                                0.0s
 => => transferring context: 10.30kB                                                                             0.0s
 => [runtime 2/4] WORKDIR /app                                                                                   0.3s
 => [runtime 3/4] RUN addgroup -S app && adduser -S app -G app                                                   1.0s
 => [deps 2/4] WORKDIR /app                                                                                      0.1s
 => [deps 3/4] COPY go.mod go.sum ./                                                                             0.0s
 => [deps 4/4] RUN go mod download                                                                               0.3s
 => [builder 3/5] COPY --from=deps /go/pkg /go/pkg                                                               0.1s
 => [builder 4/5] COPY . .                                                                                       0.0s
 => [builder 5/5] RUN CGO_ENABLED=0 go build -trimpath -ldflags="-s -w" -o /out/simple-app .                    23.3s
 => [runtime 4/4] COPY --from=builder /out/simple-app /app/simple-app                                            0.1s
 => exporting to image                                                                                           0.3s
 => => exporting layers                                                                                          0.2s
 => => writing image sha256:a788d2f7b832eb59f413c73841f3b506374f2e8ba5ceb94a25ace795f502ab99                     0.1s
 => => naming to docker.io/library/simple-app                                                                    0.0s
@YevhenMarholin ➜ /workspaces/homework-lesson-03-Yevhen-Marholin/apps/simple-app (main) $ docker run --rm -p 8080:8080 --name simple-app simple-app
2026/04/13 13:23:15 🚀 Server starting on http://localhost:8080
2026/04/13 13:23:15 📦 Version: 1.0.0 | Environment: development
```