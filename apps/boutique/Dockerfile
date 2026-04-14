FROM golang:1.25-alpine AS builder
WORKDIR /src

COPY src/frontend/go.mod src/frontend/go.sum ./
RUN go mod download

COPY src/frontend/ .

RUN CGO_ENABLED=0 GOOS=linux go build -ldflags="-s -w" -o /go/bin/frontend .

FROM gcr.io/distroless/static
WORKDIR /src
COPY --from=builder /go/bin/frontend /src/server
COPY src/frontend/templates ./templates
COPY src/frontend/static ./static

ENV GOTRACEBACK=single
EXPOSE 8080
ENTRYPOINT ["/src/server"]
