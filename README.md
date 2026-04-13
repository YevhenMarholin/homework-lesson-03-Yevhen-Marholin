# homework-lesson-03-Yevhen-Marholin
# Dockerfile для simple-app

## Збірка образу

Було створено файл `Dockerfile` для застосунку `apps/simple-app`, який описує процес збірки та запуску контейнера.

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