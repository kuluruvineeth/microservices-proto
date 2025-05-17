# Microservices Proto

- .proto files are grouped by service folders such as order/order.proto in the root
folder.
- There is a folder inside the root project for each language to store languagespecific implementations.
- Generated source code for each service will be formatted as a typical Go module project since it will be added as a dependency on the consumer side.
– As an example, the module name of the Order service will be
github.com/kuluruvineeth/microservices-proto/golang/order.
- Generated source code will be tagged: golang/<service_name>/<version> (e.g.,
golang/order/v1.2.3). This is the convention for the Go module to resolve the
dependency that lives in subfolders in the remote repository

```bash
├── golang
│   ├── order
│   │   ├── go.mod
│   │   ├── go.sum
│   │   ├── order.pb.go
│   │   └── order_grpc.pb.go
│   ├── payment
│   │   ├── go.mod
│   │   ├── go.sum
│   │   ├── payment.pb.go
│   │   └── payment_grpc.pb.go
│   └── shipping
│       ├── go.mod
│       ├── go.sum
│       ├── shipping.pb.go
│       └── shipping_grpc.pb.go
├── order
│   └── order.proto
├── payment
│   └── payment.proto
└── shipping
    └── shipping.proto
```

## Code Generation with protoc.sh

The repository includes a `protoc.sh` script that automates the process of generating Go code from .proto files and releasing new versions:

```bash
./protoc.sh <SERVICE_NAME> <RELEASE_VERSION> <USER_NAME> <EMAIL>
```

### Parameters

- `SERVICE_NAME`: The name of the service (e.g., order, payment, shipping)
- `RELEASE_VERSION`: The version for release tagging (e.g., v1.2.3)
- `USER_NAME`: Git username
- `EMAIL`: Git email

### What the script does

1. Configures Git user information
2. Installs Protocol Buffers compiler and dependencies
3. Generates Go code from .proto files in the specified service directory
4. Sets up Go module for the generated code
5. Commits and pushes changes to the main branch
6. Creates and pushes a Git tag with format `golang/<SERVICE_NAME>/<RELEASE_VERSION>`

This allows other projects to import the generated code using Go modules with a specific version.

## GitHub Actions Workflow

The repository uses GitHub Actions to automate the code generation process with a workflow defined in `.github/workflows/protoc.yaml`:

```bash
name: "Protocol Buffer Go Stubs Generation"
on:
  push:
    tags:
      - v**
```

### Workflow Behavior

This workflow automatically triggers whenever a tag matching the pattern `v**` (like v1.0.0) is pushed to the repository. It will:

1. Set up a Go environment
2. Check out the repository code
3. Extract the release version from the tag
4. Run the `protoc.sh` script for each service defined in the matrix (order, payment, shipping)

To trigger a new code generation for all services:

```bash
git tag -a golang/order/v1.0.0 -m "golang/order/v1.0.0"
git push --tags
```

This approach allows you to update all service definitions at once with a single tag push, after which the GitHub Actions workflow handles the code generation and tagging for each individual service.
