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