# homework-lesson-03-Yevhen-Marholin
# Dockerfile для simple-app

## Збірка образу

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
додав  ![alt text](image-1.png)
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