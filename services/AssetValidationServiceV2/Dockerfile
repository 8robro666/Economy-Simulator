FROM golang:1.21-alpine AS builder

WORKDIR /app

COPY go.mod go.sum ./
RUN go mod download

COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o asset-validation-service ./main.go

FROM alpine:latest AS runner

WORKDIR /app

RUN apk --no-cache add ca-certificates

COPY --from=builder /app/asset-validation-service .

EXPOSE 4300

CMD ["./asset-validation-service"]