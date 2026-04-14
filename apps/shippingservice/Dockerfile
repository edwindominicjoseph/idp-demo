ARG BUILDPLATFORM=linux/amd64

FROM --platform=$BUILDPLATFORM golang:1.26.1-alpine@sha256:2389ebfa5b7f43eeafbd6be0c3700cc46690ef842ad962f6c5bd6be49ed82039 AS builder
ARG TARGETOS=linux
ARG TARGETARCH=amd64
WORKDIR /src

COPY src/shippingservice/go.mod src/shippingservice/go.sum ./
RUN go mod download

COPY src/shippingservice/ .
RUN GOOS=${TARGETOS} GOARCH=${TARGETARCH} CGO_ENABLED=0 go build -ldflags="-s -w" -o /go/bin/shippingservice .

FROM gcr.io/distroless/static

WORKDIR /src
COPY --from=builder /go/bin/shippingservice /src/shippingservice
ENV APP_PORT=50051
ENV GOTRACEBACK=single
EXPOSE 50051
ENTRYPOINT ["/src/shippingservice"]

